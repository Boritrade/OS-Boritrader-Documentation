# Central Execution
**Boritrader LLC**   
**Last Updated:** May 28, 2025  

This document describes Boritrader’s **central execution** path: the “master” bot that drives every algorithm every 2 minutes across all tickers, and how its events are pushed into the WebSocket layer for normal (free) users. It also contrasts this with the **per‐user execution** (“premium” path) and highlights outstanding work.

---

## Table of Contents

1. [Overview](#overview)
2. [Master Bot Lifecycle](#master-bot-lifecycle)

   1. [get_or_create_master_bot](#get_or_create_master_bot)
   2. [populate_tickers](#populate_tickers)
3. [Central Runner Loop](#central-runner-loop)

   1. [run_central_algorithm_loop](#run_central_algorithm_loop)
   2. [notify_event → WebSocket groups](#notify_event-→-websocket-groups)
   3. [start_central_runner](#start_central_runner)
4. [Per-User Execution (Premium)](#per-user-execution-premium)

   1. [get_or_create_user_bot](#get_or_create_user_bot)
   2. [algorithm_execution_user.py](#algorithm_execution_userpy)
5. [WebSocket Subscription Model](#websocket-subscription-model)

   1. [NotificationConsumer.connect](#notificationconsumerconnect)
   2. [Group naming convention](#group-naming-convention)
6. [Outstanding Work & TODOs](#outstanding-work--todos)

---

## 1. Overview

* **Central execution** runs under a special “master_bot” user in Django.
* Every **2 minutes**, it loops through *every* algorithm-ticker combination, computes buy/sell/do-nothing signals, writes a `SignalResult`, and broadcasts via Channels to only those clients who have subscribed to `notifications_<algo>_<ticker>`.
* **Premium users** may spin up their own background threads to run their personal algorithms at custom frequencies — behind a subscription paywall.

---

## 2. Master Bot Lifecycle

### get_or_create_master_bot

Located in `dashboard/binance_bot_instance.py`:

1. **Provision** a disabled Django user `username="master_bot"`.
2. If newly created:

   * assign an unusable password (`.set_unusable_password()`)
   * create one `AlgorithmPreferences` row per `Algorithm`, with placeholder tickers.
3. **Within a transaction**, ensure the master user has:

   * a `UserProfile`
   * default `TradingPreferences` & `NotificationPreferences`
   * an up-to-date `ApiKey` record loaded from `settings.MASTER_*`
4. Instantiate or refresh a single `BoritraderBot` under a module-level lock, store in `master_bot_instance[...]`.

### populate_tickers

Also in `binance_bot_instance.py`. Once the master bot exists:

* Eventually fetch the true universe of tickers via `master_bot.get_available_tickers()`.
* Overwrite the master’s `AlgorithmPreferences.tickers` with that list.
* Push the new config into the in-memory `BoritraderBot` via `update_configuration_settings(...)`.

---

## 3. Central Runner Loop

### run_central_algorithm_loop

Defined in `dashboard/algorithm_execution_central.py`. Every iteration:

```python
config = master_bot.get_configuration_settings()
for pref in config['algorithm_preferences']:
    algo    = pref['algorithm_id']
    tickers = pref['tickers']
    for ticker in tickers:
        entry, exit = master_bot.execute_algorithm(algo, ticker)
        signal = "BUY" if entry else "SELL" if exit else "DO NOTHING"

        SignalResult.objects.create(
            algorithm_name=algo,
            ticker=ticker,
            signal=signal,
            result_message=f"Entry={entry}, Exit={exit}"
        )

        notify_event(algo, ticker, signal)
```

* Sleeps 120 s between runs.
* Catches & logs exceptions per-ticker.

### notify_event → WebSocket groups

```python
def _group_name(algo, ticker) -> str:
    return f"notifications_{algo}_{ticker}"

def notify_event(algo, ticker, signal):
    async_to_sync(channel_layer.group_send)(
      _group_name(algo, ticker),
      {"type":"send_notification", "notification":{…}}
    )
```

Only sockets in that group receive the event.

### start_central_runner

Called once (e.g. in `AppConfig.ready()` or via a guarded view hook):

```python
def start_central_runner():
    threading.Thread(target=run_central_algorithm_loop, daemon=True).start()
```

---

## 4. Per-User Execution (Premium)

### get_or_create_user_bot

Also in `binance_bot_instance.py`. Similar pattern to master, keyed by `user.id`:

* On first access, builds a `BoritraderBot` with that user’s API key and `DatabaseFetcher(user)`.
* On subsequent calls updates its configuration.
* Tracks last-accessed for TTL cleanup.

### algorithm_execution_user.py

* Spins up a **daemon thread** per user calling `start_algorithm_threads(user)`.
* Inside that thread:

  * Guards with `AlgorithmThreadTracker` so only one runs per user.
  * Every N seconds (customizable per-user), pulls user’s algorithm_preferences from their `binance_bot` and fires off concurrent calls to `run_algorithm(...)` via a `ThreadPoolExecutor`.
  * Uses its own `notify_socket` to group_send into
    `notifications_<algo>_<ticker>` for users who subscribed.

---

## 5. WebSocket Subscription Model

### NotificationConsumer.connect

```python
async def connect(self):
  user = self.scope['user']
  if not user.is_authenticated:
     return await self.close()

  prefs = await database_sync_to_async(…AlgorithmPreferences…)
  for algo, ticker_list in prefs:
    for ticker in ticker_list:
      await self.channel_layer.group_add(f"notifications_{algo}_{ticker}", self.channel_name)

  await self.accept()
```

On **disconnect**, all joined groups are discarded automatically.

### Group naming convention

* **Group name**: `notifications_<internal_algo_name>_<TICKER>`
* Matches both central and per-user `notify_event` / `notify_socket` calls.

---

## 6. Outstanding Work & TODOs

1. **Dynamic ticker discovery**
   * Hook up `master_bot.get_available_tickers()` → `populate_tickers()`.

1. **Registration** 
   * Remove API key restrictions on registration & dashboard. 

1. **Custom Algorithm Preferences**
   * Add to settings for premium members: [enable custom execution?]
   * In dashboard > configure, re-enable AlgorithmPreferences.execution_frequency
   * Honor `AlgorithmPreferences.execution_frequency` in `algorithm_execution_user.py`.


### UPGRADES
1. **User preference changes**
   * Currently loaded only at `connect()` time. You may want to allow clients to subscribe/unsubscribe at runtime (e.g. via `receive()` messages).

1. **Rate-limit central loop**
   * If the universe of tickers grows very large, consider sharding or horizontalizing the master runner.

1. **Error‐handling & backoff**
   * Central loop currently sleeps a fixed 2 min on any error; you may want exponential backoff for persistent failures.

1. **Monitoring & metrics**
   * Expose logs/metrics on run durations, exceptions, per-ticker counts.