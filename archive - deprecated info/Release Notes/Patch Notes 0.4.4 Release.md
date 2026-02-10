# **The Boritrader Documentation: Patch Notes - March 5 - April 4, 2025**  
**Boritrade, LLC**  
**Related Release Version:** 0.4.4  

---

## **Overview**  

1. **Launched Stripe Full-Service Integration** — subscriptions, trials, payments, and webhook-based account management.  
2. **Subscription System Upgrade** — trial banners, subscription status displays, and new checkout flows added.  
3. **Frontend Enhancements** — new site footer, UI/UX improvements to login, registration, dashboard, and settings pages.  
4. **Backend & Testing Improvements** — increased test coverage for registration, login, and subscriptions; improved error handling.  
5. **Infrastructure & Deployment Stability** — healthcheck improvements, Redis endpoint fixes, dependency version locking.  

---

## **Key Updates**  

### **Stripe Full-Service Billing & Subscription System**  

- **Stripe Integration:**  
  - Full end-to-end Stripe integration for subscription management, including trials, checkout sessions, and webhook handling.  
  - Added ghost trial users and payment metadata (e.g., username) for better customer tracking.

- **Subscription Management:**  
  - Subscriptions, cancellations, and renewals are now handled automatically based on Stripe webhooks.  
  - Moved subscription management to a dedicated section of the site for better organization.  
  - Displayed subscription statuses in the site header for real-time user feedback.

- **Checkout Improvements:**  
  - Prevented multiple active checkouts for a single user.  
  - Improved error handling for webhook failures to avoid account sync issues.

---

### **Frontend Enhancements**  

- **New Footer Added:**  
  - Introduced a new footer on all pages with links to **boritrade.net**, terms, privacy policies, and disclaimers.

- **Login & Registration Redesign:**  
  - Added logos to the login and register pages.  
  - Added Terms & Conditions acceptance on the register page.  
  - Added Checkout and Free Trial buttons to encourage user activation.  
  - Automatically redirected users post-registration to settings to avoid "no API key found" errors.

- **Dashboard Updates:**  
  - Made the Trade Logger optional via a toggle button to declutter the dashboard.  
  - Fixed alignment issues in the dashboard view status section.  

- **Header & Trial Notifications:**  
  - Added a new header image and styled trial banners for better visual distinction.

- **Miscellaneous UI Enhancements:**  
  - Added background styling to login, improved dashboard borders, and title consistency across pages.  
  - Added favicon to site for a more professional look.

---

### **Backend & Infrastructure Improvements**  

- **Improved Healthchecks:**  
  - Fixed Docker and GitHub Actions healthcheck timeout logic:
    - Docker now waits 30 seconds before marking a container healthy/unhealthy.  
    - GitHub Actions allows 125 seconds (4 attempts) before failing deployments.
  - Adjusted ALB healthcheck settings to avoid unnecessary downtime.  
  - Deleted redundant extended healthcheck code for Redis to simplify the deployment pipeline.

- **Redis Endpoint Fixes:**  
  - Hardcoded Redis connection string (`host='redis'`) inside the container to fix environment variable resolution issues in ECS Fargate.

- **Dependency Version Locking:**  
  - Locked Django version to **5.0.2** to avoid PostgreSQL compatibility issues during upgrades (Django 5.1+ requires PostgreSQL 14+).  
  - Locked critical dependency versions based on the last successful build for better CI/CD reliability.

- **Error Handling Improvements:**  
  - Improved login redirect handling for users who are already authenticated.  
  - Added better error responses for subscription syncing and user profile creation issues during registration.

---

## **Goals for the Month**  
1. **Deploy a Dedicated Development Environment** — set up a separate AWS infrastructure for testing and staging alongside production.  
2. **Email Notifications Integration** — configure the application to send emails for key events such as registration, payment confirmation, and important account actions.  
3. **Account Recovery Workflow** — build a secure process for users to recover access to their accounts (password reset, lost access support).  