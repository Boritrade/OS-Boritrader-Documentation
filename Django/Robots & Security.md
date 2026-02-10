# Robots & Other Security Measures
**Boritrade, LLC**  
**Last Updated:** May 5, 2025

This document outlines the implementation and functionality of the current dynamically generated `robots.txt` file in the Django application, as well as the planned updates to enhance its security and flexibility.

---

## **Current Implementation**

The current `robots.txt` file is generated dynamically using a Django view. This approach provides flexibility by allowing the content of the `robots.txt` to be updated programmatically without needing to modify a static file.

### **Code Overview**
From `views_robots.py`:

```python
from django.http import HttpResponse

def robots_txt(request):
    lines = [
        "User-agent: *",
        "Disallow: /",
    ]
    return HttpResponse("\n".join(lines), content_type="text/plain")
```

#### **Explanation**
- **Dynamic Generation**: The `robots.txt` file is generated dynamically at runtime using a Django view.
- **Content**:
  - `User-agent: *`: Applies the rules to all web crawlers.
  - `Disallow: /`: Prevents all crawlers from accessing any part of the site.
- **Response**: The response is served with the correct `text/plain` MIME type, ensuring compatibility with web crawlers.

#### **Integration in `urls.py`**
The view is mapped to the `/robots.txt` URL in the `urls.py` file:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("robots.txt", views.robots_txt, name="robots_txt"),
]
```

#### **Key Advantages**
- **Flexibility**: Content can be customized programmatically based on the environment or application state.
- **Simplicity**: Easy to implement and maintain as part of the Django application.

---


### **2. HTTPS via AWS ALB**

* All HTTP traffic is redirected to HTTPS using the AWS Application Load Balancer (ALB).
* SSL termination is handled by the ALB with certificates managed via **AWS Certificate Manager**.
* Nginx is updated to serve only internal HTTP, no public SSL management required.

---

### **3. Application-Level Rate Limiting**

Implemented using `django-ratelimit` decorators for high-risk endpoints:

| **Endpoint**        | **View**                  | **Limit** | **Methods** | **Purpose**                         |
| ------------------- | ------------------------- | --------- | ----------- | ----------------------------------- |
| `/login/`           | `login_view`              | 10/min    | POST        | Prevent credential stuffing         |
| `/register/`        | `register_view`           | 10/hour   | POST        | Throttle account creation           |
| `/forgot-username/` | `forgot_username_view`    | 3/hour    | POST        | Prevent email enumeration           |
| `/password-reset/`  | `PasswordResetView`       | 3/hour    | POST        | Throttle reset link requests        |
| `/checkout/`        | `create_checkout_session` | 3/min     | All         | Avoid rapid Stripe session creation |

* **Behavior**: `block=True` sends a 429 response for over-limit requests.
* **Middleware**: Custom handling can be applied using `request.limited`.

---

### **4. Django Hardening Measures**

#### Authentication & Sessions

* Enforced password validators.
* Session expiration for inactivity.
* Login redirection via `LOGIN_URL = '/login/'`.

#### Middleware Additions

* `SecurityMiddleware`: Enforces secure headers.
* `RateLimitExceptionMiddleware`: Logs and blocks excessive traffic.
* `AdminIPRestrictionMiddleware`: Restricts admin access.
* `BlockCrawlerMiddleware`: Custom bot-blocking logic.

#### Logging & Monitoring

* **Structured logging** with rotation and filters.
* **Sentry integration** for error tracking and performance tracing.
* **Sensitive data** is excluded from logs (`send_default_pii=False`).

#### WebSockets

* WebSocket traffic proxied securely to port 8001 with upgrade headers.

#### CSRF & Headers

* CSRF middleware enabled.
* Headers like `X-Frame-Options`, `X-Content-Type-Options` set via middleware.

---

### **5. Nginx Security Configurations**

* **HTTPS termination** handled by ALB.
* **Static file handling** and aliasing properly configured.
* **Stripe webhook endpoint** IP-restricted based on Stripe documentation.
* **Directory listing** disabled via `autoindex off`.
* **Sensitive file types** like `.env`, `.json`, and `.csv` explicitly blocked.
* **Caching headers** applied to static file routes.


### **6. Real Client IP Forwarding with ALB**

To properly evaluate and restrict incoming traffic based on IP addresses — particularly for **Stripe webhook requests** — we’ve configured **Nginx to respect the true client IP** passed through AWS ALB via the `X-Forwarded-For` header.

#### 🔐 Why It Matters

* When running behind an **Application Load Balancer (ALB)**, all requests appear to originate from the **ALB’s internal IP (e.g., 10.0.x.x)**.
* This prevents IP-based security rules (like Stripe allowlists) from functioning as intended.
* Stripe sends webhooks from [documented IPs](https://stripe.com/docs/ips), which must be matched against the **original client IP**, not the proxy.

#### ✅ Configuration Summary

In `nginx.conf`, we add:

```nginx
# Trust the ALB as the source of truth for the original client IP
real_ip_header X-Forwarded-For;

