# Adding Custom Trading Algorithms
**Boritrade, LLC**  
**Updated February 10, 2026**  

Learn how to create and integrate your own trading strategies into Boritrader.

---

## 📋 Overview

Boritrader's trading algorithms come from the **Boritrader_Algorithms** package, which is automatically installed from GitHub when you start the application. This guide shows you how to:

1. Understand the algorithm structure
2. Create your own custom algorithms
3. Add them to the database
4. Use local algorithms (instead of pulling from GitHub every time)

---

## 🏗️ How Algorithms Work

### The Algorithm Dependency

**Installed from**: `requirements_git.txt`

```txt
git+https://github.com/Boritrade/Boritrader_TradeBot.git@main#egg=binance_trader
git+https://github.com/Boritrade/Boritrader_TradeClients.git
git+https://github.com/Boritrade/Boritrader_Algorithms.git  ← Algorithm package
```

**When installed**: Every time you run `docker-compose up`
- The `entrypoint.sh` script runs: `pip install -r /app/requirements_git.txt`
- This **overwrites** the package with the latest from GitHub

**Current algorithms** included:
1. `golden_crossover` - Moving Average Crossover Strategy (MACS)
2. `buy_the_dip` - Catching The Bottom Strategy
3. `reversal_rsi` - Maximized RSI Strategy  
4. `simple_trend_ride` - Ride The Trend Strategy
5. `dollar_cost_averaging` - DCA Strategy
6. `pump_and_dump` - Pump and Dump Strategy
7. `trailing_stop_loss` - Trailing Stop Loss (not fully implemented)
8. `stop_loss` - Stop Loss (not fully implemented)

---

## 🎯 Algorithm Structure

### The Interface (BaseAlgorithm)

All algorithms must inherit from `BaseAlgorithm` and implement the `evaluate()` method:

```python
from boritrader_algorithms.interfaces.base_algorithm import BaseAlgorithm
from trade_client.interfaces.TradeClientInterface import TradeClientInterface

class BaseAlgorithm(ABC):
    @abstractmethod
    def evaluate(self, trade_client, symbol: str) -> (bool, bool):
        """Method to evaluate trading conditions.
        
        Parameters:
            trade_client: The trade client for API interactions
            symbol: Trading pair (e.g., 'BTCUSDT')
            
        Returns:
            (entry_condition, exit_condition): Two booleans
        """
        pass
```

### Return Values

Your algorithm must return **two booleans**:

1. **`entry_condition`**: `True` = Enter trade (buy signal)
2. **`exit_condition`**: `True` = Exit trade (sell signal)

**Important**: Entry conditions are prioritized:
```python
if entry_condition:
    # Enter trade
elif exit_condition:
    # Exit trade
else:
    # Hold
```

---

## 📝 Creating Your Custom Algorithm

### Step 1: Understand the Template

Here's a complete example algorithm:

```python
from boritrader_algorithms.interfaces.base_algorithm import BaseAlgorithm
from trade_client.interfaces.TradeClientInterface import TradeClientInterface

class MyCustomAlgorithm(BaseAlgorithm):
    def evaluate(self, trade_client: TradeClientInterface, symbol: str) -> (bool, bool):
        """
        My custom trading strategy.
        
        Strategy Logic:
        - Entry: Buy when RSI < 30 (oversold)
        - Exit: Sell when RSI > 70 (overbought)
        
        Parameters:
            trade_client: The Binance trade client
            symbol: Trading pair (e.g., 'BTCUSDT')
            
        Returns:
            (entry_condition, exit_condition): Two booleans
        """
        # Get market data and calculate indicators
        indicators = trade_client.calculate_indicators(
            symbol, 
            interval='1h',      # 1-hour candles
            lookback='1 month'  # Past month of data
        )
        
        # Extract the latest RSI value
        latest_rsi = indicators['rsi'].iloc[-1]
        
        # Define your trading logic
        entry_condition = bool(latest_rsi < 30)  # Buy when oversold
        exit_condition = bool(latest_rsi > 70)   # Sell when overbought
        
        return entry_condition, exit_condition
```

### Step 2: Available Indicators

