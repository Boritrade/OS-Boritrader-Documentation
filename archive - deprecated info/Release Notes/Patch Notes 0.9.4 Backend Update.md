# **The Boritrader Documentation: Patch Notes for TradeClient Update — Sept 5, 2025**
**Boritrade, LLC**   
**Related Release Version:** 0.9.4   

## Added

* \#TestnetPatch

  * `MockBinanceClient`: drop-in client that mimics the subset of python-binance we use:

    * `get_account`, `get_all_tickers`, `get_ticker`, `get_avg_price`
    * `get_historical_trades`, `get_aggregate_trades`
    * `get_klines`, `get_historical_klines` (string lookbacks like “30 days ago UTC”)
    * `get_order_book`, `get_orderbook_tickers`
    * `create_margin_order` (spot-like instant fill, balance updates)
  * Deterministic, seedable portfolio:

    * Starts with `USDT=200k`, `BTC=1`, `ETH=10`
    * Auto-seeds balances for bases in the (up to) 120-symbol universe
  * Price model:

    * Per-symbol base price synthesized from a USD spot table + FX table
    * Small lognormal jitter per call to simulate live moves
  * Symbol universe:

    * Accepts giant CSV; de-dupes and caps to `max_symbols` (default 120)

* \#Mock websockets

  * `MockThreadedWebsocketManager` mirroring `ThreadedWebsocketManager` API:

    * `start()`, `stop()`, `join()`, `start_symbol_ticker_socket()`, `stop_socket()`
    * Emits `24hrTicker`-shaped payloads once per second (random walk, bid/ask, highs/lows)

## Changed

* `BinanceTradeClient`

  * `check_credentials()`:

    * If `testnet=True` → **use `MockBinanceClient`** (and print “Initializing Patched Testnet Binance client.”)
    * Else → create real `binance.client.Client` (unchanged)
  * `open_ticker_socket()`:

    * If using mock client → instantiate `MockThreadedWebsocketManager` with price fn
    * Else → use real `ThreadedWebsocketManager`
    * Store returned stream id in `active_streams`
  * `close_ticker_stream(ticker)`:

    * Stops individual stream if present and removes from `active_streams`
    * If `active_streams` becomes empty → calls `close_all_ticker_streams()`
  * `close_all_ticker_streams()`:

    * No-op guard if `twm is None`
    * Always `stop()`, `join()`, then set `twm=None` and clear `active_streams`
  * `fetch_historical_data()`:

    * Compatible with mock by calling `get_historical_klines`
  * Indicator functions unchanged; validations preserved

## Tests

* Existing tests now pass with the patch when the fixture injects a `MagicMock` client.
* Adjusted expectations:

  * In “testnet enabled” path, we **no longer expect** `binance.client.Client` to be called (mock is used instead). Tests were updated to remove that assertion (or to explicitly test the mock branch).
  * Stream-closing tests updated: `close_all_ticker_streams()` now guards `twm` and sets it to `None` after stop/join. Tests assert that the calls are made **only when `twm` is set**.

## Configuration / Dev ergonomics

* Optional args for the mock:

  * `seed` (default 42) for deterministic portfolios/prices
  * `symbols_csv` (full CSV pasted list) + `max_symbols` to limit scope (e.g., top 100)
* The mock respects common symbol suffixes (`USDT/FDUSD/TUSD/USDC/BUSD/BTC/ETH/BNB` + major fiats).

---

# What still needs doing (near-term)

1. **Environment switch**

   * Add an env flag (e.g., `EXCHANGE_ENV=mock|real`) separate from `testnet` to remove ambiguity.
   * In CI, force `EXCHANGE_ENV=mock` to guarantee hermetic tests.

2. **Fixture utilities**

   * Provide a reusable pytest fixture to boot the mock with a known seed & a curated top-100 symbol CSV for consistent snapshots across test runs.

3. **Docs**

   * README snippet explaining how to:

     * Toggle mock vs real
     * Pass a symbol list
     * Seed the RNG for reproducibility
   * Short section on data shapes the mock guarantees (so future code doesn’t rely on undocumented fields).

4. **Edge-case parity**

   * Add mock behaviors for:

     * Precision/lot-size filters & min notional rejections (raise like Binance would)
     * Fees (maker/taker) applied to fills
     * Partial fills (optional toggle) and cumulative quantities
     * Rate-limit simulation (429s) behind a flag (for resilience testing)

---

# Medium-term options to replace Binance Testnet

## Option A — Keep building out our **own simulated spot testnet** (recommended)

**What:** Evolve the mock into a proper local “exchange simulator” service or module.

**Pros**

* Fully deterministic, always available, no geo/IP drama.
* We control features (partial fills, slippage, fees, outages).
* Excellent for CI, backtests, and load tests; works offline.
* Zero vendor lock for test infra.

**Cons**

* We must maintain realism (filters, rounding, error codes).
* No access to true exchange idiosyncrasies beyond what we model.

**What to add to make it robust**

* Matching engine lite:

  * Limit/market orders, IOC/FOK, partial fills, cancellations
  * Order book state, time priority
* Market data injection:

  * Replay historical candles/trades from files
  * Live “drift + volatility regimes” generator
* Compliance edges:

  * Binance-style filters (PRICE\_FILTER, LOT\_SIZE, MIN\_NOTIONAL, NOTIONAL filters by quote asset)
* Extensibility:

  * Plugin interface for other venues (OKX, Coinbase) using same mock API surface

## Option B — Move non-prod servers to a **Binance-compliant region** and geo-separate users

**Pros**

* Keeps using real Binance Spot Testnet behavior.
* Less to build if you rely heavily on exact exchange quirks.

**Cons**

* Infrastructure, compliance, and operational cost/complexity.
* Still brittle (policy can change again).
* Harder to run fully deterministic tests in CI.

**Verdict:** Feels heavy for our needs and still risky.

## Option C — Hybrid

* Keep our simulator for CI/local/dev.
* Add optional “real exchange smoke tests” executed only from a compliant runner with throwaway keys.

**Pros**: Best of both worlds.
**Cons**: Extra runners/secrets management.

---

# Recommendation

**Go with Option A (own simulator) + light Option C.**

* Treat 0.9.4 as the foundation and:

  1. Formalize it as `trade_client.sim.exchange.MockExchange` with an interface that mirrors our `TradeClientInterface`.
  2. Add order filters, partial fills, and simple fee model first.
  3. Add a “replay mode” (CSV/Parquet candles & trades) to test strategies on real-ish paths deterministically.
  4. Provide scenario knobs: latency, rejects, 429s, websockets hiccups.

Then, if desired, set up a weekly smoke test against a real venue from a compliant runner to catch divergences.

---

# Suggested roadmap

* **release 2**

  * Expand Testnet ticker portfolio
  * Filters: `LOT_SIZE`, `MIN_NOTIONAL`, rounding/precision per symbol
  * Fee model (taker by default)
  * Unit tests for rejects & rounding

* **release 3**

  * Partial fills + order statuses (`NEW/ PARTIALLY_FILLED/ FILLED/ CANCELED`)
  * Limit orders + order book snapshots
  * Replay mode (ingest historical candles/trades)

* **release 4**

  * Rate-limit & error simulation toggles
  * Latency injection
  * Docs & examples (strategy harness using the simulator)

* **Deprecation**
  * Mark `testnet=True` on Binance as **deprecated** in code comments and docs.

 