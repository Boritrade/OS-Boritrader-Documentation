# The Boritrader Documentation: Patch Notes - Jan 7 - Feb 5, 2024  
**Boritrade, LLC**
**Related Release Version:** 0.1.31=0.2.4

## Overview  

1. Implemented comprehensive unit tests and CI/CD pipeline using GitHub Actions.
2. Completed a full security audit, introducing critical security enhancements.
3. Enhanced the logging system with database integration and external file storage.
4. Finalized AWS ECS infrastructure deployment with Terraform, ensuring secure, scalable architecture.
5. Resolved critical infrastructure issues, including NGINX errors and WebSocket compatibility with HTTPS.

## Key Updates  

### Unit Testing & CI/CD Pipeline  
- **Comprehensive Unit Testing:**  
  Developed unit tests for Django views, JavaScript components, and backend Python functions to ensure application reliability and upstream dependency compatibility.
  
- **CI/CD Pipeline via GitHub Actions:**  
  Configured a CI/CD pipeline for development environment to automate the running of unit tests on every push, providing immediate feedback and preventing automatic merges to `main` on failed tests. Also provides & ensures global compatibility without global environment variable sharing.

- **Documentation:**  
  Added detailed documentation on running tests locally and understanding CI/CD workflows.

---

### Security Enhancements  

- **Rate Limiting:**  
  Implemented request rate limiting using `django-ratelimit`. Login attempts are capped at 5 per minute per IP to prevent abuse and mitigate DoS attacks.

- **Brute Force Protection:**  
  Introduced `BruteForceProtectionMiddleware` to track failed login attempts. After multiple failed attempts, IPs are temporarily blocked, and unusual login patterns are logged.

- **Admin Interface Hardening:**  
  - Changed default `/admin/` URL to `/sys-admin/` to reduce exposure.
  - Applied IP-based access restrictions to limit admin access to trusted networks.

- **Robots.txt Optimization:**  
  Prevented search engines from indexing sensitive URLs (`/sys-admin/`, `/login/`, `/register/`, `/settings/`).

- **Constants Refactoring:**  
  Moved hardcoded values to `constants.py` for improved maintainability and cleaner code structure.

---
### Logging System Enhancements  

- **Database Logging:**  
  Critical logs are now pushed to the database, improving monitoring and facilitating easier debugging.

- **External File Storage:**  
  Less-critical logs are stored in external files, accessible outside the Docker container, ensuring comprehensive log management across environments.

- **Log File Rotation:**  
  Implemented automatic log file rotation using `TimedRotatingFileHandler`, ensuring that log files are rotated at midnight daily. This prevents log bloat and maintains clean, manageable log files, with up to three backups retained for review.

- **Sentry.io Integration:**  
  **What is Sentry?**  
  Sentry is a robust error tracking and performance monitoring tool that helps identify, diagnose, and resolve issues in real-time. It captures errors, performance bottlenecks, and user interactions, providing detailed insights for rapid debugging and optimization.

  **How It Works:**  
  Sentry captures both backend and frontend errors:
  - **Backend Integration:** Configured through `sentry_sdk.init()` in Django settings, capturing server-side exceptions, performance traces, and contextual information.
  - **Frontend Integration:** JavaScript SDK added to key HTML templates to capture client-side errors and user interactions via Sentry Replays. This provides visual session recordings to trace steps leading to issues.

  **Accessing Sentry:**  
  All captured errors and performance data are accessible at [https://boritrade.sentry.io/issues/](https://boritrade.sentry.io/issues/). The dashboard offers real-time alerts, detailed stack traces, and performance metrics.

  **Pages Integrated with Sentry:**  
  - **Global Coverage:** Added to `header.html`, ensuring Sentry is active on all pages that inherit this header.
  - **Authentication Pages:** Explicitly integrated into `login.html` and `register.html` for detailed tracking of user authentication flows.
  
  **Key Features Enabled:**  
  - **Error Tracking:** Real-time capture of uncaught exceptions and handled errors.
  - **Performance Monitoring:** Enabled tracing for monitoring API calls, page load times, and more.
  - **Session Replays:** Activated session recording to visually trace user actions leading up to errors.
  - **Data Privacy:** Configured to exclude personally identifiable information (PII) from being sent, ensuring compliance with privacy standards.
---

### AWS ECS Infrastructure Deployment  

- **Terraform-based Deployment:**  
  Redeployed the entire AWS infrastructure centered around ECS compatibility using Terraform, following a modular structure for scalability and maintainability.

- **Elastic Container Service (ECS):**  
  - Utilized AWS Fargate for serverless container orchestration.
  - Configured ECS clusters, tasks, and services for efficient container management.
  - The ECS **Service** manages running tasks and integrates with an **Application Load Balancer (ALB)** to route traffic securely over HTTPS.
  - ECS **Task Definitions** define container settings, including image references, CPU/memory allocation, and environment variables.

  - **Local vs. Production:** Local development relies on `docker-compose.yml` to run Redis, Django, and NGINX in isolated containers. In contrast, Fargate requires an ECS **Task Definition** in JSON format to specify how containers should run. A **CI/CD pipeline** was created to build Docker images and dynamically update these task definitions for deployment.

  - **CI/CD Automation:** The GitHub Actions pipeline automates the entire deployment process by:
    1. Building and tagging Docker images.
    2. Pushing images to **Amazon ECR**.
    3. Updating ECS **Task Definitions** with the new image versions.
    4. Injecting environment variables securely from **AWS Secrets Manager**.

- **Application Load Balancer (ALB):**  
  Implemented ALB for secure traffic routing with SSL/TLS encryption and health checks for ECS tasks.

- **Security Configurations:**  
  - Applied stringent security group rules to limit access to ECS, ALB, and RDS.
  - Used AWS Secrets Manager for secure credential storage.
  - Configured IAM roles to follow the principle of least privilege.

- **Relational Database Service (RDS):**  
  Set up a managed PostgreSQL 16 database with AWS RDS, ensuring high availability and security.

- **Bastion Host for Secure RDS Access:**  
  Introduced a **bastion host** deployed via Terraform in a private subnet to securely manage database migrations and maintenance tasks:
  - **Private Subnet & SSM Access:** The bastion operates in a private subnet without a public IP. Access is granted via AWS Systems Manager (SSM) Session Manager, eliminating the need for direct SSH connections.
  - **Database Migrations via Bastion:** All database migrations are now performed exclusively through the bastion host to ensure consistency, minimize downtime, and prevent conflicts in the production environment. This approach improves control over schema changes and enforces best practices for database management.
---

## In Progress  
1. Expanding Sentry.io integration for deeper error analytics and alerts.  
1. Fixing bugs related to ECS deployment & static resource access with nginx & Sentry
1. Enhancing WAF Shield Rules for ongoing application security

## Goals for the Month  
1. Finalize AWS infrastructure optimizations for cost-efficiency and scalability.  
1. Marketing Website for promotion & registration.
1. User subscription account features.
1. Dedicated WAF & Shield Rules.
1. Cloudflare to handle DNS