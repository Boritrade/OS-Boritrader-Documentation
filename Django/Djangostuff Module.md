# The Boritrader Documentation: DjangoStuff Module
**Boritrader LLC**   
**Last Updated:** May 28, 2025  

## Module Overview

The `djangostuff` module is the foundation of the Boritrader project, encompassing the web interface, API endpoints, and background task management essential for running the application. This module is built using the Django framework to ensure a scalable and robust structure for crypto trading operations.

The `djangostuff` module contains the core logic and functionality for the web application, including user interface, database models, background tasks, and integration with the crypto trading module.

### Related Files and References:

- `djangostuff/config/`
- `djangostuff/dashboard/`
- `djangostuff/db.sqlite3` (legacy)
- `djangostuff/manage.py`

---

## Detailed Explanation

### `djangostuff/config/`

- **Purpose**: Contains the main configuration files for the Django application, including database settings, middleware, app registrations, and other project-wide configurations.
- **Key Components**:
  - **settings.py**: Core configuration file with settings for database connections, installed apps, middleware, etc.
  - **urls.py**: Central URL configuration for routing incoming HTTP requests.
  - **wsgi.py (Not Used)**: WSGI configuration for deploying the project on a WSGI server.
  - **asgi.py**: ASGI configuration to support asynchronous tasks and WebSocket connections, linking `dashboard/routing.py`.

### `djangostuff/dashboard/`

- **Purpose**: Contains the core logic for the dashboard (the Home Page, and core of the app) and supporting features, including views, models, tasks, and WebSocket consumers.
- **Key Components**:
  - **admin.py**: Configuration for Django's admin interface. Automatically loads all models listed in `models.py`for management. 
  - **apps.py**: Initializes the dashboard app and runs the `cleanup_inactive_bots()` function from `binance_bot_instance.py`, as well as the `clear()` function from `ThreadManager.py`. Both functions serve to clean up inactive user resources.
  - **routing.py**: Specifies WebSocket URL patterns for notifications, used by `config/asgi.py`.
  - **consumers.py**: Manages WebSocket connections, handling notifications, joining and leaving groups, and message broadcasting. Joins per `notifications_<algorithm>_<ticker>` groups and broadcasts messages.
  - **tests.py**: Currently unused. Tests stored in `dashboard/tests/` instead. See `Unit Testing` Documentation for more info.
  - **urls.py**: Contains the URL routes associated with `views.py`.
  - **models.py**: Defines and manages the database schema, connected to a PostgreSQL database for persistent data storage.
  - 
  - **tasks.py**:
    - **start_system_threads()** Calls `dashboard.algorithm_execution_central.start_central_runner()`

  - **algorithm_execution_central.py**:
      * Master every-2-minute loop that:
        * Fetches `master_bot` via `get_or_create_master_bot()`.
        * Runs each algorithm/ticker, writes to `SignalResult`, and calls `notify_event(...)` to push to WebSocket groups.

  - **algorithm_execution_user.py**: *(Premium-user only)*
    - **start_algorithm_threads(user)**
      - Spawns one daemon thread per user (guarded by `ThreadManager`)
      - Periodically (user-configurable) dispatches `run_algorithm(...)` via a `ThreadPoolExecutor`.
    - **run_algorithm(binance_bot, algo_name, tickers)**
      - Executes entry/exit for each ticker, sends to `notifications_<algo>_<ticker>` groups.
    - **notify_socket**: Sends real-time notifications to the dashboard via WebSocket.

  - **binance_bot_instance.py**:
    * **get_or_create_user_bot(user)**: Thread-safe factory for per-user `BoritraderBot`.
    * **get_or_create_master_bot()**: Ensures a disabled “master_bot” Django user exists, seeds its `AlgorithmPreferences`, then returns its `BoritraderBot` instance.


