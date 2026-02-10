# The Boritrader Documentation: Patch Notes - Dec 21, 2024  
**Boritrade, LLC**

## Overview  

1. Deployed the application to AWS with production-grade infrastructure.
1. Initiated small-group user testing.
1. Fixed critical bugs affecting API secrets and form submissions.
1. Implemented deeper logging for enhanced traceability and debugging.
1. Updated user session management to ensure logout before account creation.
1. Enhanced settings and forms functionality for improved user experience.

## Key Updates  

## Deployment to AWS  
- Successfully deployed the application to AWS infrastructure.  
  - Configured connection to AWS RDS for database management.  
  - Created repository for production version of application. 
  - Added relevant secrets to AWS parameter store and System Manager
  - Added custom scripts on EC2 instance to automatically authenticate & pull secrets
  - Implemented SSL/HTTPS for secure communications using AWS ALB  
  - Updated server security using AWS ALB WAF&Shield security rules
  - Created repository for infrastructure-as-code via terraform


### Forms and Settings  
- **Centralized Forms Management:** Updated login, register, and settings pages to utilize django's native `forms.py` to streamline form functionalities. This allows for greater control over fields including validation, requirements, and custom error messages. 
- **API Key Integration:** Added API key handling to the registration page, preventing errors associated with users connecting to dashboard without valid API keys. 


### Bug Fixes  
- **No API Key Redirect:** Fixed a bug causing crashes when no API key was present. No API Key now prompts redirects to settings page. System still found to throw 500 error if keys present, but invalid. 
- **Enhanced Logging:** Introduced deeper logging mechanisms to capture detailed activity and errors, streamlining issue resolution. All logs propagate to debug.log & errors.log.
- **User Session Management:** Ensured users are logged out automatically on registration page to avoid conflicts when creating new new accounts.


## In Progress  
1. Expanding small-group user testing to collect valuable feedback.  
1. Translating current AWS infrastructure to terrform IaC

## Goals for the Month  
1. Finalize AWS deployment and optimize performance, security, and scalability.
1. Finalize Django security & Logging