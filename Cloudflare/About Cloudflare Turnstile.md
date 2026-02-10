# Cloudflare Turnstile (CAPTCHA) – Setup & Operations
**Boritrade LLC**   
**Last Updated:** Sept 7, 2025   

## Overview

We added **Cloudflare Turnstile** to protect account creation (and optionally login) from automated scripts. Turnstile is a privacy-friendly CAPTCHA. It requires a **client-side widget** and **mandatory server-side validation** via Cloudflare’s Siteverify API. Tokens are **single-use** and **expire after \~5 minutes**; the server must validate every token before accepting the request.

This document covers: configuration, code touch points, testing, operations, and troubleshooting.

---

## 1) Prerequisites

* Cloudflare account with a **Turnstile widget** created (type: **Invisible**).
* Backend egress to `https://challenges.cloudflare.com` (allow outbound 443).
* Our app runs behind **AWS ALB + WAF**; we do **not** proxy through Cloudflare (Turnstile works fine in this mode).

---

## 2) Configuration (Django settings & env)

Set these environment variables:

```bash
# .env
TURNSTILE_SITE_KEY=************************
TURNSTILE_SECRET_KEY=************************
```

In `settings.py` we already compute the canonical site URL and whether Turnstile is enabled:

```python
# existing
SITE_URL_MAP = {
    "development": "http://127.0.0.1:8000",
    "staging":     "https://dev-boritrader.net",
    "production":  "https://boritrader.net",
}
SITE_URL = SITE_URL_MAP.get(config("SENTRY_ENV"), SITE_URL_MAP["production"]).rstrip("/")

TURNSTILE_SITE_KEY   = config("TURNSTILE_SITE_KEY")
TURNSTILE_SECRET_KEY = config("TURNSTILE_SECRET_KEY")
TURNSTILE_ENABLED    = bool(TURNSTILE_SITE_KEY and TURNSTILE_SECRET_KEY)
TURNSTILE_BYPASS_IN_DEBUG = True  # dev convenience
```

For hostname checks, we derive the expected hostname where needed:

```python
from urllib.parse import urlparse
TURNSTILE_EXPECTED_HOSTNAME = urlparse(SITE_URL).hostname
```

> **Rollback switch:** set `TURNSTILE_ENABLED=False` (or unset keys) to bypass in lower envs. In production, always validate.

---

## 3) Template changes (Register page)

We load the Turnstile script and render **one** invisible widget. The widget injects a hidden input named `cf-turnstile-response` on success.

```html
<script src="https://challenges.cloudflare.com/turnstile/v0/api.js" async defer></script>

<div id="cf-turnstile-register"
     class="cf-turnstile"
     data-sitekey="{{ turnstile_site_key }}"
     data-action="register"
     data-size="invisible"
     data-callback="onTurnstileSuccess"
     data-error-callback="onTurnstileError"
     data-expired-callback="onTurnstileExpired">
</div>
```

Context variable supplied by the view:

```python
'turnstile_site_key': settings.TURNSTILE_SITE_KEY
```

---

## 4) Client-side (minimal JS gate)

We **did not** restructure existing AJAX. We only added:

* A short **gate** at the start of form submit: if no token, call `turnstile.execute()` and retry submit on success.
* Two simple global callbacks for error/expiry.
* A `turnstile.reset()` in `finally()` because tokens are **single-use**.

Key pieces (already merged in `static/js/login_register.js`):

* `const WIDGET_ID = 'cf-turnstile-register'`
* `window.onTurnstileSuccess = (…)` (defined per submit attempt)
* `window.onTurnstileError = (…)` and `window.onTurnstileExpired = (…)`
* `turnstile.reset(WIDGET_ID)` in `finally()`.

This keeps login (which has no widget) unchanged.

---

## 5) Server-side validation (mandatory)

File: `dashboard/security.py`

* Calls **Siteverify**: `POST https://challenges.cloudflare.com/turnstile/v0/siteverify`
* Sends `secret`, `response` (token), optional `remoteip`, optional `idempotency_key`.
* Timeout: 8s; returns a structured dict; **fails closed** on any error.
* Supports **hardening**: `expected_action="register"` and `expected_hostname=…` (derived from `SITE_URL`).

Excerpt (already merged):

```python
def verify_turnstile(token, remoteip=None, *, expected_action=None, expected_hostname=None,
                     idempotency_key=None, timeout=8) -> dict:
    # returns {success, error_codes, hostname, action, challenge_ts, token_age_seconds, ...}
```

### Where we call it

In `register_view`, **before** `form.is_valid()`:

* Read token from `request.POST['cf-turnstile-response']`
* Compute IP via `get_client_ip(request)` (ALB-aware)
* Pass `expected_action="register"` and `expected_hostname=urlparse(settings.SITE_URL).hostname`
* On failure, log a `SystemLog` event and return a friendly 400 JSON message.

---

## 6) IP handling with ALB

Utility: `dashboard/utils.py::get_client_ip`

Current behavior:

