# Cloudflare Turnstile Setup Guide
**Boritrade, LLC**  
**Last Updated:** February 10, 2026   

---

## 🎯 What is Cloudflare Turnstile?
Add CAPTCHA protection to your Boritrader login and registration pages using Cloudflare Turnstile (free alternative to reCAPTCHA).

Turnstile is Cloudflare's CAPTCHA alternative that:
- ✅ **Free** for unlimited requests
- ✅ **Privacy-friendly** - No tracking
- ✅ **User-friendly** - Often invisible
- ✅ **No Google dependency**

**When to use:**
- High traffic sites prone to bot attacks
- Multiple failed login attempts
- Spam registration prevention

**When to skip:**
- Low traffic personal deployments
- Internal company use
- Development/testing environments

---

## 📋 Prerequisites

### 1. Get Cloudflare Turnstile Keys

1. **Sign up** for Cloudflare (free): https://dash.cloudflare.com/sign-up
2. **Navigate to Turnstile**:
   - Cloudflare Dashboard → Turnstile
   - Or direct: https://dash.cloudflare.com/?to=/:account/turnstile
3. **Add a Site**:
   - Site name: `MySite Production` (your choice)
   - Domain: `localhost` (for development) or `yourdomain.com` (for production)
   - Widget Mode: **Managed** (recommended - balances security and UX)
4. **Copy your keys**:
   - **Site Key** (public, starts with `0x4AA...`)
   - **Secret Key** (private, keep secure!)

### 2. Add Keys to .env File

Edit your `.env` file and add:

```env
# Cloudflare Turnstile CAPTCHA
TURNSTILE_SITE_KEY=0x4AA... # Your site key here
TURNSTILE_SECRET_KEY=0x4BB... # Your secret key here
```

**Important**: Never commit your `.env` file!

---

## 🔧 Enable Turnstile (Step by Step)

Turnstile is **already coded** in your application but **commented out**. Here's where to uncomment:

### Step 1: Enable in Django Settings
**File**: `djangostuff/config/settings.py`
**Find and uncomment** these sections: 

(around line 62-68)
```python
# Cloudflare Captcha Turnstile
#TURNSTILE_SITE_KEY   = config("TURNSTILE_SITE_KEY")
#TURNSTILE_SECRET_KEY = config("TURNSTILE_SECRET_KEY")
#TURNSTILE_ENABLED    = bool(TURNSTILE_SITE_KEY and TURNSTILE_SECRET_KEY)

# Optional: let dev/staging "fail-open"
#TURNSTILE_BYPASS_IN_DEBUG = DEBUG
```

---

### Step 2: Enable Security Module
**File**: `djangostuff/dashboard/security.py`
**Find and uncomment** these sections: **entire file**
**What this does**: Enables the `verify_turnstile()` function that validates CAPTCHA tokens with Cloudflare.

---

### Step 3: Enable in Views (Login & Register)
**File**: `djangostuff/dashboard/views_auth.py`
**Find and uncomment** these sections:

#### A) Imports and variables (around line 17, 112):
```python
# Uncomment:
#from dashboard.security import verify_turnstile
...
#EXPECTED_HOSTNAME = urlparse(settings.SITE_URL).hostname   # CAPTCHA TURNSTILE
#TURNSTILE_SITE_KEY = settings.TURNSTILE_SITE_KEY           # CAPTCHA TURNSTILE
```
#### B) Validation logic (around line 118-148):
```python
# Uncomment:
        # # --- CAPTCHA TURNSTILE ---
        # if not verify_turnstile_token(request):
        #     return JsonResponse({
        #         'status': 'error',
        #         'errors': {'__all__': ['CAPTCHA verification failed. Please try again.']}
        #     })
        # # --- END CHAPTCHA TURNSTILE ---
```

#### C) ADD Template context (around line 150):
```python
# Before:
        'turnstile_site_key': "",#TURNSTILE_SITE_KEY, # remove ["",] if enabling TURNSTILE_SITE_KEY

# After:
        'turnstile_site_key': TURNSTILE_SITE_KEY,
```

---

### Step 4: Enable in Register Template
**File**: `djangostuff/dashboard/templates/register.html`
**Find and uncomment** these sections:

#### A) Cloudflare script (around line 12):
```html
<!-- Uncomment: -->
<!-- <script src="https://challenges.cloudflare.com/turnstile/v0/api.js" async defer></script> -->
```

