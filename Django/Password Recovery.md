**Password Recovery & Reset Flow**  
**Boritrade LLC**  
**Last Updated:** April 20, 2025  

This document describes the end‑to‑end password recovery process in the Boritrade Django app: from the user requesting a reset link, through email delivery, to setting a new password. It covers the high‑level user flow, URL configuration, views, forms, and all relevant template filenames—with only minimal code references.

---

## 1. Overview

When a user forgets their password, we leverage Django’s built‑in class‑based views to:

1. **Collect their email** (PasswordResetView)  
2. **Send a one‑time reset link** via both plain‑text and HTML email  
3. **Render a confirmation** (“Check your email”) page  
4. **Validate the link token** (PasswordResetConfirmView)  
5. **Allow them to set a new password**  
6. **Render a final success** (“Password reset complete”) page  

No custom email‑sending helper is needed for the reset‑link flow; Django handles templating and dispatch via your SMTP backend.

## **2. Key Concepts & View Styles**  

- **Function‑Based View (FBV)**  
  A simple Python function that handles `request` and returns a `Response`. Custom logic and form handling are written imperatively.  

- **Class‑Based View (CBV)**  
  A Python class whose HTTP methods (`get()`, `post()`) are defined or inherited. Common patterns (forms, lists, details) are provided by Django’s generic CBVs. Configuration is done via class attributes (e.g. `template_name`).

- **Generic CBVs**  
  Pre‑built views like `FormView`, `TemplateView`, `ListView`, etc., that encapsulate common behaviors. Django’s auth views—for password reset, login, logout—inherit from these to minimize boilerplate.


## 2. User Flow Overview

1. **User clicks “Forgot your password?”** link  
2. **Password‑reset request page** (`password-reset/`):  
   - User enters their email and submits.  
3. **Email sent** (if the email matches an account):  
   - Link to `reset/<uidb64>/<token>/`.  
4. **“Check your inbox” page** (`password-reset/done/`) shown regardless of match.  
5. **User clicks link** in email → lands on **reset form** (`reset/<uidb64>/<token>/`):  
   - Enters and confirms new password.  
6. **Final confirmation page** (`reset/done/`):  
   - “Your password has been set. You may now log in.”

### Details
1. **Reset Requested**  
   - User clicks **Forgot your password?**  
   - **PasswordResetView** renders `password_reset_request.html`.  

2. **Email Submission**  
   - User enters their email.  
   - `PasswordResetForm` is validated.  
   - If a matching account exists, a token and UID are generated:  
     - **`uidb64`** is the user’s primary key encoded in URL‑safe Base64.  
     - **`token`** is produced by Django’s `PasswordResetTokenGenerator`, incorporating the user’s last login timestamp and a secret key, ensuring one‑time use and expiration.  
   - An email is sent containing a link like:  
     ```
     https://your-domain.com/reset/MQ/set-token-xyz/
     ```  
   - Regardless of account existence, `password_reset_request_done.html` is rendered to avoid enumeration.

3. **Email Delivered**  
   - Both plain‑text and HTML parts use the same context (`user`, `protocol`, `domain`, `uidb64`, `token`).  
   - The link points to `password_reset_confirm` view with URL parameters.

4. **Link Visited**  
   - **PasswordResetConfirmView** decodes `uidb64`, fetches the user, and validates the token.  
   - If valid, `password_reset_form.html` is displayed with `SetPasswordForm` (fields `new_password1`, `new_password2`).  
   - Upon submission, the password is updated (`user.set_password()`), and the user is redirected to `password_reset_complete`.

5. **Completion**  
   - **PasswordResetCompleteView** renders `password_reset_form_complete.html`, indicating success.  
   - The user may now log in with their new password.


## 3. URL Configuration

_All entries go in your main `config/urls.py` (or similar)_:  

```python
from django.urls import path, reverse_lazy
from django.contrib.auth import views as auth_views
from dashboard import views_auth

urlpatterns = [
    # 1) Request reset link
    path("password-reset/",
        ratelimit(key='ip', rate='3/h', method='POST', block=True)(
            auth_views.PasswordResetView.as_view(
                template_name="recovery/password_reset_request.html",
                email_template_name="recovery/emails/password_reset_email.txt",
                html_email_template_name="recovery/emails/password_reset_email.html",
                subject_template_name="recovery/emails/password_reset_email_subject.txt",
                success_url=reverse_lazy('password_reset_done'),
            )
        ), name="password_reset"),

    # 2) Check‑your‑inbox
    path(
        "password-reset/done/",
        auth_views.PasswordResetDoneView.as_view(
            template_name="recovery/password_reset_request_done.html"
        ),
        name="password_reset_done",
    ),

    # 3) Set new password from email link
    path(
        "reset/<uidb64>/<token>/",
        auth_views.PasswordResetConfirmView.as_view(
        views_auth.SafePasswordResetConfirmView.as_view(
            template_name="recovery/password_reset_form.html",
            success_url=reverse_lazy('password_reset_complete'),
        ),
        name="password_reset_confirm",
    ),

    # 4) Final complete page
    path(
        "reset/done/",
        auth_views.PasswordResetCompleteView.as_view(
            template_name="recovery/password_reset_form_complete.html"
        ),
        name="password_reset_complete",
    ),

    # (plus your forgot‑username FBV)
    path("forgot-username/", views_auth.forgot_username_view, name="forgot_username"),
]
```

