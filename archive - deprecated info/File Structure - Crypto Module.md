# The Boritrader Documentation: Crypto Module
**Boritrader LLC**

## Module Overview

The `crypto` module has been refactored into separate GitHub-hosted packages for better modularity, enhanced testing capabilities, and maintainability. Each package contains its own interfaces, unit tests, and setup instructions. This modular structure improves the scalability of the project and simplifies the integration process.

### New Packages and Repositories:

- `Boritrader_Bot/` https://github.com/Eborishade/Boritrader_TradeBot
- `Boritrader_Algorithms/` https://github.com/Eborishade/Boritrader_Algorithms
- `Boritrader_TradeClients/`https://github.com/Eborishade/Boritrader_TradeClients

---
## Detailed Explanation

### `Boritrader_Bot`

Here is an updated and detailed explanation of the `Boritrader_Bot` package to better illustrate its structure and logic:

---

### `Boritrader_Bot`

- **Purpose**: This package is responsible for managing trade execution and integrating with external data sources. It serves as the core trading bot that interacts with various exchange APIs, processes data, and executes trading strategies. The `BoritraderBot` class encapsulates logic for both live and simulated trading, providing a robust foundation for automated and manual trading configurations.

- **Key Components**:
  - **`BinanceTrader.py` (BoritraderBot class)**:
    - **Overview**: The main class for managing trades, executing trading algorithms, and handling user-specific configurations.
    - **Attributes**:
      - `data_handler`: An instance of `DataHandler`, used to manage and retrieve configuration data, including API keys and user preferences.
      - `client`: An instance of `BinanceTradeClient`, which handles API interactions with Binance.
      - `enabledAlgorithms`: A set of enabled trading algorithm IDs.

  - **`tools/` Directory**:
    - **`DataHandler.py`**:
      - **Purpose**: An interface that outlines how data is fetched and managed. It ensures consistency for data retrieval and configuration handling.
      - **Implementation**: Provides the blueprint for implementations like `FileFetcher`.
    - **`FileFetcher.py`**:
      - **Purpose**: An implementation of `DataHandler` that loads configuration data from local storage and assists in populating necessary data for the `BinanceTrader`.
      - **Role**: Ensures that API keys and user-specific settings are fetched and passed to the trading bot for proper initialization.
      - **Integration**: Utilizes `LoadStore.py` for data persistence tasks, supporting file-based data handling.
    - **`LoadStore.py`**:
      - **Purpose**: Contains utility functions for reading from and writing to files. Assists `FileFetcher` in data management.
    - **`SendNotifications.py`**:
      - **Status**: Deprecated. Previously used for sending notifications but no longer actively maintained or required.

  - **`enums/` Directory**:
    - **To be phased out in future updates.**
    - **Purpose**: Contains enumerations for testing that standardize certain configurations and action types within the bot. 
    - **Examples**:
      - **`coins.py`**: Enumerates supported coin types.
      - **`trade_actions.py`**: Lists possible trading actions (e.g., BUY, SELL).

  - **`tests/` Directory**:
    - **Purpose**: Contains unit tests for validating the bot’s logic and functionalities.
    - **Key Files**:
      - **`test_BinanceTrader.py`**: Tests that verify the behavior and output of methods within `BinanceTrader.py`.
      - **`configuration.py`**: Supplies configurations used during testing.

### Planned Enhancements:
- **Interface Implementation**: The `BinanceTrader.py` class currently directly references `BinanceTradeClient`. Future modifications will introduce an interface to enable `BinanceTrader` to support multiple exchange clients seamlessly. This change will enhance the flexibility and adaptability of the bot to work with additional exchanges beyond Binance.
- **DataHandler and Configuration**: The bot relies on `DataHandler` (via `FileFetcher` currently) to load and manage configuration settings, including API keys and trading preferences. This approach ensures that different implementations of `DataHandler` can be swapped in as needed for different data sources.

### Key Notes:
- The **`DataHandler` interface** enforces a standard structure for any data retrieval implementation, ensuring consistent interaction between the `BoritraderBot` and data-fetching classes.
- The **`FileFetcher`** is essential for pulling user-specific configuration data, such as API keys, which the bot requires for authenticating with the `BinanceTradeClient`.
- The **current dependency on `BinanceTradeClient`** will evolve to support other exchange clients, allowing `BoritraderBot` to integrate with a broader range of trading platforms through an interface-based approach.


### `Boritrader_Algorithms`

- **Purpose**: Contains the core trading algorithms that implement various strategies used for automated trading.
- **Key Components**:
  - **`Algorithms.py`**: Defines trading strategies like Moving Averages, RSI, and MACD, along with signal generation and backtesting logic.
  - **`interfaces/base_algorithm.py`**: Provides an abstract base class for algorithmic trading, ensuring a standard structure for new strategies.
  - **`tests`**:
    - **`test_algorithms.py`**: Unit tests to validate the functionality of different algorithms.
    - **`configuration.py`**: Supplies test data and configurations.


### `Boritrader_TradeClients`

- **Purpose**: Interfaces with the exchange APIs for data fetching and order management.
- **Key Components**:
  - **`BinanceTradeClient.py`**: Contains the `Trade_Client` class for interacting with the Binance API.
  - **`interfaces/TradeClientInterface.py`**: Defines the abstract interface to ensure uniform implementation of trade clients.
  - **`tests`**:
    - **`test_BinanceTradeClient.py`**: Unit tests to validate API interactions.
    - **`configuration.py`**: Provides test-specific configurations.

---

## Workflow Overview

### Configuration

Each package references configuration files for its tests and general functionality. Environment variables required for secure data access include:

- **READ_ONLY_PAT**: GitHub Personal Access Token for accessing private repositories.
- **GIT_API_KEY**: Binance API key for testnet usage.
- **GIT_SEC_KEY**: Binance secret key for testnet usage.

### Data Flow

1. **Initialization**:
   - The `Boritrader_Bot` initializes the main trading processes, linking with algorithm strategies and the trade client.
2. **Strategy Analysis**:
   - The `Boritrader_Algorithms` processes data from a `Trade_Client` interface to generate signals.
3. **Data Fetching**:
  - The `Trade_Client` in `Boritrader_TradeClients` retrieves market data.


### Integration with the Application Framework

These packages are built to integrate into a microservices architecture and work cohesively with broader application components like user interfaces and database services.


## Testing and Verification

### Build Setup & Requirements for Testing

1. **Create a Virtual Environment**:
   ```bash
   virtualenv myenv
   source myenv/bin/activate
   ```

2. **Set Environment Variables**:
    ```bash
    export READ_ONLY_PAT="your_pat_here"
    export GIT_API_KEY="your_binance_api_key_here"
    export GIT_SEC_KEY="your_binance_secret_key_here"
    ```

3. **Install Dependencies**:
   ```bash
   pip install -r requirements.txt
   pip install -r requirements_git.txt
   ```

4. **Run Tests**:
   ```bash
   pytest
   ```