#### B) Widget div (around lines 80-88):
```html
<!-- Uncomment: -->
      <!-- CLOUDFLARE - This injects a hidden input named cf-turnstile-response
      <div id="cf-turnstile-register"
        class="cf-turnstile"
        data-sitekey="{{ turnstile_site_key }}"
        data-theme="auto"
        data-size="normal"
        data-callback="onTurnstileSuccess"
        data-error-callback="onTurnstileError"
        data-expired-callback="onTurnstileExpired">
      </div>
      -->
```

---

### Step 5: Enable in JavaScript
**File**: `djangostuff/dashboard/static/js/login_register.js`
**Find and uncomment** these sections:

(around lines 49-68):
```javascript
// Uncomment:
// Captcha Turnstile widget id (exists only on register page)
// const CAPTCHA_WIDGET_ID = 'cf-turnstile-register';

// // Put this near the top of the file
// let pendingForm = null;

// // Auto-resubmit once the user completes the visible captcha
// window.onTurnstileSuccess = function () {
//   if (pendingForm) {
//     const btn = pendingForm.querySelector('button[type="submit"]');
//     pendingForm.requestSubmit(btn || undefined);
//     pendingForm = null;
//   }
// };
// // Provide callbacks referenced in data-attributes
// window.onTurnstileError = function () {
//     // Show a friendly message and reset
//     try { showError('Verification error. Please try again.'); } catch (e) {}
//     if (window.turnstile && document.getElementById('cf-turnstile-register')) {
//       turnstile.reset('cf-turnstile-register');
//     }
//   };
// window.onTurnstileExpired = function () {
//     // Token aged out before submit; get a fresh one
//     if (window.turnstile && document.getElementById('cf-turnstile-register')) {
//         turnstile.execute('cf-turnstile-register');
//     }
// };
```

(around lines 59-69)
```javascript
//ALSO UNCOMMENT:
    // // --- Turnstile gate (visible) ---
    // const cfWidget = document.getElementById(CAPTCHA_WIDGET_ID);
    // if (cfWidget) {
    //   const tokenInput = form.querySelector('input[name="cf-turnstile-response"]');
    //   if (!tokenInput || !tokenInput.value) {
    //     alert('Please complete the CAPTCHA verification.');
    //     return; // stop now; on success, onTurnstileSuccess will resubmit
    //   }
    // }
    // // --- /Turnstile minimal gate ---
```

---

### Step 6: Optional - Enable Tests
**File**: `djangostuff/dashboard/tests/test_captcha.py`
**Find and uncomment** these sections: **Entire file**
**Why enable**: Ensures Turnstile integration doesn't break in future updates

---

## 🚀 Restart and Test

### 1. Restart Django Service

```bash
# Apply changes
docker-compose restart django

# Or full rebuild if needed
docker-compose up --build
```

### 2. Test Registration

1. Navigate to: http://localhost:8000/register/
2. You should see the **Turnstile widget** (checkbox or invisible verification)
3. Fill out the form and submit
4. If successful, CAPTCHA verification happens automatically

### 3. Verify in Logs

```bash
# Check Django logs
docker-compose logs django | grep -i turnstile

# Should see logs about Turnstile verification
```
---

## 🎨 Customization Options

### Widget Appearance

In `register.html`, you can customize the widget:

```html
<div id="cf-turnstile-register"
  class="cf-turnstile"
  data-sitekey="{{ turnstile_site_key }}"
  data-theme="light"        <!-- Options: light, dark, auto -->
  data-size="normal"        <!-- Options: normal, compact -->
  data-callback="onTurnstileSuccess"
  data-error-callback="onTurnstileError"
  data-expired-callback="onTurnstileExpired">
</div>
```

### Bypass in Development

In `settings.py`, if you set:
```python
TURNSTILE_BYPASS_IN_DEBUG = DEBUG
```

Then Turnstile will **auto-pass** when `DEBUG=True`, making development easier.

**Production**: Set `DEBUG=False` to enforce real verification.

---

## 🐛 Troubleshooting

### Widget Not Showing

**Problem**: No CAPTCHA widget appears on register page

**Solutions**:

1. **Check site key** in template:
```bash
docker-compose logs django | grep turnstile_site_key
# Should show your actual key, not empty string
```

