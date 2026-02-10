# **The Boritrader Documentation: Patch Notes – June 4 to July 4, 2025**   
**Boritrade, LLC**   
**Related Release Version:** 0.7.4   

---

## **Overview**
The Settings page has been completely overhauled—subscription details are now front and center, email and phone fields have been added, password-change flows strengthened, deprecated options removed, and inactive controls labeled as “Coming Soon.”
---

## **Key Updates**

### **Settings Page Redesign**

* **Subscription Management Upgrades**

  * Displays **current plan name**, **billing cycle**, **next renewal date** and **customer ID**.
  * Direct links to **view invoices**, **upgrade/downgrade**, or **cancel**.
  * Bug Fixed: `plan_type` was always `null` due to a model reference error.

* **User Email Display**

  * Shows the user’s **registered email**.
  * **⚠️** Full email-change workflow (verification & notification) is deferred to future update (Issue #124).

* **Phone Number Collection**

  * Added an optional **phone number** field in E.164 format.
  * **⚠️** Verification/SMS workflows will be added in a subsequent release (Issue #114).

* **Password Change Reinforcement**

  * New flow requires **current password** entry, **new password + confirmation**, with partial strength checks on length & complexity.

* **Deprecated Fields Removed**

  * `date_of_birth`, `address`, and `plan_type` controls have been stripped from the UI.
  * Backend data slated for purge per retention policy.
  * Privacy Policy updated to reflect reduced data collection.

* **Inactive Settings Marked**

  * Outdated or non-functional controls are now labeled **“Coming Soon”** with tooltip explanations.


### **Admin Control Panel & Maintenance Tools**

* **New Admin UI**
  * Superuser-only **Control Panel** added under `/sys-admin/maintenance/`.
  * Displays relevant statistics related to signups, signals, system logs, and async controls.

* **Robots.txt Update**
  * Now only allows search indexing on **/register/** and **/login/** pages.


### **Additional Fixes & Minor Enhancements**

* **Styling & UX**

  * Polished Settings and Dashboard pages (e.g. relabeled **Show History** → **Show All History**).
  * Added a subtle **background image** to the Settings page for visual cohesion.

* **Bug Fixes**

  * Corrected broken links on the **registration** page.
  * Addressed dashboard issues around subscription history display and form validation.

---

## **Goals for the Month**
1. **Audit & Finalize Privacy Policy** for full compliance.
2. **Asynchronous Task Handler** — implement background jobs for maintenance tasks (e.g. inactive-user cleanup, reminder emails).
3. **Algorithm Preferences Ticker Update** — add “Popular Tickers” selector, enforce USDT suffix, and tighten ticker validation.
4. **Admin-Only Maintenance Page** — build a superuser dashboard for stats, troubleshooting, and manual interventions.
5. **Notification System Enhancements** — add audible alerts, deeper signal-change analysis, and multi-channel delivery (email/SMS).
6. **Mobile UX Improvements** — optimize Dashboard, Settings, Login, and Registration for smartphones and tablets.
