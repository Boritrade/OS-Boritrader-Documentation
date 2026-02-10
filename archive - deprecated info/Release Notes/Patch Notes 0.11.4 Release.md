# **The Boritrader Documentation: Patch Notes — October 4 to November 4, 2025**

**Boritrade, LLC**
**Related Release Version:** 0.11.4

---

## **Overview**

**Focus:** refreshed **footer & socials**, stronger **email delivery** (proper Reply-To and environment-aware sending), and more reliable **Stripe webhook processing**.

---

## **Key Updates**

### 1) **Footer & Socials**

* Added social media links to the site footer and refined icon sizing/alignment.
* Updated the Discord invite link and finalized footer styling.

---

### 2) **Email Delivery Strengthening**

* Added a proper **Reply-To** header to outbound emails so responses reach the intended inbox.
* Corrected input handling: `reply_to` is now passed as a list/tuple as required.
* Implemented **environment-aware sending** to ensure development vs. production routes use the correct identities.

---

### 3) **Stripe Webhooks & Subscriptions**
Bug Fix: A configuration issue in our payment webhook caused Stripe to re-send certain events. Our system treated those as new and sent repeat emails. This was a notification glitch only—no duplicate charges were made, and your account data remains secure.

* Fixed a database truncation error (**“value too long for type character varying(20)”**) encountered during subscription updates.
* Standardized storage of plan labels to prevent retry loops and ensure consistent display in account pages.