- **Helper Classes**:  
  - Managers:
    - **ThreadManager**: A thread-safe utility class for managing the active status of user-specific algorithm threads in `tasks.py`.
      - **is_thread_active(user_id)**: Checks if a thread is currently active for a given user ID.
      - **set_thread_active(user_id)**: Marks a thread as active for a given user ID.
      - **set_thread_inactive(user_id)**: Marks a thread as inactive for a given user ID.
      - **list_active_threads()**: Lists all currently active threads as a dictionary.
      - **clear()**: Clears all thread statuses, marking all threads as inactive.
    - **NotificationManager**: Database Object class for managing user algorithm notification entries from Websocket updates in `dashboard.js`. Related: `Shared Resources` Documentation, section: `algorithm_status.html`.
    - **SignalManager**: Database Object class for managing system signals from master_bot `SignalResult` records and prunes *all* signals older than 30 days.
  - **binance_bot_instance.py**:
    - **get_or_create_user_bot(user)**: Retrieves or creates a `binance_bot` instance (thread-safe) of the `BinanceTrader` class responsible for trading operations on per-user basis.
    -  **get_or_create_master_bot()**: Ensures a disabled “master_bot” Django user exists, seeds its `AlgorithmPreferences`, then returns its `BoritraderBot` instance, responsible for trading operations on a system-wide basis. 
    - **cleanup_inactive_bots()**: Cleans up inactive  `binance_bot` user instances periodically.
  - **DatabaseFetcher.py**: A helper class designed for efficient data retrieval from the database, supplying user's the configuration data for the `binance_bot` (the user's instance of the `BinanceTrader` class).

- **All Views**:
  - **views.py**: Manages the main dashboard functionality, user interactions, and serves the `dashboard.html` template.
  - **views_auth.py**: Handles user authentication processes: login, logout, and registration. Serves the `login.html`, and `register.html` templates. Routes for these functions are defined in `config/urls.py`.
  - **views_settings.py**: Manages user settings, including rendering the `settings.html` page. Corresponding routes are also in `config/urls.py`.
  - **views_health**: Returns JSON Response 200 OK when successfully accessed for health checkers (Docker, AWS ALB, etc). 
  - **views_robots**: Generates a robots.txt file dynamically, disallowing robots from internal pages.
  - **views_stripe.py**: Manages Stripe integration including creating checkout sessions, handling payment success and cancellation redirects, syncing subscription status, accessing the customer portal, and deleting user accounts while cancelling their Stripe subscriptions.
  - **views_webhooks.py**: Defines a CSRF-exempt endpoint that verifies Stripe webhook signatures and processes events—such as checkout session completions, subscription updates, cancellations, and payment failures—to update user profiles, log activity, and send corresponding notification emails.


