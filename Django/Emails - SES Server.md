**Email Notifications & Amazon SES Integration**  
**Boritrade LLC**  
**Last Updated:** April 21, 2025  

This document outlines the email notification system used throughout the Boritrader Django project and the steps required to configure Amazon SES to send emails via SMTP (or API) using verified domains and credentials.

---

## 1. Overview of Email Functionality

Emails are sent for various user‑related events to improve user experience and security:

| **Event**                     | **Purpose**                                                 |
|-------------------------------|-------------------------------------------------------------|
| Welcome Email                 | Sent to new users after successful registration.            |
| Password Change Confirmation  | Sent after a user changes their password.                   |
| Stripe Subscription Emails    | Sent when a user subscribes, updates, cancels, or fails a payment. |

---

## 2. Amazon SES SMTP Setup

Boritrader uses Amazon SES to send transactional emails. SES supports both SMTP and API (via `django-ses`). Below are the SMTP‑based steps; API steps are noted in §2B.

### 2A. Verify Your Domain

1. **SES Console → Verified identities → Create identity → Domain**  
2. Enter `your.domain.net`, enable DKIM.  
3. Copy the **three CNAME** DKIM records and add to your DNS.  
4. (Optional) Under **Mail from domain**, specify `mail.your.domain.net`, then add its **MX** and **TXT** (SPF) records.  
5. Wait for SES to show **Verified**.

### 2B. Obtain SMTP Credentials

1. SES Console → **SMTP settings** → **Create SMTP credentials**.  
2. Note the **Username** (`ses-smtp-user…`) and **Password**.  
3. Store them securely (e.g. environment variables).

### 2C. Configure Django (SMTP Backend)

```python
# settings.py

EMAIL_BACKEND    = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST       = 'email-smtp.us-east-2.amazonaws.com'  # SES region endpoint
EMAIL_PORT       = 587
EMAIL_USE_TLS    = True

EMAIL_HOST_USER     = config('SES_SMTP_USERNAME')        # ses-smtp-user
EMAIL_HOST_PASSWORD = config('SES_SMTP_PASSWORD')        # matching secret

DEFAULT_FROM_EMAIL  = 'Name <noreply@your.domain.net>'
SERVER_EMAIL        = DEFAULT_FROM_EMAIL
```

### 2D. (Alternative) Configure Django (API via `django-ses`)

```bash
pip install django-ses
```

```python
# settings.py

INSTALLED_APPS += ['django_ses']

EMAIL_BACKEND           = 'django_ses.SESBackend'
AWS_ACCESS_KEY_ID       = config('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY   = config('AWS_SECRET_ACCESS_KEY')
AWS_SES_REGION_NAME     = 'us-east-2'
AWS_SES_REGION_ENDPOINT = 'email.us-east-2.amazonaws.com'

DEFAULT_FROM_EMAIL      = 'Name <noreply@byour.domain.net>'
SERVER_EMAIL            = DEFAULT_FROM_EMAIL
```

---

## 3. Core Email Logic: `dashboard/emails.py`

All outgoing emails are centralized here.

### Email Utilities

- `send_safe_email(subject, plain_text, html_text, to_email)`  
  Sends both text and HTML versions using SES. Attaches the SES configuration set header and logs success or failure.

- `html_email_body(user, body_content)`
  Wraps HTML email content in a branded, styled template.

- `plain_email_body(user, body_content)`
  Formats plain-text email with basic branding and support footer.

### Event‑Specific Functions

| **Function**                                  | **Trigger Event**                              |
| --------------------------------------------- | ---------------------------------------------- |
| `send_welcome_email(user)`                    | Successful user registration                   |
| `send_password_change_email(user)`            | Password changed via settings or recovery flow |
| `send_payment_confirmation_email(user)`       | Stripe subscription success                    |
| `send_subscription_updated_email(user, …)`    | Subscription updated via Stripe                |
| `send_subscription_canceled_email(user)`      | Subscription canceled                          |
| `send_subscription_ended_email(user)`         | Subscription fully ended                       |
| `send_subscription_expiring_soon_email(user)` | Subscription nearing its end date              |
| `send_payment_failed_email(user)`             | Stripe payment failure                         |
| `send_view_status_change_email(user, …)`      | Ticker view status changed                     |
| `send_username_recovery_email(user)`          | Username recovery requested                    |
| `send_account_deleted_email(user)`            | Account permanently deleted                    |

---

## 4. Where Emails Are Triggered

