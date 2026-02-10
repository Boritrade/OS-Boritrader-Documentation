# Server Documentation: Asgi & Rigis
**Boritradder LLC**   
**Last Updated:** May 28, 2025  

## ASGI Server

1. **Related Files and References**:
    - `config/asgi.py`
    - `config/settings.py`
    - `dashboard/tasks.py`
    - `dashboard/consumers.py`
    - `dashboard/routing.py`
    - `Dockerfile`
    - `supervisord.conf`
    - `docker-compose.yml`

### What is ASGI?

- **ASGI (Asynchronous Server Gateway Interface)** is a standard for Python asynchronous web servers and applications.
- It supports long-lived connections, such as WebSockets, allowing for real-time features in web applications.
- It is particularly useful for applications that require real-time updates, such as chat applications, live notifications, and online games.
- In this application it is used to send notifications to the foreground application (django server) from algorithms running in the background. 

## ASGI Server Workflow Overview

1. **Configuration**:
    - The ASGI server is configured in the `asgi.py` file, which sets up the application to handle both HTTP and WebSocket protocols.
    - The configuration ensures that WebSocket connections are routed correctly and that authentication is managed.

2. **WebSocket Connections**:
    - WebSocket URL patterns are defined in a routing file. These patterns map specific URLs to WebSocket consumers, which handle the communication.
    - When a client connects to the WebSocket URL, the connection is established and managed by the ASGI server.

3. **Consumer Setup**:
    - A WebSocket consumer is a class that manages WebSocket connections. It handles events like connection, disconnection, and receiving messages.
    - In this application, the consumer is responsible for adding the connection to a group that will receive notifications.

4. **Notification Group**:
    - When the WebSocket consumer accepts a connection, it adds the connection to a group: `notifications_<algo>_<ticker>`. This group is used to broadcast messages to all connected clients.

5. **Sending Notifications**:
    - When a trading algorithm triggers an event (such as a buy or sell signal), it calls a function to send a notification.
    - This function sends a message to the corresponding `notifications_<algo>_<ticker>` group. The message includes details about the event, such as the ticker symbol, the algorithm name, and the event type.

6. **Broadcasting Messages**:
    - The WebSocket consumer listens for messages sent to the group. When a message is received, the consumer sends the message to all clients connected to the WebSocket.
    - Clients receive the notification in real-time, allowing them to react to trading events immediately.

### Integration with Other Components

- **Settings Configuration**:
    - The settings file configures the ASGI application and the channel layers, specifying the backend (Redis) for managing the real-time connections.

- **Docker and Supervisord**:
    - Docker and Supervisord are used to manage and run the ASGI server along with other services like Redis, ensuring that the application is scalable and can handle multiple connections efficiently.





## Code Breakdown

### 1. **config/asgi.py**:
- **Purpose**: The `asgi.py` file sets up the ASGI application instance for handling HTTP and WebSocket connections using Django's `get_asgi_application` function.
- **How it Works**: 
    - Configures the environment to specify the settings module for the Django application.
    - Uses `ProtocolTypeRouter` to route HTTP requests to `get_asgi_application` and WebSocket connections to `AuthMiddlewareStack`.
    - The `AuthMiddlewareStack` adds authentication to WebSocket connections.
    - The `URLRouter` routes WebSocket connections based on URL patterns defined in `dashboard/routing.py`.
- **Components**:
    - `ProtocolTypeRouter`: Routes different types of protocol requests to appropriate handlers.
    - `AuthMiddlewareStack`: Ensures WebSocket connections are authenticated.
    - `URLRouter`: Routes WebSocket connections to the appropriate consumer based on URL patterns.
```python
"""
ASGI config for config project.

It exposes the ASGI callable as a module-level variable named ``application``.

For more information on this file, see
https://docs.djangoproject.com/en/5.0/howto/deployment/asgi/
"""

import os, sys
sys.path.append(os.path.join(os.path.dirname(os.path.abspath(__file__)), '..'))

from channels.auth import AuthMiddlewareStack
from channels.routing import ProtocolTypeRouter, URLRouter
from django.core.asgi import get_asgi_application
from dashboard.routing import websocket_urlpatterns

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')

application = ProtocolTypeRouter({
    "http": get_asgi_application(),
    "websocket": AuthMiddlewareStack(
        URLRouter(websocket_urlpatterns)
    ),
})
```

### 2. **config/settings.py**:
- **Purpose**: Configures the Django settings, including the ASGI application and channel layers for WebSocket support.
- **How it Works**:
    - Specifies the ASGI application module with `ASGI_APPLICATION`.
    - Configures Redis as the backend for channel layers using `CHANNEL_LAYERS`, enabling real-time features and WebSocket support.
- **Components**:
    - `ASGI_APPLICATION`: Defines the ASGI application entry point.
    - `CHANNEL_LAYERS`: Configures Redis for managing real-time WebSocket connections.