- **`template_name`**  
  Specifies which HTML template is rendered for each view.  
- **`email_template_name` / `html_email_template_name`**  
  Point to the plain‑text and HTML versions of the reset email.  
- **`subject_template_name`**  
  (Optional) Allows templating of the email’s subject line.  
- **`success_url`**  
  The redirect destination upon successful form submission.  
- **`name`**  
  The URL’s reverse lookup key, used in `{% url %}` tags and redirects.

Those CBVs handle form validation, token generation/validation, and email dispatch via the configured SMTP backend. No additional email‑sending code is required for the reset‑link flow.


## 4. Views & Forms

- **PasswordResetView**  
  Uses `PasswordResetForm` (built‑in) to collect an email and, if valid, send the reset email.  

- **PasswordResetConfirmView**  
  Uses `SetPasswordForm` (alias `PasswordResetConfirmForm`) to validate and set the new password.  

- **PasswordResetDoneView** & **PasswordResetCompleteView**  
  Simple `TemplateView`‑based classes to render confirmation pages.

- **PasswordChangeView** (separate, for logged‑in users)  
  Uses `PasswordChangeForm`; not part of the “forgot password” flow.

You do **not** need to write custom forms for the reset flow unless you want to override validation or widgets.  


## 5. Templates

All HTML pages live under `templates/recovery/`:

| **Template**                          | **Purpose**                      |
|---------------------------------------|----------------------------------|
| `password_reset_request.html`         | Collect user email               |
| `password_reset_request_done.html`    | “Check your inbox” confirmation  |
| `password_reset_form.html`            | Enter new password & confirm     |
| `password_reset_form_complete.html`   | “Password reset complete”        |
| *(plus)*                              |                                  |
| `forgot_username.html`                | Collect email for username reminder |
| `forgot_username_done.html`           | “If your email matches…” page    |

Each template reuses the same layout/CSS as your login/register pages, swapping in the appropriate `<form>` blocks.

Email templates under **`templates/recovery/emails/`**:

| **Template**                          | **Purpose**                      |
|---------------------------------------|----------------------------------|
- **`password_reset_email.txt`**  
- **`password_reset_email.html`**  
- **`password_reset_email_subject.txt`**


## **5. Views & Forms**  

| **View**                         | **Form**                 | **Purpose**                                         |
|----------------------------------|--------------------------|-----------------------------------------------------|
| `PasswordResetView`              | `PasswordResetForm`      | Collect email and send reset link email.            |
| `PasswordResetDoneView`          | —                        | Inform user to check their inbox.                   |
| `PasswordResetConfirmView`       | `SetPasswordForm`        | Validate token and collect new password.            |
| `PasswordResetCompleteView`      | —                        | Confirm reset completion.                           |
| `PasswordChangeView` (opt.)      | `PasswordChangeForm`     | In‑app password change for authenticated users.     |
| Custom FBV: `forgot_username_view`| `ForgotUsernameForm`    | Email username reminder; separate from reset flow.  |
| `SafePasswordResetConfirmView` (CBV)  | `SetPasswordForm` | Validates token, sets new password, **and** calls `send_password_change_email(user)` on success. |


## 6. Email Templates

Under `templates/recovery/emails/`:

- **`password_reset_email.txt`**  
  Plain‑text email with the reset link.  

- **`password_reset_email.html`**  
  HTML email version, styled button linking to the reset URL.  

- **`password_reset_email_subject.txt`**  
  Plain-text email subject header. 

Django’s `PasswordResetView` automatically loads both, merges them into a single `EmailMultiAlternatives` message, and sends via your SMTP settings.


## 7. Customization Points

- **Custom form classes**  
  If you need extra fields or widget tweaks, subclass `PasswordResetForm` or `SetPasswordForm` and pass `form_class=` to `.as_view()`.  

- **Subclassing the CBVs**  
  To hook in your `send_password_change_email(user)` after a reset, subclass `PasswordResetConfirmView` and override `form_valid()`.  

- **Template context**  
  Use `extra_context={…}` on `as_view()` to inject additional variables into your templates.
