# **The Boritrader Documentation: Patch Notes — July 4 to August 4, 2025**
**Boritrade, LLC**   
**Related Release Version:** 0.8.4   
---

## **Overview**

1. **Marketing & Registration**: clarified signup UX, new flow routing (trial/promo), and GA4 analytics.
2. **Privacy & Compliance**: site-wide cookie banner + preferences in Settings.
3. **System Logging (Dev)**: critical events now persisted to **SystemLog** for admin visibility.
4. **Notifications UX**: user-facing algorithm **display names** in alerts.

---

## **Key Updates**

### **Marketing Updates — Registration & Analytics** *(Issue #65)*

* **Registration Page UX**

  * Refreshed HTML/CSS to better explain the signup steps and outcomes.
  * Previously register page had two buttons [start free trial] & [pay now]. 
  * Now: app uses url routing to display the relevant registration landing page & corresponding button.

* **Routing & Checkout Logic**

  * `register?trial` → directs to **Free Trial**.
  * `register?promo=CODE` → directs to **Checkout** with promo applied.
  * `register/` (no params) → **Checkout** with promo **optional** (default).

* **Analytics**

  * Added **GA4** event tracking on registration funnel and key interactions.

* **Cookie Compliance**

  * Site-wide cookie banner + consent handling added to **all pages**.
  * **Cookie Preferences** page/section added to **Settings** for granular opt-ins.

* **Test Stability**

  * Resolved a **version-based** issue causing WebSocket tests to fail despite no logic changes (updated tests/deps to restore pass).

---

### **System Logs Implementation**

* **SystemLog Table**

  * Introduced a **SystemLog** database table to persist **critical events** for review and trend analysis in the admin dashboard.

* **Notification Payloads**

  * `notify_user(...)` now uses **algorithm display names** (user-friendly) instead of internal identifiers in pushed updates.