```python
...
# Channels & WebSocket
ASGI_APPLICATION = 'djangostuff.asgi.application'
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [('redis', 6379)],
        },
    },
}
```

### 3. **dashboard/routing.py**:
- **Purpose**: Defines the URL routing for WebSocket connections.
- **How it Works**:
    - Maps the WebSocket URL pattern `ws/notifications/` to the `NotificationConsumer`.
- **Components**:
    - `websocket_urlpatterns`: List of URL patterns for WebSocket connections.
```python
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
    re_path(r'ws/notifications/$', consumers.NotificationConsumer.as_asgi()),
]
```

### 4.4 **dashboard/algorithm_execution_central.py**
* **Purpose**
  Implements the *centralized* “master” execution loop that runs **all** enabled algorithms on **all** subscribed tickers (via the master bot), persists the results, and pushes notifications to exactly those user-specific groups who have opted in.

* **How it Works**

  1. **Bootstrapping**

     * Calls `get_or_create_master_bot()` to ensure a dedicated, disabled “master_bot” user exists with API keys, profile, notification & trading prefs, and `AlgorithmPreferences` seeded with every algorithm/ticker.
  2. **Loop**

     * Every 2 minutes:

       * Fetches `config = master_bot.get_configuration_settings()`.
       * Iterates over `config['algorithm_preferences']`, and for each `(algorithm_id, tickers)` pair:

         * Calls `master_bot.execute_algorithm(algo, ticker)` → `(entry_cond, exit_cond)`.
         * Translates to `"BUY"`, `"SELL"`, or `"DO NOTHING"`.
         * **Persists** to `SignalResult`.
         * Calls `notify_event(algo, ticker, signal)` to broadcast via Channels.

* **Key Functions**

  ```python
  def _group_name(algo: str, ticker: str) -> str:
      return f"notifications_{algo}_{ticker}"

  def notify_event(algo: str, ticker: str, signal: str):
      async_to_sync(channel_layer.group_send)(
          _group_name(algo, ticker),
          {
              "type": "send_notification",
              "notification": {
                  "algorithm_name": algo,
                  "ticker": ticker,
                  "event_type": signal,
              }
          }
      )

  def run_central_algorithm_loop():
      master_bot = get_or_create_master_bot()
      if not master_bot:
          logger.error("Central Runner :: Failed to initialize master bot.")
          return

      while True:
          try:
              prefs = master_bot.get_configuration_settings()['algorithm_preferences']
              for pref in prefs:
                  algo, tickers = pref["algorithm_id"], pref["tickers"]
                  for ticker in tickers:
                      entry, exit = master_bot.execute_algorithm(algo, ticker)
                      signal = "BUY" if entry else "SELL" if exit else "DO NOTHING"
                      SignalResult.objects.create(...)

                      notify_event(algo, ticker, signal)
              time.sleep(120)
          except Exception as e:
              logger.error(f"Central Runner crashed: {e}")
              time.sleep(60)

  def start_central_runner():
      threading.Thread(target=run_central_algorithm_loop, daemon=True).start()
  ```

---

### 4.5 **dashboard/algorithm_execution_user.py**

* **Purpose**
  Enables **premium** users to run their *own* algorithms on their *own* schedules, completely separate from the master loop behind a subscription guard.

* **How it Works**

  1. **Thread Tracker**

     * Guards via `AlgorithmThreadTracker` so each user only ever gets one background thread.
  2. **Per‐User Loop**

     * On `start_algorithm_threads(user)`, spawns a daemon thread that:

       * Retrieves the user’s `BoritraderBot` via `get_or_create_user_bot(user)`.
       * In an infinite loop (sleeping e.g. 10 min between runs):

         * Loads `algo_preferences = bot.get_configuration_settings()['algorithm_preferences']` and `enabled = bot.get_enabled_algorithms()`.
         * For each enabled algo, schedules `run_algorithm(bot, algo, tickers)` on a `ThreadPoolExecutor`.
  3. **run_algorithm**

     * Iterates its tickers, calls `bot.execute_algorithm(algo, ticker)`, and for each result sends to the same `notifications_<algo>_<ticker>` groups via `notify_socket()`.

* **Key Functions**

  ```python
  def run_algorithm(binance_bot, algo_name, tickers):
      for ticker in tickers:
          entry, exit = binance_bot.execute_algorithm(algo_name, ticker)
          if entry:  notify_socket(ticker, algo_name, 'BUY')
          if exit:   notify_socket(ticker, algo_name, 'SELL')
          if not entry and not exit:
                     notify_socket(ticker, algo_name, 'NOTHING_FOUND')

  def start_algorithm_threads(user):
      def run():
          if AlgorithmThreadTracker.is_thread_active(user.id):
              return
          AlgorithmThreadTracker.set_thread_active(user.id)

          bot = get_or_create_user_bot(user)
          while True:
              prefs = bot.get_configuration_settings()['algorithm_preferences']
              mapping = {p['algorithm_id']: p['tickers'] for p in prefs}
              enabled = bot.get_enabled_algorithms()
              for algo in enabled:
                  executor.submit(run_algorithm, bot, algo, mapping.get(algo, []))
              time.sleep(600)

          AlgorithmThreadTracker.set_thread_inactive(user.id)

      threading.Thread(target=run, daemon=True).start()

  def notify_socket(ticker, algo_name, event_type):
      grp = f"notifications_{algo_name}_{ticker}"
      async_to_sync(_channel.group_send)(
          grp,
          {
              "type": "send_notification",
              "notification": {
                  "ticker": ticker,
                  "algorithm_name": algo_name,
                  "event_type": event_type,
              }
          }
      )
  ```

