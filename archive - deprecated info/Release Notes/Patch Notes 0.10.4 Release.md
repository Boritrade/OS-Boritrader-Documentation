# **The Boritrader Documentation: Patch Notes — September 4 to October 4, 2025**
**Boritrade, LLC**   
**Related Release Version:** 0.10.4

---

## Overview
**Focus:** reliable **offline email alerts**, groundwork for **SMS**, **Cloudflare Turnstile** security, and **logging/test** polish.

* **Offline Email Notifications (disabled)** for signal changes are implemented, taken offline until anti-bot & spam measures are in tightened.
* **SMS groundwork (Twilio - disabled)** added, **disabled by design** until policy/verification is complete.
* **Cloudflare Turnstile CAPTCHA** integrated (visible), with CI/CD env vars wired.
* **Notification routing tightened** to per-(algorithm, ticker) subscriptions; unsub flow improved.
* **Fan-out logging noise reduced** (removed per-message DB writes; kept SystemLog where useful).
* **Test suite expanded** around notifications and unsubscribe flows.

---

## **Key Updates**

### 1) **Notification Analysis & Signal-Change Highlighting**

* **What Changed:**
  Notifications now explicitly call out **state transitions** per **(algorithm, ticker)** (e.g., **BTCUSDT: HOLD → BUY**), rather than repeating the current state.

* **How It Works:**
  Tracks the previous signal and **emits only on change**; standardizes on **BUY/SELL/HOLD**; respects user **NotificationPreferences** and **AlgorithmPreferences**.

* **Why It Matters:**
  Focuses on **actionable deltas** users care about and lays groundwork for **digests**, **rate-limiting** high-volume tickers, and future **rule-based automations**.


### 2) Notifications Delivery

* **Email (Disabled for now):**

  * Implemented baseline **offline notification** flow for signal changes.
  * Pre-fetches **NotificationPreferences** and checks **AlgorithmPreferences** so users only receive alerts for subscribed **(algorithm, ticker)** pairs.
  * Minor improvements to **unsubscribe** handling and related logs.

* **SMS (Disabled for now):**

  * Added **Twilio** framework and message pipeline placeholders; **all Twilio calls are commented out** pending verification, consent, and rate-limit policy.
  * Standardized phone handling to **E.164** via a **PhoneNumber object** (phonenumbers/django-phonenumber-field).

### 3) Security & Anti-Abuse

* Added **Cloudflare Turnstile** CAPTCHA and made enforcement **visible** in UI.


### 4) Testing & QA

* Added tests around **offline notifications** and **unsubscribe** flows.
* General **linting** and **package** fixes; ensured all required deps are present.
* Removed per-message **DB logging** during fan-out; retained targeted **SystemLog** entries for high-level tracing.
* Trimmed test assertions that depended on SystemLog internals.

### 5) Dependencies & Ops

* Fixed **missing packages**; standardized installs via `requirements.txt`.
* CI/CD updated to include new **env vars** for notifications and Turnstile.

---


## **Known Limits / Follow-Ups**

* **SMS remains OFF** until verification, consent, rate-limit, and opt-out flows are finalized.
* Consider digest/quiet-hour options for high-volume email signals.
* Keep an eye on per-(algorithm, ticker) group scaling for very large subscriber sets.

