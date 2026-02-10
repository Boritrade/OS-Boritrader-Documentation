# Boritrader Registration and Onboarding Workflow Documentation

**Boritrade, LLC**  
**Last Updated:** April 2, 2025

This document outlines the entire user onboarding journey for Boritrader—from the initial registration through to dashboard access. It includes detailed Descriptions of each step along with a summary for quick reference.

---

## Part I. Registration Page Features & Documentation

### 1. Registration Form

**Description:**  
The registration form is designed to guide new users through creating an account on Boritrader. Users start by entering basic account information including a unique username, a valid email address, and their password (entered twice for confirmation). A convenient “Show/Hide” option is provided for the password fields. The form then integrates with Binance’s API, where users must supply their API key and API secret—ensuring that they have the necessary permissions (e.g., “Enable Reading” with unrestricted IP restrictions). Users can also specify the API TLD (region) and indicate if they are using a testnet key, with direct guidance provided through helper links. Before submission, users are required to accept Boritrader’s Terms and Conditions and Privacy Policy. Finally, users can choose to either “Pay Now” to start their subscription immediately via Stripe or begin a “30-Day Free Trial” for full dashboard access without immediate payment.

**Summary:**
- **Basic Account Info:** Unique username, valid email, and password confirmation with visibility toggle.
- **Binance API Integration:** 
  - API key and secret with a helper link.
  - API TLD (region) selection.
  - Testnet option with a direct Binance Testnet link.
- **Legal Compliance:** Acceptance of Terms & Conditions and Privacy Policy via checkbox.
- **Onboarding Options:** Choose between “Pay Now” (immediate Stripe checkout) or “Start 30-Day Free Trial.”

---

### 2. Additional Resources and Guidance

**Description:**  
To assist users in successfully completing registration, the page provides direct access to essential resources. Users can consult a comprehensive guide on Binance’s support pages to learn how to create API keys with the proper permissions and security practices. A link to the Binance Testnet is also available for those who wish to experiment with their API setup in a risk-free environment. Additionally, users are given direct access to the legal documents, ensuring they understand their rights and obligations under Boritrader’s Terms and Conditions and Privacy Policy.

**Summary:**
- **Binance API Guide:** Detailed instructions on generating API keys with proper permissions.
- **Testnet Access:** Link provided for users to test API setup in a simulated environment.
- **Legal Documents:** Direct links to review the Terms & Conditions and Privacy Policy.

---

### 3. User Experience Enhancements

**Description:**  
The design of the registration form emphasizes user friendliness. A clean, organized layout breaks down the process into clear sections, while inline validation and detailed helper texts reduce errors and support users in entering accurate information. This thoughtful design is intended to provide a seamless registration experience that minimizes user frustration and improves overall satisfaction.

**Summary:**
- **Clean Layout:** Organized into clear, navigable sections.
- **Inline Validation:** Real-time error-checking for immediate feedback.
- **Helper Text:** Detailed guidance, especially in the API key section.

---

## Part II. Registration to Dashboard Workflow

### 1. Registration Process

**Description:**  
After completing the registration form with the required details (username, email, and password), new users establish their account on Boritrader. While essential information is mandatory, additional personal details (such as full name, address, and date of birth) remain optional. This initial step confirms the user’s identity and sets the stage for accessing Boritrader’s trading tools.

**Summary:**
- **Account Creation:** Mandatory fields for username, email, and password.
- **Optional Details:** Additional personal information may be provided.
- **Identity Confirmation:** Establishes the user’s identity for accessing trading tools.

---

### 2. Workflow Based on User Action

#### 2.1 Choosing the 30-Day Free Trial (Default Action)

**Description:**  
If the user does not explicitly choose an onboarding action, the system automatically starts a 30-day free trial. This “ghost trial” grants full access to the dashboard immediately after registration. During the trial period, users enjoy complete access to all features, but once the trial expires without activating a subscription, dashboard access is restricted.

**Summary:**
- **Default Action:** Automatic initiation of a 30-day free trial.
- **Immediate Access:** Full dashboard access during the trial.
- **Trial Expiry:** Access is restricted if no subscription is activated post-trial.

#### 2.2 Choosing “Pay Now” (Immediate Checkout)

**Description:**  
Alternatively, if users select the “Pay Now” option during registration, they are taken directly to the Stripe checkout process. Here, a Stripe customer profile is created (if it does not already exist), and upon successful payment, their subscription is activated. In the event of a canceled payment, the user is informed that a valid subscription is required and redirected back to the checkout page.

**Summary:**
- **Immediate Payment:** “Pay Now” redirects to Stripe checkout.
- **Subscription Activation:** Subscription is activated upon successful payment.
- **Payment Cancellation:** Notification provided with redirection back to checkout if payment fails.

---

### 3. Post-Registration Workflow

**Description:**  
After the registration process, access to the dashboard is managed through a subscription check mechanism. This system verifies whether the user is within the valid 30-day trial period or has an active paid subscription (synchronized with Stripe). Users without a valid subscription are redirected to the subscription page. Additionally, the Settings page allows users to manage their subscription—updating plans, canceling subscriptions, or synchronizing their status via Stripe’s customer portal.

**Summary:**
- **Subscription Check:** Verifies trial validity or active paid subscription.
- **Dashboard Access:** Granted only when a valid subscription or trial is confirmed.
- **Subscription Management:** Options to update, cancel, or synchronize subscription details through the Settings page.

---

### 4. Security and Safeguards

**Description:**  
Security is a top priority throughout the entire onboarding process. Sensitive data such as passwords and API credentials are protected by advanced encryption and hashing techniques. The system employs rate limiting to prevent abuse during registration and login, and all critical user actions are logged and monitored. Regular synchronization with Stripe helps ensure that subscription statuses are current and accurate, reinforcing the platform’s secure environment.

**Summary:**
- **Data Protection:** Advanced encryption and hashing for passwords and API credentials.
- **Abuse Prevention:** Rate limiting on registration and login attempts.
- **Monitoring:** Logging and monitoring of critical user actions.
- **Subscription Accuracy:** Regular synchronization with Stripe for up-to-date subscription status.