### 5. **dashboard/consumers.py**:
- **Purpose**: Manages WebSocket connections and handles real-time notifications for trading events.
- **How it Works**:
    - `NotificationConsumer` class: Manages WebSocket events like connect, disconnect, and receive messages.
    - Adds connections to the "notifications" group and broadcasts messages to all connected clients.
- **Components**:
    - `connect()`: Handles new WebSocket connections and adds them to the notifications group.
    - `disconnect()`: Removes WebSocket connections from the notifications group.
    - `send_notification(event)`: Sends notification messages to WebSocket clients.
```python
from channels.generic.websocket import AsyncWebsocketConsumer
import json

class NotificationConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        user = self.scope["user"]
        if not user.is_authenticated:
            return await self.close()

        # subscribe per-algo/ticker
        prefs = await self._fetch_user_preferences(user)
        for algo_name, tickers in prefs:
            for ticker in tickers:
                await self.channel_layer.group_add(
                    f"notifications_{algo_name}_{ticker}",
                    self.channel_name
                )
        await self.accept()

    async def disconnect(self, close_code):
        for grp in getattr(self, "joined_groups", []):
            await self.channel_layer.group_discard(grp, self.channel_name)

    async def send_notification(self, event):
        await self.send(text_data=json.dumps(event["notification"]))

    @database_sync_to_async
    def _fetch_user_preferences(self, user):
        return list(
            user.algorithm_settings
                .values_list("algorithm__internal_name", "tickers")
        )
```

### 6. **Dockerfile**:
- **Purpose**: Defines the Docker image for the application, specifying how the application is built and run in a containerized environment.
- **How it Works**:
    - Configures the environment, installs dependencies, and sets up log files for ASGI and other services.
- **Components**:
    - `RUN touch /app/logs/...`: Ensures log files for various services are created.
```dockerfile
...
RUN touch /app/logs/supervisord.log /app/logs/supervisord.pid /app/logs/django.log /app/logs/django_err.log /app/logs/asgi.log /app/logs/asgi_err.log
```

### 7. **supervisord.conf**:
- **Purpose**: Configuration file for Supervisor, a process control system that manages and monitors application processes.
- **How it Works**
    - Uses supervisord to launch uvicorn server using asgi specifications.

```conf
...
uvicorn djangostuff.config.asgi:application --host 0.0.0.0 --port 8001
```

### 8. **docker-compose.yml**:
- **Purpose**: Defines the Docker services for the application, including the web server, database, and any other required services.
- **How it Works**:
    - Configures services, networks, and volumes.
    - Exposes ports for the ASGI server and other services.
- **Components**:
    - `services`: Defines each service required for the application.
    - `ports`: Exposes the necessary ports for each service.
```yaml
version: '3'

services:
redis:
    image: redis:latest
    ports:
    - "6379:6379"

django:
    build: .
    ports:
    - "8000:8000"
    - "8001:8001"
    depends_on:
    - redis
    env_file:
    - ./../env.list
```

---

## RIGIS Channel Server

- **Purpose**: Used for managing specific types of connections and data processing that require separate handling from the main application server.
- **Related Files**:
    - `config/settings.py`
    - `docker-compose.yml`
    - `Dockerfile`
    - `supervisord.conf`

### Configuration and Setup

### 1. **config/settings.py**:
- **Purpose**: Configures custom apps and the channel layers for the RIGIS server.
- **How it Works**:
    - Specifies additional apps and channel layers configurations for managing real-time connections and data processing.
- **Related Sections**:
    - `INSTALLED_APPS`: Lists all installed apps, including custom and third-party apps.
    - `CHANNEL_LAYERS`: Configures Redis for managing real-time WebSocket connections.
```python
...
# Custom Apps
'myapp',
'dashboard.apps.DashboardConfig',
'channels',

...
# Channels & WebSocket
ASGI_APPLICATION = 'djangostuff.asgi.application'
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [('redis', 6379)],
        },
    },
}
```

### 2. **docker-compose.yml**:
- **Purpose**: Sets up and configures Docker services for the RIGIS server.
- **How it Works**:
    - Defines services, networks, and volumes for the RIGIS server.
    - Sets up Redis as a service for channel layers.
```yaml
...
services:
    redis:
        image: redis:latest
        ports:
        - "6379:6379"
```
