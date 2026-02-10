# The Boritrader Documentation: Patch Notes - Dec 3
**Boritrade, LLC**


## Overview
1. Restructured internal dependencies (removed local crypto repository, now installed via `pip install`)
1. Finalized the functionality for BinanceTrader to utilize settings stored in the database for automated trading.
1. Implemented paper trading by adding support for Binance Testnet Keys, allowing users to switch between their own API keys and testnet keys.
1. Added Algorithm Customization to Homepage, including new Popup window
1. Added Error/Success Message Popups to home

## Dependency Restructuring
Previously  The `crypto` directory containing the source code was split into three separate GitHub repositories. The Django portion of the app has now been successfully configured to import these repositories directly from github using a Personal Access Token passed as an environmental variable passed to docker via the env.list file.
This also required modifications to the workflow to account for an additional requirements.txt file, `requirements_git.txt`, which installs all github dependencies using envvar passed to the container at runtime, rather than during initialization.

## Database Settings Connected to BinanceTrader
Dependency was updated to use abstract class for configuration data, allowing database compatibility. Settings stored by users in Settings Page are now applied immediately across the app.

## Binance Testnet Support
Users can now choose whether to use live API keys or testnet API keys in the settings page. Testnet keys will give users a Paper Trading Account prepopulated with various assets as well as ample cash for additional trading.
Note: Actual automated trading is still disabled system-wide for safety while timings are improved.

## Algorithm Customization on Homepage
Added a popup menu on the homepage to configure settings on a per-algorithm basis for a more user-friendly experience. Removed Algorithm Preferences settings from settings page.

## Added Algorithm Descriptions to Homepage
Added a popup window to the homepage displaying descriptions of the algorithms, including purpose, use cases, and potential scenarios.
Descriptions can be saved to database using html formatting and will display exactly as formatted in db. Admins are only ones allowed to modify algorithm descriptions.

## Added Algorithm Status to Homepage
Added Algorithm Status section t to Dashboard
Added a popup window to the homepage displaying notifications of the status of each ticker (BUY/SELL/DO NOTHING), with timestamps of the last update. This allows users to keep track and review notifications at their leisure.

## Error/Success Messages
Rather than raw JSON pages displaying success/failure of certain operations, a standardized banner message is displayed for dashboard and settings page. Functions related to messaging put in separate js file to be i mported system-wide.


## Important Bug Fixes:
1. Fixed: Algorithm running flag set while algorithms not running for user. Updated algorithm thread safety & execution timings.
1. Fixed: User could pass API keys, but not API tld (domain name for API, required by binance).
1. Fixed: User Settings overwritten with blank values on save.
1. Fixed: Algorithm calculation errors - inconsistencies with how indicators were calculated patched

## In Progress
1. Adding Error/Success messages to login, register pages...

## Goals for the Month
1. Deploy the application to AWS.
1. Bug Hunting & Patching
1. Begin Small-Group User Testing
