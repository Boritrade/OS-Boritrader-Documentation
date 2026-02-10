# **The Boritrader Documentation: Patch Notes - Feb 5 - March 5, 2024**  
**Boritrade, LLC**  

## **Overview**  

1. **UI/UX Overhaul** – Redesigned dashboard, portfolio, and settings pages for a more streamlined user experience.  
2. **Infrastructure Stability** – Resolved **ECS, Fargate, and NGINX** issues, fixing static resource access and ALB health checks.  
3. **Security Enhancements** – Strengthened **AWS WAF Shield rules**, introduced CAPTCHA for admin login, and refined access control policies.  
4. **Sentry.io Expansion** – Enhanced logging, deeper error analytics, and improved frontend monitoring.  
5. **Deployment & Rollback Protections** – Implemented Terraform safeguards to prevent failed deployments from disrupting production.  
6. **Cloud & Networking Optimizations** – Improved **TLS handshake stability**, updated **Redis configurations**, and prepared for **Cloudflare DNS integration**.  

## **Key Updates**  

### **Frontend & UI Redesign**  

- **Major Dashboard & Settings Redesign:**  
  - **Portfolio Section:** Introduced a **detailed portfolio overview** with better asset tracking.  
  - **Popup & Dropdown Updates:** Improved UI for algorithm selection and configuration.  
  - **Color & Border Adjustments:** Enhanced overall readability and aesthetics.  

- **Refactored Global Styling:**  
  - Removed **TailwindCSS** from dashboard for better control over styles.  
  - Consolidated basic styles into **base.css** for easier maintenance.  
  - Standardized UI components across the entire app.  

- **Authentication Page Overhaul:**  
  - **Login & Registration Pages:** Redesigned for better user experience and performance.  
  - Improved form validation and error handling.  

---

### **AWS & Infrastructure Fixes**  

#### **ECS Deployment & NGINX Fixes**  
- **Resolved Static File Issues:**  
  - Fixed **404 errors** affecting JavaScript & CSS assets on production servers.  
  - Updated **entrypoints & health checks** for proper resource handling.  

- **Fargate & Load Balancer Improvements:**  
  - Fixed **ALB health check failures** caused by incorrect HTTPS settings.  
  - Updated **healthcheck URLs** (`/healthcheck/`) to prevent unnecessary redirects (301 errors).  
  - Addressed **TLS handshake failures** affecting internal HTTPS communications.  

- **Rollback & Stability Improvements:**  
  - Added **Terraform rollback protections** to ensure failed deployments don’t break production.  
  - Improved lifecycle rules for AWS WAF, reducing unnecessary resource changes.  

#### **Security & WAF Enhancements**  
- **Stronger WAF Rules:**  
  - Moved WAF configuration to a dedicated **Terraform module**, improving maintainability.  
  - Implemented **stricter rule sets** to block automated attacks.  
  - **Removed outdated rules** and optimized priority order.  

- **Added CAPTCHA for Admin Login:**  
  - **Replaced IP-based admin blocking** with a CAPTCHA system to improve security.  

- **Dedicated WAF Shield Rules:**  
  - Enforced manual configuration for TLS & security settings.  
  - Improved DDoS protection by refining access control policies.  

---

### **Additional Fixes & Optimizations**  

- **Sentry Enhancements:**  
  - Improved backend logging to capture more granular tracebacks.  
  - Fixed issues with missing static resources breaking frontend logging.  
  - Resolved JavaScript errors caused by incomplete log captures in Sentry.  

- **Redis Configuration for Fargate:**  
  - Updated **Redis endpoint settings** to work with AWS Fargate deployment.  

- **Updated ALB Health Checks:**  
  - Fixed incorrect health check target paths, resolving issues with deployment readiness.  

- **Improved CI/CD Pipeline:**  
  - Updated **GitHub Actions pipeline** to account for health checks and entrypoints.  

---

## Goals for the Month  
1. **Marketing Website Launch** for user acquisition & promotion.  
1. **Subscription Features** for premium account management.  
1. **Cloudflare DNS Configuration** for improved routing & security.  

---