# **Email Notifications & Microsoft 365 SMTP Integration**  
**Boritrade LLC**  
**Last Updated:** April 18, 2025  

This document outlines the email notification system used throughout the Boritrader Django project and the steps required to configure Microsoft 365 to send emails via SMTP using **app passwords**.


## **1. Overview of Email Functionality**

Emails are sent for various user-related events to improve user experience and security. These include:

| **Event**                     | **Purpose**                                                 |
|------------------------------|-------------------------------------------------------------|
| Welcome Email                | Sent to new users after successful registration.            |
| Password Change Confirmation | Sent after a user changes their password.                   |
| Stripe Subscription Emails   | Sent when a user subscribes, updates, cancels, or fails a payment. |


## **2. Microsoft 365 SMTP Setup**

Boritrader uses Microsoft 365 to send transactional emails via SMTP. Microsoft enforces Multi-Factor Authentication (MFA), which means you must create an **App Password** specifically for Django to use.

### **Steps to Enable App Passwords on Microsoft 365**

> 💡 Reference: [Microsoft Support Article](https://answers.microsoft.com/en-us/msoffice/forum/all/app-passwords-for-sign-in-does-microsoft-still/323829cc-69d9-4baf-a95d-f7c5e1fc0919)

1. **Go to Microsoft 365 Admin Center**  
   - Navigate to **Users** → **Active users**  
   - Click on the user account you want to enable  
   - Click **Manage multi-factor authentication**

2. **Enable MFA and App Passwords**  
   - In the MFA portal, check the box next to the account  
   - Click **Enable**, then click **Enforce**

3. **Generate App Password**  
   - Sign in to [office.com](https://office.com) using the same account  
   - Go to **My Account** → **Security Info**  
   - Click **+ Add method** and choose **App Password**  
   - Name it (e.g., `Boritrade SMTP`) and copy the generated password

4. **Update Django settings with the app password**  
   ```python
   EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
   EMAIL_HOST = 'smtp.office365.com'
   EMAIL_PORT = 587
   EMAIL_USE_TLS = True
   EMAIL_HOST_USER = 'noreply@boritrade.net'
   EMAIL_HOST_PASSWORD = config('EMAIL_HOST_PASSWORD')
   DEFAULT_FROM_EMAIL = 'Boritrade [DEVSERVER] <noreply@boritrade.net>'
   ```
   👉 *Ensure the password is present in environment variables.*
blocked
IMAP blocked
Other email apps allowed
Manag

## **3. Core Email Logic: `emails.py`**

All outgoing transactional emails are centralized in `dashboard/emails.py`.

### **Email Utilities**

- `send_safe_email(...)`  
  Internal helper that sends email using both plain text and HTML. Automatically logs delivery or failures.

---

### **Event-specific Email Functions**

| **Function**                                    | **Trigger Event**                                         |
|------------------------------------------------|-----------------------------------------------------------|
| `send_welcome_email(user)`                     | Successful user registration                              |
| `send_password_change_email(user)`             | Password change from user dashboard                       |
| `send_payment_confirmation_email(user)`        | Stripe subscription success                               |
| `send_subscription_updated_email(user, old, new, end_date)` | Subscription updated via Stripe             |
| `send_subscription_canceled_email(user)`       | Subscription canceled                                     |
| `send_payment_failed_email(user)`              | Stripe invoice payment failure                            |
| `send_view_status_change_email(user, ticker, old_status, new_status)` | Ticker view status changed (e.g., BUY → SELL) |


## **4. Where Emails Are Triggered**

| **File**           | **Function**                             | **Email Sent**                                     |
|--------------------|------------------------------------------|----------------------------------------------------|
| `forms.py`         | `RegisterForm.save()`                    | `send_welcome_email`                               |
| `forms.py`         | `PasswordChangeForm.save(user)`          | `send_password_change_email`                       |
| `views_stripe.py`  | `stripe_webhook()` (checkout completed)  | `send_payment_confirmation_email`                  |
| `views_stripe.py`  | `stripe_webhook()` (subscription updated)| `send_subscription_updated_email`                  |
| `views_stripe.py`  | `stripe_webhook()` (subscription deleted)| `send_subscription_canceled_email`                 |
| `views_stripe.py`  | `stripe_webhook()` (payment failed)      | `send_payment_failed_email`                        |
| `views.py`         | `send_view_status_change_email(...)`     | When ticker view status changes (manual trigger)   |


## **5. Best Practices**

### **Environment Variables**

Never commit `EMAIL_HOST_PASSWORD` directly in `settings.py`. Use Django-environ or OS environment variables to manage sensitive info.

---

### **Logging**

All email sends and failures are logged under the `security` logger:

```python
logger = logging.getLogger('security')
```

---

### **Silent Failures & Error Visibility**

#### Silent Failures

Emails are configured to **raise exceptions by default**, which helps catch configuration or delivery issues early during development and deployment.

In Django, email sending methods like `EmailMessage.send()` or `EmailMultiAlternatives.send()` default to `fail_silently=False`, meaning **they will raise an error if sending fails**.

This is handled within `emails.py`:

```python
def send_safe_email(subject, plain_text, html_text, to_email):
    from_email = settings.DEFAULT_FROM_EMAIL
    try:
        msg = EmailMultiAlternatives(subject, plain_text, from_email, [to_email])
        msg.attach_alternative(html_text, "text/html")
        msg.send()  # fail_silently=False is implicit here
        logger.info(f"Sent email to {to_email} - '{subject}'")
    except Exception as e:
        logger.error(f"Error sending email to {to_email} - '{subject}': {str(e)}")
```

Because we **do not override `fail_silently`**, any SMTP errors (e.g., invalid credentials, network issues, misconfigured email host) will raise exceptions and be captured in the logs. This ensures visibility during development and production monitoring.

If there is ever a need to suppress errors in specific scenarios (e.g., optional emails or silent background jobs), explicitly set:

```python
msg.send(fail_silently=True)
```

However, **this is not recommended** for critical notifications like password changes, payment failures, or registration confirmations.

---

### **Reusability**

All transactional emails are constructed using both `plain_message` and `html_message` formats to ensure broad compatibility, improved deliverability, and enhanced accessibility.

The `EmailMultiAlternatives` class is used to include both content types in a single message:

```python
msg = EmailMultiAlternatives(subject, plain_text, from_email, [to_email])
msg.attach_alternative(html_text, "text/html")
msg.send()
```

#### **Benefits of Dual Formatting**

| **Reason**                  | **Description**                                                                 |
|-----------------------------|---------------------------------------------------------------------------------|
| **Client Compatibility**    | Ensures messages are viewable in all email clients, including those that do not support HTML. |
| **Spam Filter Avoidance**   | Reduces the likelihood of emails being flagged as spam by including a plain-text version.     |
| **Accessibility Compliance**| Supports screen readers and accessibility tools that rely on plain-text content.              |
| **Graceful Degradation**    | Provides a fallback in cases where HTML content is blocked or fails to render.                |


## **Troubleshooting**

### Error: SMTP Authentication Unsuccessful

```log
2025-04-20 16:49:41 ERROR emails Error sending email to appdev@boritrade.net - 'Your Boritrader Subscription Was Updated': 
(535, b'5.7.139 Authentication unsuccessful, SmtpClientAuthentication is disabled for the Tenant. 
Visit https://aka.ms/smtp_auth_disabled for more information.')
```

#### Cause  
SMTP AUTH is disabled for the sending account (`noreply@boritrade.net`). Microsoft 365 disables SMTP AUTH by default for new tenants.

---

#### Resolution Steps

**1. Enable SMTP AUTH via Microsoft 365 Admin Center**  
1. Go to [https://admin.microsoft.com](https://admin.microsoft.com)  
2. Navigate to **Users** → **Active users**  
3. Select `noreply@boritrade.net`  
4. In the right panel, choose **Mail** → **Manage email apps**  
5. Ensure **Authenticated SMTP** is enabled  
6. (Optional) Leave **Outlook on the web** and **Exchange Web Services** enabled for browser access  
7. Save changes  

> If the “Mail” tab is not visible, assign the **Exchange Administrator** role under **Roles** → **Admin roles** → **Exchange administrator**.

**2. (If Required) Enable SMTP AUTH Tenant‑wide via PowerShell**  
```powershell
# Enable SMTP AUTH globally
Set-TransportConfig -SmtpClientAuthenticationDisabled $false

# Enable SMTP AUTH for the specific mailbox
Set-CASMailbox -Identity noreply@boritrade.net -SmtpClientAuthenticationDisabled $false
```
> Requires Exchange Administrator or Global Administrator permissions.

---

### Best Practices for `noreply@boritrade.net`

| Setting                     | Recommendation                          |
|-----------------------------|------------------------------------------|
| Authenticated SMTP          | ✅ Enabled (required for sending)       |
| Outlook on the web          | ✅ Optional (for inbox access)          |
| Exchange Web Services       | ✅ Optional                             |
| POP, IMAP, MAPI, ActiveSync | ❌ Disabled                             |
| Shown in Global Address List| ❌ Hidden                               |
| Mailbox License             | ✅ Exchange Online Plan 1 (send‑only)   |


## Enabling Tenant‑Wide SMTP AUTH via Azure Cloud Shell

Because local environments often encounter compatibility or policy blocks, Azure Cloud Shell provides a turnkey way to flip the tenant‑wide SMTP AUTH flag. Follow these steps whenever ExchangeOnlineManagement cmdlets are not loading locally:

1. **Ensure an Azure Subscription**  
   - Sign up for a free Azure account at https://azure.microsoft.com/free using any Global/Exchange Admin account in the same Azure AD tenant.  

2. **Launch Cloud Shell**  
   - Navigate to https://shell.azure.com and select the **PowerShell** environment.  
   - Accept the prompts to provision storage if this is the first run.  

3. **Authenticate to Exchange Online**  
   ```powershell
   Connect-ExchangeOnline -UseDeviceAuthentication
   ```
   - Copy the device code and visit https://microsoft.com/devicelogin to complete sign‑in/MFA.  

4. **Enable SMTP AUTH Tenant‑Wide**  
   ```powershell
   Set-TransportConfig -SmtpClientAuthenticationDisabled $false
   ```

5. **Verify the Change**  
   ```powershell
   Get-TransportConfig | Select-Object SmtpClientAuthenticationDisabled
   # Expected: False
   ```

6. **Disconnect**  
   ```powershell
   Disconnect-ExchangeOnline
   exit
   ```
   —closing the browser tab suffices to end the session.


## Local PowerShell & Linux Challenges

### 8.1 Linux + PowerShell Core  
- **Module Compatibility**  
  – ExchangeOnlineManagement v3 preview requires .NET 8 runtime **and** SDK; many Linux hosts only had the runtime, leading to `System.Runtime, Version=8.0.0.0` load failures.  
- **WinRM & RPS**  
  – Legacy `Connect‑ExchangeOnline -UseRPSSession` relies on WinRM Basic‑auth, which is disabled on most Linux and in Azure’s backend.  

### 8.2 Windows PowerShell  
- **REST‑only v3 Module**  
  – Does not include `Get‑TransportConfig`/`Set‑TransportConfig`; those live only in the deprecated remote‑PSSession (V1) interface.  
- **Execution Policy & WinRM**  
  – Execution policies often block imported scripts without `RemoteSigned` or `Bypass`.  
  – RPS fallback (`-UseRPSSession`) fails if WinRM Basic‑auth or trusted hosts are disallowed, and Microsoft is deprecating that interface.  


## Why Azure Cloud Shell Is Recommended

| Challenge                     | Local PowerShell or Linux Host                                            | Azure Cloud Shell                                    |
|-------------------------------|----------------------------------------------------------------------------|-------------------------------------------------------|
| **Module & .NET Dependencies**| Manual install of PowerShell, .NET 8 SDK/runtime, and Exchange module      | Fully managed, up‑to‑date environment                 |
| **Legacy Cmdlet Access**      | Requires `-UseRPSSession` and WinRM Basic‑auth (deprecated / blocked)     | V3 module’s REST‑based cmdlets load without RPS fallbacks |
| **Execution & Policy**        | Execution policies, WinRM config, and network rules must be adjusted      | No local policy or network changes required           |
| **Authentication Flow**       | GUI dialogs (WinPS), console prompts (pwsh), device codes                  | Built‑in device‑code flow in Cloud Shell              |

**Conclusion:** Azure Cloud Shell eliminates local setup and policy hurdles, providing a reliable, one‑step method to enable tenant‑wide SMTP AUTH and immediately resolve the 535 authentication errors.```