* If `X-Forwarded-For` exists, return its **first** (leftmost) IP; else return `REMOTE_ADDR`.
* Works correctly **when traffic cannot reach the app except through the ALB** (prevents spoofed XFF in practice).

If we later expose the app directly or add more proxies, consider a stricter “**leftmost untrusted**” parser that strips trusted proxy CIDRs from the right of XFF and returns the next address.

---

## 7) Error handling & friendly messages

On failed validation we log:

```
SystemLog(event="CaptchaFailed",
          description="Turnstile failed ip=<ip> errors=<codes> host=<hostname> action=<action> age=<age>")
```

User-facing message mapping (subset):

* `missing-input-response`, `invalid-input-response`, `timeout-or-duplicate` → “Verification expired/missing. Please try again.”
* `action_mismatch`, `hostname_mismatch` → “Verification mismatch. Refresh and try again.”
* Network/HTTP errors (any exception) → “Network error. Please try again.”

---

## 8) Testing (manual & automated)

### Manual (dev/staging)

Use Cloudflare’s **dummy keys** in `.env`:

* Always-pass sitekey & secret (for quick sanity checks).
* Always-fail variants to test the error path.
  Then verify:

1. Submitting the Register form succeeds with valid token.
2. Hostname/action mismatch produces 400s.
3. Submitting twice without a new token fails (single-use).
4. Disconnecting egress returns the generic network error and logs `internal-error`.

### Automated (pytest)

* `tests/test_captcha.py` mocks `requests.post` inside `dashboard.security` to simulate Cloudflare responses (success, action/hostname mismatch, timeout-or-duplicate, and network failure).
* For legacy tests that do not include CAPTCHA (e.g. `test_auth_views.py`), we disable Turnstile with an **autouse fixture** that sets `dashboard.security.TURNSTILE_ENABLED=False`.
  This preserves prior expectations while keeping the CAPTCHA suite strict.

---

## 9) Operations & Maintenance

### Secrets

* Rotate `TURNSTILE_SECRET_KEY` on a standard cadence or when staff with access leave.
* Store in a secure secret manager; never log the secret or raw tokens.

### Monitoring

* **Sentry** will capture unexpected exceptions (we catch broadly but still log).
* Watch `security` logger and `SystemLog` entries for spikes in:

  * `CaptchaFailed`
  * `internal-error` (network or Cloudflare outage)
  * `timeout-or-duplicate` (automation/bot retries)

### Networking

* Ensure outbound 443 to `challenges.cloudflare.com` from the task/container subnets.
* If you add an egress proxy, allow that host.

### Performance

* Siteverify timeouts are set to **8s**; users get a friendly message on transient issues.
* Tokens are short-lived; the client always fetches a **fresh** token on submit.

### Change management

* **Enable on login** (optional):
  Add the same widget to `login.html` (change `data-action="login"`), call `verify_turnstile(..., expected_action="login")` in `login_view` before auth.
* **Disable quickly** (rollback): set `TURNSTILE_ENABLED=False` (env/config) – the view will bypass validation and proceed (intended only for dev/staging).
* **Hostname updates**: When changing domains, update `SITE_URL` (and rebuild) so `expected_hostname` stays aligned.

---

## 10) Quick Go-Live Checklist

* [ ] `TURNSTILE_SITE_KEY` and `TURNSTILE_SECRET_KEY` present in env for target environment.
* [ ] `SITE_URL` matches the public domain; `TURNSTILE_EXPECTED_HOSTNAME = urlparse(SITE_URL).hostname`.
* [ ] Outbound 443 to Cloudflare challenges endpoint allowed.
* [ ] Exactly **one** widget on the page (avoid duplicate Turnstile DIVs).
* [ ] JS callbacks present (`onTurnstileSuccess`, `onTurnstileError`, `onTurnstileExpired`) and `turnstile.reset()` happens after submit.
* [ ] Logs visible in Sentry / CloudWatch; alarms for spikes in `CaptchaFailed`.
* [ ] Pytest suite passing (captcha tests + legacy auth tests with disabled fixture).

---

## 11) Appendix: Adding to another form quickly (e.g., login)

1. **Template**: add:

   ```html
   <div id="cf-turnstile-login"
        class="cf-turnstile"
        data-sitekey="{{ turnstile_site_key }}"
        data-action="login"
        data-size="invisible"
        data-callback="onTurnstileSuccess"
        data-error-callback="onTurnstileError"
        data-expired-callback="onTurnstileExpired"></div>
   ```
2. **JS**: either reuse the same gate by generalizing `WIDGET_ID`, or add a second small gate that looks for `cf-turnstile-login` on that page.
3. **View**: call

   ```python
   result = verify_turnstile(token, ip,
                             expected_action="login",
                             expected_hostname=urlparse(settings.SITE_URL).hostname,
                             idempotency_key=str(uuid.uuid4()))
   ```

   Reject on failure with the same friendly mapping.

---

### TL;DR

* Turnstile is **on for registration**, invisible on the client, and **strictly validated** server-side with hostname/action checks.
* It’s easy to extend to other forms.
* We’ve got **minimal JS**, robust server verification, clear logging, and clean test coverage.