- **Static and Template Files**
  - **static/**:
    - **css/**:
      - **base.css**: General page styling, including the header shared across pages.
      - **dashboard.css**: Specific styles for the main dashboard page.
      - **login_register.css**: Styles shared by `login.html` and `register.html`.
      - **settings.css**: Custom styles for `settings.html` page.
      - **algorithm_status.css**: Specific styles for formatting notificaitons in `algorithm_status.html`
    - **js/**:
      - **base.js**: Shared JavaScript logic for error & success message functionalities. Requires import statement, further requiring the use of type: module within html calls.  
      See documentation: Django/webpages/`Shared Resources` for more. 
      - **dashboard.js**: JavaScript for the dashboard's interactive features.
      - **login_register.js**: Shared JavaScript logic for login and registration functionalities.
      - **settings.js**: JavaScript for user settings interaction.
    - **img/**: All Image assets
      - **favicon.ico**: Boritrade Logo Asset (favicon version) used in all page meta data. 
      - **logo.png**: Boritrade Logo Asset - used in `dashboard.html`
      - **website-header-white.png**: Boritrade Banner Asset - used in `header.html`
      - **optional-background.jpg**: Background image used in `login.html` & `register.html` pages.

  - **templates/**:
    - **dashboard.html**: Main template for the dashboard interface.
    - **login.html**: Login page.
    - **register.html**: Registration page.
    - **settings.html**: User settings page.
    - **redirect_message.html**: Handles generic format for all redirect messages.
    - **includes/**:
      - **header.html**: Header template included in the `dashboard.html` and `settings.html` pages.
      - **footer.html**: Footer template included in the `dashboard.html` and `settings.html` pages.
      - **algorithm_preferences.html**: Modular HTML to display algorithm preferences form on the `dashboard.html` page.
      - **algorithm_details.html**: Modular HTML to display algorithm details on the `dashboard.html` page.
      - **algorithm_status.html**: Modular HTML to display algorithm status notifications on the `dashboard.html` page.
    - **recovery/**:
      - **emails/**: Holds email template to send password reset token
        - **password_reset_email.html**: HTML template for the password reset email, including the reset link and styled instructions for the user.
        - **password_reset_email_subject.txt**: Plain-text file defining the subject line used when sending the password reset email.
        - **password_reset_email.txt**: Plain-text version of the password reset email body, providing fallback instructions for clients that don’t render HTML.
      - **forgot_username.html**: Template displaying a form where users can enter their email to have their forgotten username sent to them.
      - **forgot_username_done.html**: Confirmation page shown after a user successfully submits the forgot-username form.
      - **password_reset_request.html**: Template rendering the form for users to request a password reset by entering their email address.
      - **password_reset_request_done.html**: Page confirming that the password reset request has been sent to the user’s email.
      - **password_reset_form.html**: Template for the form where users set and confirm a new password after following the reset link.
      - **password_reset_form_complete.html**: Final confirmation page indicating that the user’s password has been successfully reset.

### `djangostuff/db.sqlite3` (Legacy)

- **Purpose**: Previously used as the SQLite database file for development purposes but has been superseded by PostgreSQL for production.

### `djangostuff/manage.py`

- **Purpose**: Command-line utility for managing the Django project, running servers, migrations, and administrative tasks. Referenced in `supervisord.conf` for automated process control.

### `staticfiles/`

- **Purpose**: Contains compiled and processed static files used by the application, including admin-specific assets, images, and additional JavaScript and CSS libraries.

---

## Workflow Overview

1. **Initialization**:
   - The Django application is set up and initialized using `manage.py`, configuring the environment for handling user requests and background tasks.

2. **URL Routing**:
   - HTTP requests are routed through `config/urls.py` and further delegated to `urls.py` within `dashboard` or other apps.

3. **Request Handling**:
   - Views in `views.py`, `views_auth.py`, and `views_settings.py` handle various requests related to the dashboard, user authentication, and settings management.

4. **Background Tasks**:

   * **Central runner** started once per-application (in first dashboard view): every 2 minutes runs *all* algorithms/tickers via `algorithm_execution_central`.
   * **Premium runner** per-user via `algorithm_execution_user` behind subscription guard.
5. **Real-Time Communication**:

   * **notify_event** and **notify_socket** send to `notifications_<algo>_<ticker>` groups.
   * **NotificationConsumer** subscribes each socket to exactly those groups matching a user’s `AlgorithmPreferences`.

5. **Real-Time Communication**:
   - The `notify_socket` function in `tasks.py` uses Django Channels to send notifications to the WebSocket `notifications_<algo>_<ticker>` groups. This allows `consumers.py` to handle real-time messaging between the server and connected clients.
   - `dashboard/consumers.py`:
     - The `NotificationConsumer` class manages WebSocket connections, allowing users to receive updates in real-time as algorithms execute and trigger events. 
     - It handles WebSocket connections, joining groups, sending messages to connected clients, and managing disconnections.
     - Subscribes each socket to exactly those groups matching a user’s `AlgorithmPreferences`.
   - These notifications facilitate live updates to the dashboard interface, ensuring users see real-time trading data and results.

6. **ASGI and WebSocket Integration**:
   - The `config/asgi.py` file sets up the ASGI application, routing WebSocket connections using `ProtocolTypeRouter` and `AuthMiddlewareStack`.
   - WebSocket routes are defined in `dashboard/routing.py`, linking the URL pattern `/ws/notifications/` to the `NotificationConsumer`.
   - This WebSocket route is used in the client-side `dashboard.js` to establish a WebSocket connection, allowing the server to push updates to the user's browser for dynamic, real-time feedback on trading algorithm performance.