# Webpages - User Settings  
**Boritrader LLC**   
**Last Updated 11/14/24**

This section provides an overview of the **User Settings Page**, where users can manage their profile, account preferences, API keys, trading settings, notifications, and algorithm configurations.
---

## **Overview of Features**  

- **Access Control**:  
  - Users must be **logged in** to access the settings page.
  - Django’s `@login_required` decorator ensures only authenticated users can access the page.

- **Rendering Settings**:  
  - On page load, the system retrieves the user's **profile, trading preferences, notification settings, API keys**, and **algorithm configurations** to display their current values.

- **Form Submission**:  
  - Form submissions are processed atomically to ensure data consistency using Django’s `@transaction.atomic`.
  - JavaScript manages form data collection and submission using the `fetch()` API for an interactive, asynchronous experience.

---

## 1. **Design and Layout Overview**  `settings.html`

### **Toggle Labels**  
- The HTML structure includes reversed labels for toggles:
  - This unique design helps the toggles visually indicate an active state towards their labels. For example, the `Notification Only` label is associated with the `auto_purchase_toggle`, creating a clear, user-friendly indication of the state.

### **HTML Templates**  
- The page uses (`settings.css`) and (`settings.js`), as well as shared styles `base.css` and `headers.html`. 

- **Sections Layout**:  
  - The page is divided into intuitive sections:
    - **Profile Information**
    - **Account Settings**
    - **Trading Preferences**
    - **Notification Preferences**
    - **API Keys**
    - **Algorithm Preferences**

---

## 2. **Sections on the Settings Page**  `settings.html`, `views_settings.py`

### **Profile Information**  
- Users can update their **full name**, **address**, **date of birth**, and **subscription plan**.  
- The subscription functionality is currently not yet supported.

### **Account Settings**  
- Users can update their **email address**.
  - If the new email is already registered, the system will prompt the user to use a different one.
- **Password Changes**: Users can update their password. Django's `validate_password()` function ensures new passwords meet security standards.  
  - The `update_session_auth_hash()` method ensures users remain logged in after password changes.

### **Trading Preferences**  
- Users can toggle between:  
  - **Auto Purchase vs. Notification Only**  
  - **Paper Trading vs. Live Trading**
- **Toggle Behavior**:
  - Each toggle has a corresponding label that may not visually match its function. This was done to maintain intuitive behavior where toggles activate towards their labels.
- **Toggle States**:
  - The interface uses hidden input fields to ensure current states are sent to the server during form submissions. Even if no change is detected, the form submits these hidden fields to prevent a descrete bug where data reverts to unknown default settings.
- **Logic**:
  - When `auto_purchase_enabled` is toggled, its state inverts the `notification_only_enabled` field.
  - When `paper_trading_enabled` is toggled, its state inverts the `live_trading_enabled` field.

### **Notification Preferences**  
- Users can manage notification settings by enabling or disabling:  
  - **Email Notifications**  
  - **SMS Notifications**

### **API Keys Management**  
- Users can manage their **API key**, **API secret**, and **region code (TLD)** for trading platform connectivity.  
  - If both key and secret fields are filled, the system updates or creates a new record.

### **Algorithm Preferences (Setting to be moved to Dashboard)**  
- Users can configure algorithms with options such as:
  - **Algorithm selection**
  - **Tickers** (e.g., BTC, ETH)
  - **Quantity to trade**
  - **Execution frequency**

---

## 3. **JavaScript and Form Handling** `settings.js`

### **JavaScript Overview**
- JavaScript appends hidden input elements to ensure current toggle states are sent during form submission.
- **Event Listeners** update hidden input values upon toggle changes.
- **Fetch API** processes form submissions asynchronously and displays feedback.

### **Detailed Form Submission Steps**
1. **Prevent Default Form Submission**:
   - `event.preventDefault()` is used to enable asynchronous handling.
2. **Submit via Fetch API**:
   - Form data is sent using `fetch()`:
     - URL: `form.action`
     - Method: `POST`
     - Body: `formData`, with a CSRF token for security.
3. **Response Handling**:
    - **Success**: Page reloads to show changes.
    - **Failure**: Error messages are parsed from the server's JSON response and displayed via the `messageBox`.

    - **Displaying Messages with messageBox**:
      - The messageBox element shows both success and error messages.
      - It applies CSS classes (alert-success or alert-error) for appropriate styling.
      - The messageBox displays the data.message or error.message, making feedback immediately visible to users.
        - **Common Error Sources and Examples**:
          - **Duplicate Email**:
            - If the server responds with { status: 'fail', message: 'Email already in use' }, it is displayed as "Error: Email already in use".
          - **Invalid Password**:
            - For an invalid password, a response like { status: 'fail', message: 'Password is too weak' } is shown as "Error: Password is too weak".
          - **JavaScript-Level Errors**:
            - Issues like network failures are captured by the catch block and displayed as "Error: An unknown error occurred" or with a specific error message if available.


### **Password Visibility and Input Toggles**
- JavaScript allows toggling password visibility.
- **Trading Preference Toggles** have reversed label behavior for intuitive state indications.

### **Multiple Submission Prevention**
- The **submit button** is temporarily disabled post-click to prevent duplicate submissions.

---

## 4. **Security Considerations**  

### **Authentication and Access Control**  
- The page is protected with Django’s `@login_required` decorator to prevent unauthorized access.
- If a non-authenticated user attempts to access the page, they are redirected to the login screen.

### **Password Security**  
- Passwords are validated using Django’s `validate_password()` and stored securely using PBKDF2 hashing with SHA256.

3. **CSRF Protection**  
   - All form submissions include **CSRF tokens** to protect against cross-site request forgery attacks.

4. **Atomic Transactions**  
   - Updates are wrapped within **atomic transactions** to ensure data consistency. If any part of the update fails, the entire transaction is rolled back.

### **Error Handling**  
- User-friendly messages are displayed when errors occur, such as duplicate email warnings or invalid password feedback. See `Shared Resources` documentation for more information on `base.js`.
