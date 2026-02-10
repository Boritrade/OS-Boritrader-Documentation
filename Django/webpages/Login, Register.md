# Webpages - User Login & Registration  
**Boritrader LLC**

This section outlines the implementation of the **login**, **logout**, and **registration** workflows in Boritrader. It explains the key features, restrictions, and security practices to ensure a smooth and secure user experience.

Note: Login & Register do **NOT** import header.html using `{% include 'includes/header.html' %}`, and therefore do not share the same header with the rest of the html pages. 
---

## 1. **User Login Process**

### **How User Login Works**  
- Users access the **login page** and submit their credentials (username and password).  
- The system uses Django’s `authenticate()` function to verify the credentials.  
  - If the credentials are valid and the account is active, Django’s `login()` function creates a session for the user.  
  - If the login fails (due to incorrect credentials or inactive accounts), the system returns an error message.  
- On successful login, the user is redirected to the **dashboard** or the appropriate page.

### **Features**  
- **Session Management**: The system maintains sessions for authenticated users.  
- **Error Handling**: Provides feedback through JSON responses for success or failure.  
- **Frontend Validation**: Ensures all required fields are filled before form submission.

### **Restrictions**  
- **Active Accounts Only**: Users must have an active account to log in.  
- **Valid Credentials Required**: Username and password must match the system records.

---

## 2. **User Logout Process**

### **How User Logout Works**  
- When a user logs out, Django’s `logout()` function is called to terminate the session.  
- The system invalidates the session and redirects the user to the **login page**.  
- This ensures no residual access to user data from the previous session.

### **Features**  
- **Session Termination**: Immediately ends the session upon logout.  
- **Redirect to Login**: Users are redirected to the login page after logging out.

### **Restrictions**  
- **Logged-in Users Only**: Users must be logged in to access the logout functionality.

---

## 3. **User Registration Process**

### **How User Registration Works**  
1. **Form Submission**:  
   - Users provide a **username**, **email**, and **password** through the registration form.  
2. **Username and Email Validation**:  
   - The system checks if the provided username or email is already registered.
   - If a duplicate is found, the registration fails, prompting the user to try another username or email.  
3. **Password Validation**:  
   - Django’s `validate_password()` function is used to ensure the password meets security standards (e.g., length, character variety).  
4. **User Creation and Login**:  
   - If all inputs are valid, the system creates the new user and securely stores the password using Django’s default hashing algorithm (PBKDF2 with SHA256).  
   - The user is logged in immediately upon successful registration and redirected to the **dashboard**.

### **Features**  
- **Password Validation**: Enforces strong passwords following Django’s guidelines.  
- **Atomic Transactions**: Uses `@transaction.atomic` to ensure data integrity during registration.  
- **Immediate Login**: Users are logged in automatically after registration.  
- **Error Handling**: Provides feedback for invalid inputs, such as duplicate usernames or weak passwords.

### **Restrictions**  
- **Unique Username and Email**: Users must register with a unique username and email.  
- **Valid Passwords Required**: Passwords must meet complexity requirements.

---

## 4. **Login and Registration Page Design**

### **HTML Login Template (`login.html`)**  
- **Layout**: The login page includes input fields for the **username** and **password**, with a login button.  
- **Error Handling**: Displays error messages for incorrect credentials.  
- **Password Visibility**: Users can toggle password visibility for convenience.  
- **Shared CSS & JS**: The same styling and JavaScript are used for both the login and registration pages.

#### **Security Considerations**  
- **CSRF Protection**: All form submissions include CSRF tokens to prevent cross-site request forgery.  
- **Limited Error Feedback**: Error messages are generic to avoid exposing sensitive information (e.g., "Invalid credentials" instead of "User not found").

---

### **HTML Registration Template (`register.html`)**  
- **Layout**: The registration page collects **username**, **email**, and **password** information, with a confirmation password field.  
- **Redirection Link**: Users can easily switch to the login page if they already have an account.  
- **Password Visibility**: Users can toggle the password field to prevent typing errors.  

#### **Security Considerations**  
- **CSRF Protection**: Form submissions are secured with CSRF tokens.  
- **Password Matching**: Both backend and frontend checks ensure the password and confirmation match.  
- **Minimal Information Exposure**: Errors do not indicate if a username or email is already registered.

---

## 5. **JavaScript for Password Visibility and Form Validation**

- **Password Visibility Toggle**:  
  - Both the login and registration pages use JavaScript to toggle password fields between hidden and visible.  
  - This feature helps users avoid typing errors during input.

- **Form Validation**:  
  - JavaScript ensures that all required fields are filled before form submission.
  - If a field is left empty, the form displays an error message and prevents submission.

- **Submission Prevention**:  
  - To prevent multiple submissions, the submit button is temporarily disabled once the form is submitted.

---

## **Total Security Related**

1. **Session Management**:  
   - Sessions are terminated immediately upon logout to prevent unauthorized access.  
   - Session cookies are secure and set with proper expiration policies.

2. **Password Security**:  
   - Passwords are hashed using Django’s PBKDF2 algorithm with SHA256 for maximum security.  
   - The `validate_password()` function ensures password strength by enforcing length, character variety, and more.

3. **Input Validation and Error Handling**:  
   - User input is validated both on the frontend and backend to prevent SQL injection and other malicious activities.  
   - Error messages are designed to provide minimal information to avoid revealing whether an account exists.

4. **CSRF Protection**:  
   - All form submissions are protected with CSRF tokens to prevent cross-site request forgery.

---