The `trade_client.calculate_indicators()` method returns a pandas DataFrame with:

**Available indicators**:
- `close` - Closing price
- `open` - Opening price
- `high` - High price
- `low` - Low price
- `volume` - Trading volume
- `rsi` - Relative Strength Index (14-period)
- `ma9` - 9-period Moving Average
- `ma50` - 50-period Moving Average
- `ma100` - 100-period Moving Average
- `ema50` - 50-period Exponential Moving Average
- `ema100` - 100-period Exponential Moving Average

**Time intervals**:
- `'1m'`, `'5m'`, `'15m'`, `'30m'`
- `'1h'`, `'2h'`, `'4h'`, `'6h'`, `'12h'`
- `'1d'`, `'3d'`, `'1w'`, `'1M'`

**Example usage**:
```python
# Get 4-hour candles for the past month
indicators = trade_client.calculate_indicators(symbol, interval='4h', lookback='1 month')

# Access specific indicators
latest_rsi = indicators['rsi'].iloc[-1]       # Most recent RSI
latest_ma9 = indicators['ma9'].iloc[-1]       # Most recent MA9
previous_rsi = indicators['rsi'].iloc[-2]     # Previous RSI
```
---

## 🔧 Integrating Your Algorithm

### Method 1: Fork and Modify the Boritrader_Algorithms Package (Recommended)

**Best for**: Permanent custom algorithms you want to maintain

**Steps**:

1. **Fork the repository**:
   - Go to: https://github.com/Boritrade/Boritrader_Algorithms
   - Click "Fork" (top right)
   - Now you have your own copy

2. **Clone your fork locally**:
```bash
git clone https://github.com/YOUR-USERNAME/Boritrader_Algorithms.git
cd Boritrader_Algorithms
```

3. **Edit `Algorithms.py`**:
```python
# Add your new algorithm class
class MyCustomAlgorithm(BaseAlgorithm):
    def evaluate(self, trade_client: TradeClientInterface, symbol: str) -> (bool, bool):
        # Your logic here
        return entry_condition, exit_condition

# Add to the strategy functions map
strategy_functions = {
    'golden_crossover': GoldenCrossoverStrategy(),
    'simple_trend_ride': SimpleTrendRideStrategy(),
    # ... existing strategies ...
    'my_custom_strategy': MyCustomAlgorithm(),  # ← Add here
}
```

4. **Commit and push**:
```bash
git add Algorithms.py
git commit -m "Add custom algorithm: my_custom_strategy"
git push origin main
```

5. **Update `requirements_git.txt`** in your Boritrader project:
```txt
git+https://github.com/Boritrade/Boritrader_TradeBot.git@main#egg=binance_trader
git+https://github.com/Boritrade/Boritrader_TradeClients.git
git+https://github.com/YOUR-USERNAME/Boritrader_Algorithms.git  ← Use your fork
```

6. **Restart Container**:
```bash
docker-compose down
docker-compose up # no need to rebuild, git repos refreshed every restart
```

---

## 💾 Adding Algorithm to Database

Once your algorithm is in the package, you must add it to the database to use:

### Option 1: Via Admin Panel (Easy)

1. **Navigate to**: http://localhost:8000/sys-admin/
2. **Login** with superuser credentials
3. **Click**: "Algorithms"
4. **Click**: "Add Algorithm"
5. **Fill in**:
   - **Display Name**: `My Custom Strategy` (what users see)
   - **Internal Name**: `my_custom_strategy` (must match `strategy_functions` key)
   - **Description**: Explain when to use this strategy

**Example**:
```
Display Name: Advanced Momentum
Internal Name: my_custom_strategy
Description: Combines MA crossover, RSI momentum, and volume surge. 
             Best for trending markets with strong momentum.
```

6. **Save**

### Option 2: Via Django Shell (Fast)

```bash
docker-compose exec django python /app/djangostuff/manage.py shell
```

```python
from dashboard.models import Algorithm

# Add your algorithm
Algorithm.objects.create(
    display_name='Advanced Momentum',
    internal_name='my_custom_strategy',  # Must match strategy_functions key!
    description='Combines MA crossover, RSI momentum, and volume surge. Best for trending markets.'
)

print("✅ Algorithm added!")
exit()
```