# Trust ALB internal subnets only (defined via Terraform)
set_real_ip_from 10.0.1.0/24;
set_real_ip_from 10.0.3.0/24;
```

This tells Nginx:

> “If a request comes from 10.0.1.x or 10.0.3.x, trust the IP in `X-Forwarded-For` as the real client IP.”

Nginx then rewrites `$remote_addr` using the value from `X-Forwarded-For`, allowing rules like `allow 52.15.183.38;` to correctly match Stripe's source IP.

#### 📦 Stripe Webhook Security

In the `/str-webhook/` location block:

```nginx
location /str-webhook/ {
    allow 3.18.12.63;
    allow 3.130.192.231;
    ...
    deny all;

    proxy_pass http://127.0.0.1:8000;
    proxy_set_header X-Real-IP $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

Combined with the `real_ip_header` logic, this setup ensures:

* Only Stripe's official IPs can access the webhook.
* All requests are verified against the **real source IP**, not the ALB.

---

## Future Implementation Plans

### **1. Log and Block Malicious IPs**

* **Monitor Logs**:
  * Track repeated requests from the same IP addresses probing for vulnerabilities.
* **Block IPs**:

  * Implement IP blocking via Django middleware or Nginx configuration.

---

### **2. Secure WebSockets**

* Ensure all WebSocket connections are **encrypted** and **authenticated** to prevent hijacking or eavesdropping.
* Maintain proxy rules in Nginx with proper `Upgrade` and `Connection` headers.

---

### **3. Django-Level Hardening**

1. **Secure Authentication and User Management**

   * Enforce strong password policies.
   * Enable multi-factor authentication (MFA), particularly for admin accounts.
   * Set session timeouts for inactive users.

2. **Middleware Enhancements**

   * Enable `SecurityMiddleware` to enforce:

     * HTTPS redirection: `SECURE_SSL_REDIRECT = True`
     * HSTS headers: `SECURE_HSTS_SECONDS`, `SECURE_HSTS_INCLUDE_SUBDOMAINS`
   * Set secure headers such as:

     * `X-Content-Type-Options`
     * `X-Frame-Options`

3. **Content Security Policy (CSP)**

   * Restrict loading of external scripts, styles, and other resources to reduce XSS risk.

4. **Database Security**

   * Use secure credentials via environment variables.
   * Encrypt sensitive data at rest.
   * Evaluate field-level encryption for high-risk fields.
   * Consider column-based encryption using Django wrappers or PostgreSQL extensions.

5. **Logging and Monitoring**

   * Expand structured logging to capture anomalous and security-related events.
   * Ensure sensitive data is never logged in plaintext.

---

### **4. Nginx-Level Hardening**

1. **SSL/TLS Configuration**

   * Handled by AWS ALB with AWS Certificate Manager.
   * No direct SSL termination in Nginx.

2. **Rate Limiting** *(Planned)*

   * Mitigate brute-force and DoS attacks with Nginx-level rate limits.
   * Example configuration:

     ```nginx
     limit_req_zone $binary_remote_addr zone=one:10m rate=5r/s;

     server {
         location / {
             limit_req zone=one burst=10;
         }
     }
     ```

3. **Security Headers**

   * Apply these headers:

     ```nginx
     add_header X-Frame-Options "DENY";
     add_header X-Content-Type-Options "nosniff";
     add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
     add_header Referrer-Policy "strict-origin-when-cross-origin";
     add_header Permissions-Policy "interest-cohort=()";
     ```

4. **Protect Static and Media Files**

   * Block access to sensitive file types:

     ```nginx
     location ~* \.(json|csv|env|config)$ {
         deny all;
         return 403;
     }
     ```
   * Apply aggressive caching for static content:

     ```nginx
     location /static/ {
         add_header Cache-Control "public, max-age=31536000";
     }
     ```

5. **Prevent Directory Indexing**

   * Disable listing of directory contents:

     ```nginx
     autoindex off;
     ```

6. **DDoS Mitigation**

   * Configure request rate limiting to throttle abusive clients:

     ```nginx
     limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;
     ```



## **System Logging & Audit Events**

Every significant security, billing, or administrative action in Boritrade now writes a row to the `system_logs` table via `SystemLog.objects.create(...)`. This provides an auditable trail for forensic investigation and real-time monitoring.

### 7.1. **Signals** (`signals.py`)

* **`pre_delete` on `User`** → `SubscriptionDeleted`

  * Cancelling Stripe subscriptions and marking the Stripe customer as deleted when a user account is removed.
* **`m2m_changed` on `User.groups` & `User.user_permissions`** → `SecurityPermissionGranted` / `SecurityPermissionRevoked`

  * Tracks when staff are added to or removed from Django groups or granted/revoked individual permissions.
* **`user_logged_in` signal** → `AdminLogin`

  * Records every superuser or staff login, to detect potentially malicious admin access.

### 7.2. **Authentication Views** (`views_auth.py`)

* **Login rate-limit fences** → `Ratelimit`

  * Fired whenever a client exceeds the 10 POST/min limit on `/login/`.
* **Registration rate-limit fences** → `Ratelimit`

  * Fired on exceeding 10 POST/hr to `/register/`.
* **Username-recovery rate-limit fences** → `Ratelimit`

  * Fired on exceeding 3 POST/hr to `/forgot-username/`.
* **Successful registration** → `UserRegistered`

  * Logs every new account, including flags for `trial` or `promo`.
* *(Note: individual login attempts are also captured in the `login_attempts` table via `track_login_attempt`.)*

### 7.3. **Stripe Webhooks** (`stripe_webhooks.py`)

* **Signature verification failure** → `WebhookSignatureFailed`
* **Checkout/session completed** → `SubscriptionCreated`
* **Subscription updated** → `SubscriptionUpdated` (plus `SubscriptionCanceledAtPeriodEnd` when appropriate)
* **Subscription deleted** → `SubscriptionEnded`
* **Invoice payment failed** → `PaymentFailed`

### 7.4. **Outbound Emails** (`emails.py`)

* **Every successful send** in `send_safe_email(...)` → `EmailSent`

  * Captures subject line and recipient address for all transactional emails.

### 7.5. **Access Control Decorators** (`decorators.py`)

* **Unauthorized sys-panel access** → `AccessDenied - Sys Panel`

  * Logs when any non-superuser attempts to hit the `/admin-panel/` UI.

### 7.6. **Password Changes** (`forms.py`)

* **Password reset/change** in `PasswordChangeForm.save()` → `PasswordChanged`

---

## Admin Panel Overview**

### 8.1. **Purpose**

The **Admin Dashboard** (view name: `admin_dashboard`) gives superusers a one-stop security and health check for the platform. It combines:

1. **User Metrics** (counts, signup trends, churn)
2. **Activity Metrics** (daily/monthly active users, algo usage, top tickers)
3. **Recent Logs** (DB-backed events and raw logfile tails)

This allows us to spot spikes in failed logins, subscription churn, payment failures, or suspicious admin access in real time.

---

### 8.2. **Data Panels**

| Section                  | What’s Shown                                                                           |
| ------------------------ | -------------------------------------------------------------------------------------- |
| **User Counts**          | Total, active vs. inactive, banned (disabled)                                          |
| **Sign-Ups**             | Today’s sign-ups, last 30 days total, daily (30 days) & monthly (12 months) breakdowns |
| **DAU / MAU**            | Unique users with a successful login in last 1 day / 30 days                           |
| **Algo Usage (24 h)**    | Number of `SignalResult` per algorithm, ordered by volume                              |
| **Top Tickers (7 d)**    | Most-queried tickers via `Notification` records                                        |
| **Row-Counts (“Sizes”)** | Current table counts for `SignalResult`, `Notification`, `UserProfile`                 |
| **Recent DB Logs**       | Last 50 entries from each of: `SystemLog`, `LoginAttempt`, `PasswordReset`             |
| **Local Logfile Tail**   | Last N lines from local `DEBUG` or `ERROR` log, with graceful fallback in prod         |

---

### 8.3. **Use Cases**

1. **Spot anomalies**
   * A sudden surge in `Ratelimit` or `PaymentFailed` events can indicate brute-force or payment gateway issues.

2. **Easy-Access logs**
   * Click “ERROR” at top right to switch logfile tail from `DEBUG` → `ERROR` level (local feature only)

3. **Audit admin access**
   * All `AdminLogin` and `AccessDenied - Sys Panel` events feed into the “Recent DB Logs” panel.
   
4. **Plan capacity**
   * Use DAU/MAU and algo-usage charts to right-size worker pool and database resources.
