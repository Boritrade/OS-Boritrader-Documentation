# The Boritrader Documentation: Shared Resources
**Boritrader LLC**

## Overview

The **Shared Resources** in the Boritrader project consist of reusable HTML templates, CSS stylesheets, and JavaScript modules. These resources ensure consistency across the application by providing shared functionality and styling elements for various pages.

### Summary

- **`base.js`**: JavaScript module for managing error and success messages.
- **`base.css`**: Core CSS stylesheet for general styling, headers, and alerts.
- **`header.html`**: HTML template for the website’s navigation menu.
- **`algorithm_details.html`** - HTML template, controls the html formatting for the details popup of each algorithm.
- **`algorithm_preferences.html`** - HTML template, controls the html formatting of the algorithm preferences popup form of each algorithm.   
- **`algorithm_status.html`** - HTML template, controls html formatting of the algorithm status popup of each algorithm.

---

## Detailed Explanation

### `base.js`

#### Purpose

Provides utilities for displaying global error and success messages in the application. This modular approach ensures consistency and reusability across multiple JavaScript files.

#### Key Features

1. **Message Box Management**:
   - Dynamically creates a global message box (`#message-box`) to display alerts.
   - Supports success and error message types with customizable styles.

2. **Exported Functions**:
   - `showError(message)`: Displays an error message.
   - `showSuccess(message)`: Displays a success message.

3. **Message Display Logic**:
   - Messages appear for 3 seconds before disappearing automatically.
   - The message box styling dynamically adjusts based on the message type.

#### Import Usage

The `base.js` module must be imported into other JavaScript files to utilize its functions. For example, in `dashboard.js`:

```javascript
import { showError, showSuccess } from './base.js';
```

#### Additional Notes

- Requires scripts importing `base.js` (e.g., `dashboard.js`) to be declared as **modules** using the `<script type="module">` attribute in HTML.
- Example usage in `dashboard.html`:

```html
<script type="module" src="{% static 'js/dashboard.js' %}"></script>
```
- **Warning:** Scripts imported as modules will not have their functions shared with the html automatically. 

---

### `base.css`

#### Purpose

Serves as the foundational stylesheet for the Boritrader project, providing global styling rules and reusable components.

#### Key Style Topics

1. **Global Layout**:
   - Ensures consistent spacing, padding, and background colors for the entire application.

2. **Headers**:
   - Defines styles for the header, including layout, button alignment, font properties, and hover effects displayed via `header.html`.

3. **Message Box**:
   - Customizes the appearance of success and error messages displayed via `base.js`.

4. **Buttons**:
   - Provides general button styles, including hover effects and transitions.

5. **Typography**:
   - Applies consistent font-family and font-size settings across the application.

#### Import Usage

`base.css` is imported into other CSS files using the `@import` rule. For example, in `settings.css`:

```css
@import url('base.css');
```

This approach ensures that all styles defined in `base.css` are applied consistently across different pages.

---

### `header.html`

#### Purpose

A shared HTML template that provides a responsive navigation menu with user information and links to key pages (e.g., Logout, Home, Settings).

#### Key Components

1. **Header Layout**:
   - Displays user-specific details, such as username and ID.
   - Provides navigation buttons for quick access to essential pages.

2. **Styling Integration**:
   - Styled using classes defined in `base.css`.
   - Utilizes `header`-specific styles for consistent appearance.

3. **Dynamic Content**:
   - Uses Django template tags (e.g., `{{ user.username }}`, `{{ user.id }}`) to dynamically populate user information.

#### Import Usage

The `header.html` template is included in other HTML templates using the `{% include %}` tag. Example usage in `dashboard.html`:

```html
{% include 'includes/header.html' %}
```

---

### **`algorithm_details.html` and `algorithm_preferences.html`**

#### Purpose
These templates are essential for providing **dynamic, user-specific views** of algorithm details and preferences in `dashboard.html`. They are integrated with the dashboard to allow users to customize algorithm settings, and access detailed algorithm information.


#### Key Features of **`algorithm_details.html`**
- **Dynamic Algorithm Information**:
  - Displays the algorithm's **name** (`{{ algorithm.display_name }}`) and **description** (`{{ algorithm.description|safe }}`).
  - The description includes **HTML content** stored in the database, allowing for formatted text, lists, and other structural elements.
  
- **Warning Message**:
  - A warning is prominently displayed to remind users that the content is not financial advice.


#### Key Features of **`algorithm_preferences.html`**
- **Interactive Preferences Form**:
  - Provides input fields for **tickers**, **quantity**, and **execution frequency**, dynamically populated using the Django form (`AlgorithmPreferencesForm`).
  - Includes CSRF protection (`{% csrf_token %}`) for secure POST requests.
- **Customization Workflow**:
  - Enables users to modify algorithm-specific settings and save them to the database.


#### Workflow for Popup Content
1. **Trigger**:
   - Users click the **Configure** button (`.configure-btn`) or  (`.details-btn`) on the dashboard.
   - A JavaScript event listener (`dashboard.js`) calls the `loadForm(algorithmId)` or `loadDetails(algorithmId)` function.
