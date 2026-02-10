# **The Boritrader Documentation: Patch Notes – May 9 to June 4, 2025**

**Boritrade, LLC**
**Related Release Version:** 0.6.4

---

## **Overview**

1. **Central Algorithm Execution Refactor** completed — system now runs all algorithms globally via a single runner.
2. **API Key Requirement Removed** — personal API keys are now optional for registration and dashboard access.
3. **Improved Testing & Documentation** — added test coverage and internal documentation for algorithm updates and notification logic.
4. **Email Template Cleanup** — email formatting and delivery improved with branding tweaks and edge-case handling.
5. **Pipeline & EnvVar Updates** — new secrets, pipeline stability, and production compatibility verified.

---

## **Key Updates**

### **1. Algorithm Runner Refactor: Central Execution Enabled**

* **Old Behavior:**
  Algorithms were previously executed per user — each user connection created a thread for each of their selected tickers, leading to exponential CPU usage at scale.

* **New System:**
  All algorithms now run globally in a **single scheduled service**, and results are stored in the database.
  Users subscribe to tickers and receive updates based on global signals.

* **Architecture Changes:**

  * Introduced centralized runner file (`algorithm_central_execution.py`)
  * Separated algorithm execution from Django — now runs as a standalone service.
  * User API keys are optional; system uses a **default admin key** for analysis tasks.
  * Updated environment variables and permissions to manage key exposure.

* **Benefits:**
  ✅ Reduced critical CPU load from O(N) to O(1) 
  ✅ Scales predictably
  ✅ Simplifies user onboarding (no API key required)

* **Repercussions:**
  ⚠️ Signals are now universal — all users receive the same output per ticker.
  ⚠️ Database size increases due to continuous signal storage.

---

### **2. API Key Requirement Removed**

* **Registration Flow Updated:**
  Users can now sign up and access the dashboard without entering API keys.
  Personal keys are still supported for advanced features (e.g., automated trading).

* **Dashboard Logic Updated:**
  Fallbacks now use admin key for signal visibility; user keys override only if present and validated.

* **Security & Access:**
  Default key visibility is blocked from frontend; access controlled via env vars and subscription plan.

---

### **3. Notifications Improvements**

* **Centralized Notification Dispatch:**
  * Introduced `notify_user()` utility to streamline alerts for status changes and user updates.
  * Fixed critical issue where ALL users were receiving ALL notifications instead of subscription group-based blasting
  * All notifications sent are now stored in separate table for debugging (in addition to seprate table for notifications recieved by users)
---

### **4. Infrastructure & Pipeline Stability**

* **Pipeline Updates:**
  * Added new environment variables to GitHub Actions pipeline
  * Locked down secrets for compatibility with updated API key logic

* **Frontend Fixes:**

  * Fixed favicon display bug in Chrome
  * Linting and formatting cleanup across multiple files

---

### **5. Robust Error Handling for Algorithm Execution & Data Handling**

* **Custom Error Classes Added:**
  Introduced `LoadConfigError`, `LogTradeError`, & `InvalidApiCredentialsError` to improve visibility and control over failures during algorithm execution and data ingestion.

* **Validation Improvements:**

  * **API Key Validation:**
    * Invalid keys (missing `api_key` or `api_secret`) are now explicitly caught.
    * If no valid keys are returned, the system raises a `LoadConfigError` immediately.
    * If valid keys syntactically valid but found non-functional, sysem raises a `InvalidApiCredentialsError` and redirects user to their API key settings. 

  * **Trading Preferences & Notification Settings:**
    * `get_trading_preferences()` and `get_notification_preferences()` now raise `LoadConfigError` if either is missing.

  * **Algorithm Preferences Validation:**
    * Algorithm input configs are pre-validated before processing.
    * Misformatted entries now fail early rather than producing undefined behavior.


---

## **Goals for the Month**
1. **Asyncronous Task Handler** — handle one-off maintenance tasks outside basic app execution (inactive users, reminder emails, db cleanup, etc)
2. **Algorithm Preferences Ticker Update** - Add Popular Ticker Selection, Enforce USDT Suffix, and Improve Ticker Validation
3. **Special Admin Page for Stats, Maintenance** - Add Superuser-Only Admin Page for Statistics and Maintenance Tools
4. **Enhance Notification System** - Audible Alerts, Signal Change Analysis, and Email/SMS Delivery
5. **UX for Mobile devices** - Improve Mobile UX on Dashboard, Settings, Login, and Registration