| **File**            | **Function**                              | **Email Sent**                                  |
| ------------------- | ----------------------------------------- | ----------------------------------------------- |
| `forms.py`          | `RegisterForm.save()`                     | `send_welcome_email`                            |
| `forms.py`          | `PasswordChangeForm.save(user)`           | `send_password_change_email`                    |
| `views_stripe.py`   | `stripe_webhook()` (checkout)             | `send_payment_confirmation_email`               |
| `views_stripe.py`   | `delete_account_view()`                   | `send_account_deleted_email`                    |
| `views_webhooks.py` | `stripe_webhook()` (subscription updated) | `send_subscription_updated_email`               |
| `views_webhooks.py` | `stripe_webhook()` (subscription cancel)  | `send_subscription_canceled_email`              |
| `views_webhooks.py` | `stripe_webhook()` (subscription ended)   | `send_subscription_ended_email`                 |
| `views_webhooks.py` | `stripe_webhook()` (subscr. ending soon)  | `send_subscription_expiring_soon_email`         |
| `views_webhooks.py` | `stripe_webhook()` (payment failed)       | `send_payment_failed_email`                     |
| `views.py`          | `send_view_status_change_email(...)`      | Ticker view status changes                      |
| `views_auth.py`     | `forgot_username_view()`                  | `send_username_recovery_email(user)`            |
| `views_auth.py`     | `CustomPasswordResetConfirmView()`        | `send_password_change_email(user)`              |
| `urls.py`           | `password-reset`                          | reset token - see: `templates/recovery/emails/` |

### Username & Password Recovery-Related Emails
*See `Password Recovery` Documentation for more details.*

#### 1. Forgot Username  
- **Endpoint & View**  
  - `POST /forgot-username/` → `forgot_username_view` (FBV)  
- **User Flow**  
  1. Renders `recovery/forgot_username.html`  
  2. On valid email, calls `send_username_recovery_email(user)`  
  3. Renders `recovery/forgot_username_done.html`  
- **Email Details**  
  - **Function**: `dashboard/emails.send_username_recovery_email(user)`  
  - **Subject**: `Your Boritrade Username`  
  - **Content**: Plain-text and HTML bodies defined directly in `emails.py`

---

#### 2. Password Reset Request  
- **Endpoint & View**  
  - `GET/POST /password-reset/` → `PasswordResetView` (CBV)  
- **User Flow**  
  1. Renders `recovery/password_reset_request.html`  
  2. Validates email via built-in `PasswordResetForm`  
  3. Generates `uidb64` + `token`, sends reset-link email  
  4. Redirects to `recovery/password_reset_request_done.html`  
- **Email Details**  
  - **Subject**: from one-line template `recovery/emails/password_reset_subject.txt`  
  - **Body (plain-text)**: `recovery/emails/password_reset_email.txt`  
  - **Body (HTML)**: `recovery/emails/password_reset_email.html`  
  - **Dispatch**: handled automatically by `PasswordResetView` via `EmailMultiAlternatives` and SES

---

#### 3. Password Change Confirmation  
- **Endpoint & View**  
  - `POST /reset/<uidb64>/<token>/` → `SafePasswordResetConfirmView` (subclass of `PasswordResetConfirmView`)  
- **User Flow**  
  1. Validates `uidb64` and `token`, renders `recovery/password_reset_form.html` with `SetPasswordForm`  
  2. On submission, `form_valid()` updates `user.set_password()`  
  3. Redirects to `recovery/password_reset_form_complete.html`  
  4. Calls `send_password_change_email(user)`  
- **Email Details**  
  - **Function**: `dashboard/emails.send_password_change_email(user)`  
  - **Subject**: `Your Boritrade Password Was Changed`  
  - **Content**: Plain-text and HTML bodies defined in `emails.py`

---

## 5. Best Practices

### Environment Variables

- Never commit secrets.  
- Use `django-environ` or OS env vars for `SES_SMTP_USERNAME`, `SES_SMTP_PASSWORD`, `AWS_*` keys.

### Logging & Failures

```python
logger = logging.getLogger('security')
```

- Use default `fail_silently=False` to catch errors.  
- In `send_safe_email`, exceptions are logged as errors for visibility.

### Reusability

- Always send both plain‑text and HTML (`EmailMultiAlternatives`).  
- Benefits: compatibility, deliverability, accessibility, graceful degradation.

---

## 6. Troubleshooting

### SES Sandbox vs. Production

- **Sandbox**: can only send to verified addresses; 200 messages/24 hrs; 1 msg/sec.  
- **Production**: send to any recipient; higher quotas (e.g. 50,000 msgs/day); request increases as needed.

_Request removal from sandbox via SES Console → Account dashboard → Production access._

### Common Errors

- **Authentication failures**: ensure SMTP creds are correct and TLS port 587 is open.  
- **“Mail from domain not verified”**: confirm you added MX/SPF for custom MAIL FROM.  
- **Permission issues** (API): IAM user/role needs `ses:SendEmail`, `ses:SendRawEmail`, plus read on identities.

---

## 7. Next Steps

1. **Terraform**: see the separate Terraform module for SES provisioning (domain, DKIM, MAIL FROM, DNS, SNS, config‑set).  
2. **Bounce Handling**: implement SNS subscription to `/ses_feedback/` endpoint, parse bounces/complaints, and suppress addresses.  
3. **Monitoring**: set CloudWatch alarms on bounce/complaint rates and quota usage.
