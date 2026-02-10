# The Boritrader
**Boritrade, LLC**
**Last Updated:** January 30, 2025  

The Boritrader is designed to facilitate automated cryptocurrency trading decisions on platforms like Binance. Users visit our website to sign up, customize, and utilize our application, making for an entirely digital experience, ideal for day traders looking to scan the entire market and for passive traders aiming to mitigate losses and pinpoint optimal entry points.

Our mission is to democratize access to advanced trading algorithms and financial tools, traditionally reserved for professionals, by providing an intuitive platform that empowers users to manage and grow their wealth. We strive to close the gap between expert traders and everyday investors, offering a solution that is both accessible and effective, enabling our users to trade with confidence and success.

---
### Features
- **Automated Trading**: Executes trades based on predefined strategies.
- **Real-time Market Data**: Fetches and processes live market data.
- **User Account Syncing**: Securely syncs user data straight from the exchange.
- **Flexible Strategies**: Allows users to tailor their own trading strategies.
- **Advanced Logging**: Implements robust logging mechanisms so users can see all activity.
- Currently, it supports the Binance API with plans to introduce additional features & platforms soon.
---


# Project Structure
This application is built primarily using Python and the python-binance API. The front-end is built using the Django framework, containing HTML, CSS, and JavaScript, with a PostgreSQL database integration. The entire instance is configured to be compiled and run out of a Docker container using docker compose. This project is developed and maintained through several Github repositories & packages, and is scheduled to be deployed on an AWS EC2 instance.

## The project is organized into several key repositories:

https://github.com/Boritrade/Boritrader_TradeBot
- Contains the trade-client package
- Main Class - BinanceTradeClient:
    - Represents a Binance trading bot with various functionalities for interacting with the Binance API,
        executing trading algorithms, and managing user-specific trading configurations.


https://github.com/Boritrade/Boritrader_Algorithms
- Contains the algorithms package
- Main Module - Algorithms.py:
   - Utility File containing all functions related to buying/selling rules & strategies.
   - dependencies:
    - BinanceTradeClient

https://github.com/Boritrade/Boritrader_TradeClients
- Contains the binance-trader package:
- Main Class: BinanceTrader:
   - Represents a client for interacting with the Binance API for trading purposes.
   - Dependencies:
    - Algorithms
    - BinanceTradeClient


https://github.com/Boritrade/Boritrader-open
- **djangostuff/**: Contains all files necessary to host instance on web.
    - config/: holds primary django project configuration settings
    - dashboard/: holds all files related to the app dashboard
        - static/: holds all css & js files
        - templates/: holds all html files
    - dependencies:
        - BinanceTrader
        - Algorithms
        - BinanceTradeClient

 

For detailed project documentation: https://github.com/Boritrade/Boritrader-Documentation