2. **Backend Handling**:
   - The request is routed via `urls.py` to the `save_algorithm_preferences` or `algorithm_details` view in `views.py`, which loads corresponding html file from the `includes/` folder the selected algorithm.
3. **Frontend Display**:
   - The form is dynamically loaded into the `#popup-content` container in `dashboard.html` and displayed using the `popup-container` styling defined in `dashboard.css`.
   - The popup is hidden or shown using the `hidden` class, toggled by JavaScript functions (`closePopup`, `loadForm`, `loadDetails`).

4. **Form Submission** (algorithm_preferences only):
   - On form submission, the data is POSTed back to the same view (`save_algorithm_preferences`) and saved to the database if valid.


#### Key JavaScript Functions
- **`loadDetails(algorithmId)`**:
  - Fetches the `algorithm_details.html` content via a GET request to `/dashboard/algorithm_details/<algorithm_id>/`.
  - Inserts the response into `#popup-content`.

- **`loadForm(algorithmId)`**:
  - Fetches the `algorithm_preferences.html` form via a GET request to `/dashboard/save_algorithm_preferences/<algorithm_id>/`.
  - Binds a `submit` event listener to save the form data via a POST request.

---

### `algorithm_status.html`

#### Purpose

Provides a user-friendly interface for displaying the latest notifications and signals for a specific algorithm. The signals are populated dynamically using backend data and reflect the real-time updates pushed to the database.

Workflow for the popup is similar to the `algorithm_details` and `algorithm_preferences`, however this html also relies on additional js functions that process WebSocket data and upload to the database via the `update_algorithm_status` view, and custom styling with `algorithm_status.css`.
---

#### Workflow (in addition to default Popup window Workflow)

1. **Receiving Signals** `dashboard.js`:
   - The WebSocket connection listens for incoming messages.
   - Each message is parsed, logged, and passed to `update_algorithm_status` view.

1. **WebSocket Integration**: `update_algorithm_status` View:
   - Real-time signals for algorithms (e.g., BUY, SELL, HOLD) are received from a WebSocket connection.
   - The `update_algorithm_status` view, when called automatically after receiving updates from the Websocket, processes these signals and stores the signal in the database using the `NotificationManager`.

1. **Template Rendering**: `algorithm_status` View:
   - The `algorithm_status` view, when called manually by the user, retrieves the latest notifications for each ticker associated with the selected algorithm.
   - These notifications are passed to `algorithm_status.html` and displayed in a structured, styled format. This formatting is handled primarily by `algorithm_status.css`, in contrast to how `algorithm_details` & `algorithm_preferences` only use `dashboard.css` for formatting.

 #### Key JavaScript Functions

   - **`loadAlgorithmStatus(algorithmId)`**
      - fetches the `algorithm_status.html` content via a GET request to `/dashboard/algorithm_status/${algorithmId}/` url from `dashboard/urls.py`.


   - **`updateAlgorithmStatus(message)`**
      - called by Websocket watcher on new message, passes message to `parseMessage` then passes result to `update_algorithm_status` view using fetch to `/dashboard/update_algorithm_status/` url from `dashboard/urls.py`.

   - **`parseMessage(message)`**
      - Parses the message for algorithm name, ticker, signal, and timestamp.


#### `NotificationManager`

#### Purpose
Manages the creation and pruning of notifications entries for Notifications Table in the database (related `Database - Table Summary` Documentation). Ensures users receive up-to-date signals while maintaining a manageable database size.

#### Key Features

1. **Notification Creation**:
   - Creates notifications for user-specific algorithms and tickers.
   - Stores metadata including the algorithm name, message, ticker, status, and timestamp.

2. **Database Management**:
   - Maintains a maximum of `MAX_ENTRIES_PER_TICKER` notifications per ticker for each algorithm.
   - Automatically deletes older notifications once the limit is exceeded.

3. **Integration**:
   - Used by the `update_algorithm_status` view to handle incoming signal data.
   - Facilitates mapping of notifications to algorithms and users.

---

## Integration & Usage of Shared Resources

### Example Usage

In a typical workflow, shared resources integrate seamlessly to provide consistent styling and functionality:

1. **HTML Templates**:
   - Include `header.html` for navigation.
   - Dynamically load JavaScript modules like `dashboard.js` (which imports `base.js`).

2. **CSS**:
   - Apply `base.css` globally via `@import` in other CSS files.

3. **JavaScript**:
   - Use `showError` and `showSuccess` from `base.js` to display alerts.

### Example Code Snippets

#### `dashboard.html`

```html
<head>
    <link rel="stylesheet" href="{% static 'css/dashboard.css' %}">
    <meta name="csrf-token" content="{{ csrf_token }}">
    <script type="module" src="{% static 'js/dashboard.js' %}"></script>
</head>
<body>
    {% include 'includes/header.html' %}
    <div class="page-content">
        <!-- Page-specific content here -->
    </div>
</body>
```

#### `dashboard.css`

```css
@import url('base.css');
```

#### `dashboard.js`

```javascript
import { showError, showSuccess } from './base.js';

// Example usage
showSuccess("Dashboard loaded successfully!");
```