---

## 🎯 Internal Name Mapping

**Critical**: The `internal_name` in the database **MUST** match the key in `strategy_functions`:

**In `Algorithms.py`**:
```python
strategy_functions = {
    'my_custom_strategy': MyCustomAlgorithm(),  # ← Key is 'my_custom_strategy'
}
```

**In Database**:
```
Internal Name: my_custom_strategy  # ← Must be EXACT match
```

**If these don't match**: You'll get an error when users try to enable the algorithm.

---

## 🧪 Testing Your Algorithm

### Manual Testing

1. **Add to database** (as shown above)
2. **Login** to Boritrader web interface
3. **Go to Dashboard**
4. **Enable** your new algorithm for a trading pair
5. **Monitor** the signals in the dashboard
6. **Check logs**:
```bash
docker-compose logs -f django | grep -i "my_custom_strategy"
```

### Unit Testing

Boritrader_Algorithms package also comes with test file. Feel free to add tests and run as needed using pytest.



## 📚 Best Practices

### 1. Error Handling

Always check for sufficient data:

```python
def evaluate(self, trade_client, symbol):
    indicators = trade_client.calculate_indicators(symbol, interval='1h', lookback='1 month')
    
    # Check we have enough data
    if len(indicators['rsi']) < 2:
        raise ValueError("Insufficient data for this strategy")
    
    # Your logic here
```

### 2. Use Boolean Casting

Always cast conditions to `bool`:

```python
# Good
entry_condition = bool(latest_rsi < 30)

# Bad (might return numpy.bool_ which causes issues)
entry_condition = latest_rsi < 30
```

### 3. Handle Edge Cases

```python
# Check for NaN values
if pd.isna(latest_rsi):
    return False, False  # No signal if data is invalid

# Prevent division by zero
if latest_close == 0:
    return False, False
```

### 4. Document Your Strategy

Include clear docstrings:

```python
def evaluate(self, trade_client, symbol):
    """
    My Custom Strategy
    
    Entry Logic:
    - Condition 1: ...
    - Condition 2: ...
    
    Exit Logic:
    - Condition 1: ...
    
    Best Use Case:
    - Works well in trending markets
    - Avoid during high volatility
    
    Risk Level: Medium
    Recommended Timeframe: 1h - 4h
    """
```

---

## 🐛 Troubleshooting

### "Unknown strategy: my_custom_strategy"

**Cause**: Internal name mismatch

**Fix**:
1. Check `strategy_functions` in `Algorithms.py`:
```python
strategy_functions = {
    'my_custom_strategy': MyCustomAlgorithm(),  # ← This key
}
```

2. Check database:
```bash
docker-compose exec django python /app/djangostuff/manage.py shell
```
```python
from dashboard.models import Algorithm
algo = Algorithm.objects.get(display_name='My Custom Strategy')
print(algo.internal_name)  # Should match the key above
```

### Changes Not Reflecting

**Cause**: Old package cached

**Fix**:
```bash
# Force rebuild
docker-compose down
docker-compose build --no-cache django
docker-compose up
```

---

## 🎯 Quick Reference

### Algorithm Checklist

- [ ] Inherits from `BaseAlgorithm`
- [ ] Implements `evaluate(trade_client, symbol)` method
- [ ] Returns `(bool, bool)` tuple
- [ ] Checks for sufficient data
- [ ] Casts conditions to `bool()`
- [ ] Has clear docstring
- [ ] Added to `strategy_functions` map
- [ ] Internal name matches database entry
- [ ] Tested manually or with unit tests

### File Locations

| What | Where |
|------|-------|
| Algorithm code | `dependencies/Boritrader_Algorithms/Algorithms.py` (local)<br>or your GitHub fork |
| Database entry | Django admin (http://localhost:8000/sys-admin/) or shell |
| Configuration | `requirements_git.txt` (fork method)<br>`Dockerfile` (local method) |
| Tests | `dependencies/Boritrader_Algorithms/tests/` |
