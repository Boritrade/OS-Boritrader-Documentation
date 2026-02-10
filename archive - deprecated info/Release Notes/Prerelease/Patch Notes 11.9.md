# The Boritrader Documentation: Patch Notes - Nov 9
**Boritrade, LLC**


## User Registration, Login, and Settings

### User Registration and Login
- Users can now create their own accounts, providing access to the dashboard and settings pages. This is managed using Django's native authentication library, supporting password encryption, field validation, redirects, and more.
- Currently, errors are returned as JSON responses; however, this will be updated to show popups in the near future.
- A universal menu is now present at the top of all pages, giving users access to Logout, Home, and Settings.

### Settings Page
- All settings on the settings page are stored in the database. When a user clicks **Save**, all database fields are updated with the new or existing information.
- **Note**: While the settings page is connected to the database, the BinanceTrader does not yet use this data for automated trading. This functionality is expected to be completed in the coming weeks.
- API keys provided in the settings are currently not in use. Test keys from Binance are used instead, granting users access to a paper trading account. As a result, assets and available balances are displayed on the dashboard.
- **Important**: This app is not live, so even when trading in "live mode," the app will not submit actual orders.

### Algorithm Preferences
- Users will soon be able to customize algorithm-specific settings. This section is still in development and may be moved to the dashboard for better accessibility.

### Database Management
- All database tables are managed through Django and accessible via the Django admin panel.
- When a user is deleted, all related entries in other tables that reference their `user.id` are also removed, except for admin-specific tables such as system logs, login attempts, and password resets.

## Login Redirects
- Users must be logged in to access the dashboard or settings. If a user tries to access these pages without logging in, they will be redirected to the login page. Once authenticated, they will be redirected back to the page they were attempting to access. This redirection is handled using Django's native libraries.

## GitHub Encapsulation

### Repository Management
- The `crypto` directory containing the source code has been split into three separate GitHub repositories, each with its own unit tests, CI/CD pipelines, and `setup.py` files to support `pip install` (provided authentication succeeds and the developer has access to the repo).
- The GitHub Actions pipeline for these repositories is as follows:
  1. When a user pushes to the `develop` branch:
     - Dependencies are compiled and installed.
     - Linting tests are run to ensure best practices and catch errors.
     - Unit tests are run for overall app compatibility.
     - A draft PR request is created for merging with the `main` branch.
  2. Currently, the pipeline bypasses the draft stage and merges with `main` automatically if all tests pass. This setup exists because the repositories are private and not part of an organization account yet. They will be moved to the Boritrader Organization GitHub account in the upcoming week. The delay is due to work by an external contractor, which posed a risk of repository data being wiped or deleted. With this risk now mitigated, full integration into the Boritrader main repository is pending.

### Main Repository Updates
- The main repository is being restructured to pull packages from these separate repositories via `pip install`, rather than hosting all files in one directory. This involves:
  - Deleting the `crypto` folder.
  - Refactoring import statements.
  - Passing keys to Docker using GitHub secrets and environment variables.

## Summary of Immediate Next Steps

1. Complete the reworking of the main repository to support pulling packages from the split repositories using `pip install`. This includes:
   - Deleting the `crypto` folder.
   - Refactoring import statements.
   - Configuring Docker to use GitHub secrets and environment variables.
2. Finalize the functionality for BinanceTrader to utilize settings stored in the database for automated trading.

### Goals for the Month
Refer to the GitHub Project Issues for more details:
1. Implement Paper Trading (an internal system for long-term algorithm efficiency tracking).
2. Add support for Binance Testnet Keys, allowing users to switch between their own API keys and testnet keys.
3. Deploy the application to AWS.
4. Enhance the GUI with a professional styling overhaul.
5. Optimize algorithms for improved speed and utility.