2. **Check browser console** (F12):
```javascript
// Should NOT see errors about Turnstile
```

3. **Verify script loaded**:
```html
<!-- Should be in <head> of register.html -->
<script src="https://challenges.cloudflare.com/turnstile/v0/api.js" async defer></script>
```

### Verification Always Fails

**Problem**: "CAPTCHA verification failed" error every time

**Solutions**:

1. **Check secret key** in `.env`:
```bash
cat .env | grep TURNSTILE_SECRET_KEY
# Should show your secret key
```

2. **Check domain in Cloudflare**:
   - Dashboard → Turnstile → Your Site → Settings
   - Make sure `localhost` is in "Domains" for development

3. **Check logs** for specific error:
```bash
docker-compose logs django | grep -i "turnstile\|captcha"
```

Common errors:
- `hostname_mismatch`: Domain not configured in Cloudflare
- `action_mismatch`: Expected action doesn't match
- `timeout-or-duplicate`: Token expired or reused

### Development Bypass Not Working

**Problem**: Still requires CAPTCHA even with `DEBUG=True`

**Solutions**:

1. **Verify DEBUG setting**:
```bash
docker-compose exec django python -c "from django.conf import settings; print(settings.DEBUG)"
# Should print: True
```

2. **Check TURNSTILE_BYPASS_IN_DEBUG**:
```bash
docker-compose exec django python -c "from django.conf import settings; print(settings.TURNSTILE_BYPASS_IN_DEBUG)"
# Should print: True
```

3. **Restart Django** after `.env` changes:
```bash
docker-compose restart django
```

### ImportError: security module

**Problem**: `ImportError: cannot import name 'verify_turnstile'`

**Solution**: Make sure you **uncommented** the entire `security.py` file.

```bash
# Check if security.py is uncommented
head -5 djangostuff/dashboard/security.py
# Should NOT start with "# # dashboard/security.py"
```

---

## 🔒 Security Best Practices

### Production Checklist

- [ ] Use separate site for production domain (not `localhost`)
- [ ] Set `TURNSTILE_BYPASS_IN_DEBUG = False` in production
- [ ] Keep `TURNSTILE_SECRET_KEY` secure (never commit!)
- [ ] Monitor Cloudflare dashboard for unusual activity
- [ ] Set up rate limiting alongside Turnstile
- [ ] Configure proper `EXPECTED_HOSTNAME` for your domain


## 📚 Reference

### Files Modified

| File | Purpose |
|------|---------|
| `.env` | Store CAPTCHA keys |
| `config/settings.py` | Load keys, enable feature |
| `dashboard/security.py` | Verification logic |
| `dashboard/views_auth.py` | Call verification |
| `templates/register.html` | Widget display |
| `static/js/login_register.js` | Widget callbacks |
| `tests/test_captcha.py` | Tests (optional) |

### Environment Variables

```env
TURNSTILE_SITE_KEY=0x4AA...      # Public key (visible in HTML)
TURNSTILE_SECRET_KEY=0x4BB...     # Private key (server-side only)
```

### Settings

```python
TURNSTILE_ENABLED = bool(TURNSTILE_SITE_KEY and TURNSTILE_SECRET_KEY)
TURNSTILE_BYPASS_IN_DEBUG = DEBUG  # Auto-pass in development
```

---

## 🔗 Additional Resources

- **Cloudflare Turnstile Docs**: https://developers.cloudflare.com/turnstile/
- **Dashboard**: https://dash.cloudflare.com/?to=/:account/turnstile
- **Migration from reCAPTCHA**: https://developers.cloudflare.com/turnstile/migration/

---

## ❓ FAQ

**Q: Is Turnstile really free?**  
A: Yes, unlimited requests for any site.

**Q: Can I use it on localhost?**  
A: Yes, add `localhost` to "Domains" in Cloudflare settings.

**Q: Does it work without JavaScript?**  
A: No, Turnstile requires JavaScript. Consider fallback for accessibility.

**Q: Can I use multiple domains?**  
A: Yes, create separate sites in Cloudflare for each domain.

**Q: What if Cloudflare is down?**  
A: Requests will fail. Consider implementing a fallback or timeout.

---

**Need help?** Open an issue on GitHub or check the Cloudflare Turnstile documentation.
