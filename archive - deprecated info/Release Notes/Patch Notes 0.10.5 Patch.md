# **The Boritrader Documentation: Patch Notes — September 24, 2025**
**Boritrade, LLC**   
**Related Release Version:** 0.10.5

---

## **Key Update**

### 1) Sept 24 User Registration Redirect Patch
- added Turnstile bypass for local testing 
- added user config initialization to registration page
  - fixed edge case: when user skips settings redirect and goes directly to dashboard, LoadConfigError pops up due to missing user settings.  

