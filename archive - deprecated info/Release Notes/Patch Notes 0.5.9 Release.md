# **The Boritrader Documentation: Patch Notes - April 4 - May 9, 2025**  
**Boritrade, LLC**  
**Related Release Version:** 0.5.4 -> 0.5.9  

---

## **Overview**

1. **Email Notifications** system launched — with templates, transactional triggers, and SMTP configuration (migrated from Outlook to AWS SES).
2. **Account Recovery System** implemented — password reset and username recovery via email now live.
3. **Login & Registration Security** enhanced — rate limiting for form submissions and user login attempts.
4. **New AWS Development Environment** set up — mirroring production for safer, isolated testing.
5. **Testing Coverage Expanded** — new tests for email flows, password resets, and rate limits.

---

## **Key Updates**

### **Email Notifications System (SES Integration)**

* **SMTP Configuration:**

  * Switched from Outlook to **Amazon SES** for email delivery due to reliability and permissions issues with Microsoft tenancy.
  * Configured secure SMTP credentials for `noreply@boritrade.net` and added headers for mail authentication.

* **Email Sending Logic Implemented:**

  * Introduced a centralized `emails.py` module to organize transactional email logic.
  * Wrapped all email-sending in a safe `send_safe_email()` helper to prevent user-facing failures.

* **Trigger Events for Emails:**

  * **Welcome Email** — sent on successful registration.
  * **Payment Confirmation** — sent after successful Stripe payment.
  * **Password Change Notification** — sent upon password updates.
  * **Password Reset Email** — customized and extended from Django default.
  * **Username Recovery Email** — added for forgotten usernames.
  * **(Planned)** View Status Notifications — to alert users on significant trading signal changes (e.g., BTCUSD → SELL).

* **Template Support:**
  HTML email templates created for all major email types, with improved branding and formatting.

---

### **Account Recovery & Security Enhancements**

* **Account Recovery Workflow:**

  * Added password **and** username recovery via email.
  * Streamlined UX and validated flows with full test coverage.

* **Rate Limiting Improvements:**

  * Login and registration forms are now protected by request throttling to prevent brute-force attacks.
  * Confirmed with new test cases for rate limit enforcement.

---

### **New Development Environment (AWS)**

* **Production-Parity Staging Setup:**

  * Cloned the production ECS/Fargate infrastructure into a separate AWS account.
  * Deployed dev-specific secrets and environment variables using SSM.
  * Isolated CI/CD access and IAM roles to ensure safe deployment testing.


---

### **Testing, Infrastructure, & Cleanup**

* **Expanded Test Coverage:**

  * Added unit tests for:

    * Password reset emails
    * Login redirect behaviors
    * Registration form UX
    * Rate limit enforcement


## **Goals for the Month**

1. **Deploy View Status Email Alerts** for major ticker signal changes.
3. **Implement Email Template Styling** using Django HTML rendering for improved branding.
4. **Evaluate Async Email Sending** (via Celery or background tasks) for improved UX performance.
5. **Prepare Algorithm Runner Refactor** — moving from per-user instances to centralized service runner that processes all tickers/algorithms globally for improved scalability. 
6. **Remove API Key registration restriction** - use centralized service runner instead of individual user keys