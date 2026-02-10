# Algorithm Central Execution Runner Performance Analysis
**Boritrader LLC**   
**Last Updated:** June 5, 2025  

## Question, how many of these end in `USDT`?
MARKET SNAPSHOT MAY 29, 2025, 5:41PM
[from file: `market_snapshot_5.29.25.md`]

```
tokens = symbols.split(',')
usdt_tokens = [s for s in tokens if s.endswith('USDT')]
print("Count of symbols ending with 'USDT':", len(usdt_tokens))

STDOUT/STDERR:
Count of symbols ending with 'USDT': 305
```
- ***Note: Actual performance results indicates `396` tickers ending in USDT.***

## Performance Analysis
That 305‐ticker loop per algorithm dictates worker count.  A simple way to size it is:

> workers ≥ (N × tₐᵥg) / T

Where:
* N = 305 tickers
* tₐᵥg = your average execute\_algorithm runtime per ticker
* T = your target loop time (120 s)

So, for example, if each call takes \~1 s, you’d need

> 305 s total / 120 s ≈ 2.5 ⇒ round up to at least 3 workers

In practice you’ll have some overhead (DB writes, notify\_user, context‐switching), so bump that by 2×–4×.

– **CPU-bound**: match your physical cores (e.g. 8 cores ⇒ 8 workers)
– **I/O-bound/network-blocked**: 2–4× cores (e.g. 16–32 if you’ve got 8 cores)

And if you benchmark and find tₐᵥg ≈ 0.5 s, you’d get
>(305×0.5)/120 ≈ 1.3 ⇒ so 2–4 workers should finish comfortably  

**Recommendation:**

1. Measure your average per-ticker execution time.
2. Plug into the formula above to get a hard minimum.
3. If it’s CPU-heavy, set workers ≃ #cores; if it blocks on I/O, try 2–4× #cores.

Most teams start around 8–12 workers on an 8-core box, then tune up or down once they’ve got real timing data.


Here’s a ball-park of what to expect, expressed both as formulas you can plug your own measurements into and with a concrete example assuming typical per-ticker timings:

---

## 1. Calls per loop

* **Algorithms**: 6
* **Tickers per algo**: 305
* **Total `execute_algorithm` calls**:

  $$
    N_{\text{calls}} = 6 \times 305 = 1{,}830
  $$

---

## 2. Threading & chunking

* **Workers**: 12
* **Per-algo workers**:

  $$
    W_{\text{per\_algo}} = \max\bigl(\lfloor 12/6\rfloor,\,1\bigr) = 2
  $$
* **Chunks per algo**: 2, each ≈153 tickers
* **Futures submitted**: 6 algos × 2 chunks = 12 tasks in parallel

---

## 3. Database writes

Each task does one bulk-insert of its results (≈153 rows) with `batch_size=500`, so:

* **Bulk-inserts per loop**: 12
* **Rows per insert**: \~153
* **Total rows per loop**: 12 × 153 = 1,836 (off by rounding) ≈1,830
* **Latency per insert**: typically 5–20 ms on a modern Postgres/MySQL server
* **Total DB time**: roughly

  $$
    12 \times (5\text{–}20\text{ ms}) \;\approx\; 60\text{–}240 ms
  $$

---

## 4. Notifications (WebSocket)

* **Calls to `notify_user`**: 1,830 per loop
* If each push takes \~1 ms of I/O/serialization, total is on the order of \~1.8 s—but these happen interleaved with the algorithm work and won’t block the bulk-insert.

---

## 5. Algorithm execution & wall-clock time

Let

* $t_{\text{avg}}$ = your measured **average wall-clock time** per `execute_algorithm` call (in seconds).

Then:

* **Total sequential time** = $1{,}830 \times t_{\text{avg}}$
* **Parallel wall-time** (≈ the longest single task, since they run concurrently):

  $$
    T_{\text{wall}} \approx \frac{305}{W_{\text{per\_algo}}}\;\times\; t_{\text{avg}}
    = 152.5 \times t_{\text{avg}}
  $$

| Example $t_{\text{avg}}$ |  0.1 s |  0.2 s |  0.5 s |
| :----------------------: | :----: | :----: | :----: |
|  **Parallel wall-time**  | 15.3 s | 30.5 s | 76.3 s |

---

## 6. CPU utilization

Let $c_{\text{avg}}$ = your measured **CPU‐time** per call (in seconds). Total CPU‐seconds per loop:

$$
  C_{\text{total}} = 1{,}830 \times c_{\text{avg}}
$$

Average cores used:

$$
  \text{Cores} = \frac{C_{\text{total}}}{120\text{ s}}
$$

| $c_{\text{avg}}$ | 0.05 s | 0.1 s | 0.2 s |
| :--------------: | :----: | :---: | :---: |
|  **CPU-seconds** | 91.5 s | 183 s | 366 s |
|  **Cores used**  |  0.76  |  1.53 |  3.05 |

---

## 7. Putting it all together

* **Expected loop time** = algorithm wall-time (\~15–80 s) + DB writes (\~0.1 s) + notification I/O (\~1–2 s).
* **Slack** = 120 s − loop time (so you still sleep some seconds each cycle).
* **DB load** = \~12 INSERT statements every 2 minutes—trivial for any modern RDBMS.
* **CPU load** depends entirely on your per-call costs; use the formulas above to size your box or dial the worker count.

---

### Next steps

1. **Benchmark** one algorithm over, say, 100 tickers: record wall-time and CPU-time per call.
2. **Plug** those numbers into the formulas above.
3. **Tune** $_WORKER_COUNT$ so that

   * Wall-time ≪ 120 s (so you’re not cutting into the next cycle too much), and
   * CPU cores used ≲ your available cores (to avoid thrashing).

That will give you a solid, empirical performance profile for your exact environment.





On Fargate/ECS, your “CPU units” map directly to vCPUs (where 1 vCPU = 1,024 CPU units). Inside your Django container you’ve allocated:

* **`django_cpu = 512`** CPU units ⇒ **0.5 vCPU**
* **`django_memory = 1 024 MB`**

A “worker” in your Python `ThreadPoolExecutor` is just a thread, and if your code is CPU-bound (e.g. pure-Python math, tight loops) then only one thread can run on one vCPU at a time (and the GIL may further limit you to one at a time in CPython).

---

### Sizing for CPU-bound work

* **Rule of thumb:** *workers ≃ number of vCPUs*
* You have **0.5 vCPU**, so you can effectively drive *one* CPU-bound thread.
* If you crank up to 1.0 vCPU (1,024 units), you could go to *workers = 1–2* (one to match cores, two only if you’re okay with a bit of context-switching).

### Sizing for I/O-bound or mixed work

If your `execute_algorithm` spends significant time waiting (HTTP calls, DB I/O, async notifications that release the GIL), you can oversubscribe:

* **Rule of thumb for I/O-bound:** *workers = 2–4 × vCPUs*
* With 0.5 vCPU, that suggests *workers ≈ 1–2*.
* You’ll only get so much CPU—but when one thread goes to sleep on I/O, another can run.

---

### What this means for your `_WORKER_COUNT`

| Scenario           | vCPUs | Recommended `_WORKER_COUNT` |
| ------------------ | :---: | :-------------------------: |
| CPU-bound          |  0.5  |              1              |
| Moderate I/O-bound |  0.5  |             1–2             |
| Pure I/O-bound     |  0.5  |              2              |

If you find you need more parallelism, you’ll either need to:

1. **Increase your container CPU** (e.g. bump to 1 vCPU by setting `django_cpu = 1024`),
2. **Spread work across multiple ECS tasks**, or
3. **Switch to a process pool** (for true multi-core CPU use in CPython).

Start with `_WORKER_COUNT = 1` (or `2` if you see plenty of I/O wait) and benchmark your loop time and CPU usage. If your loop finishes well under 120 s and CPU stays ≲ 50 %, you can raise the count; if you see context-switch churn or CPU contention, dial it back.


# Potential Performance Improvements

With hundreds of symbols to scan on every two-minute tick, the current single-threaded “for each algo, for each ticker” loop is going to start taking noticeable time (and pushing load onto both your own DB and Binance). Here are some high-level strategies you can apply:


## 1. Cache “all tickers” at startup (and on change) [IN PROGRESS]

Binance’s list of trading pairs won’t change every two minutes, so pull it once on startup (or when you detect a listing change) rather than inside every loop. For example:

```python
# at module level
_MASTER_TICKERS = None

def _get_master_tickers():
    global _MASTER_TICKERS
    if _MASTER_TICKERS is None:
        _MASTER_TICKERS = get_or_create_master_bot().get_available_tickers()
    return _MASTER_TICKERS
```

Then in your runner you refer to `_get_master_tickers()` instead of calling `.get_available_tickers()` each time or inside `populate_tickers`.

---

## 2. Parallelize per‐algorithm or per‐ticker work

Rather than nesting two loops in the same thread, use a `ThreadPoolExecutor` (or `ProcessPoolExecutor` if you need true CPU concurrency) so you can run, say, each algorithm on its block of tickers concurrently:

```python
from concurrent.futures import ThreadPoolExecutor

_executor = ThreadPoolExecutor(max_workers=8)

def run_one_algo_on_all_tickers(algo, tickers, master_bot):
    for t in tickers:
        entry, exit = master_bot.execute_algorithm(algo, t)
        # …persist & notify…

def run_central_algorithm_loop():
    master_bot = get_or_create_master_bot()
    algos = [p["algorithm_id"] for p in master_bot.get_configuration_settings()['algorithm_preferences']]
    tickers = _get_master_tickers()
    while True:
        futures = []
        for algo in algos:
            # You can even partition your tickers list to smaller chunks
            futures.append(_executor.submit(run_one_algo_on_all_tickers, algo, tickers, master_bot))
        # wait for all of them (or as they finish)…
        time.sleep(120)
```

That way your 400+ tickers get spread over 8 threads rather than one.

---

## 3. Bulk‐insert your SignalResults [DONE]

Right now you call `SignalResult.objects.create()` for each ticker. That’s one SQL `INSERT` per symbol. Instead, accumulate your per‐run rows into a list of `SignalResult(...)` objects and then call:

```python
SignalResult.objects.bulk_create(rows, batch_size=200)
```

That single bulk insert is vastly faster than hundreds of individual inserts.

---

## 4. Push notifications in batches [DONE]

Likewise, if you know you’re going to send one WebSocket message per ticker, consider grouping them: e.g. send a single “batch” payload like:

```json
{"type": "batch_notifications", "notifications": [ {...}, {...}, ... ]}
```

And then decode on the client into individual UI updates. That reduces channel layer overhead.

---

## 5. Prune old SignalResults asynchronously [IN PROGRESS]

Rather than deleting everything older than 30 days each time you insert, schedule a once-a-day background task (with your automations or using Celery beat) to:

```python
SignalResult.objects.filter(created_at__lt=now()-timedelta(days=30)).delete()
```

This keeps your run loop lean.

---

## 6. Only run algorithms on users’ *subscribed* tickers [REJECTED]

If many tickers have no active user subscriptions, you can scan your `AlgorithmPreferences` table once per loop to find the union of all tickers your users actually care about—drop the others entirely.

```python
subscribed_tickers = AlgorithmPreferences.objects\
    .values_list('tickers', flat=True)
subscribed = set().union(*subscribed_tickers)
```

Then only call `execute_algorithm` on that reduced set.

---

### Putting it all together

Here’s a sketch of how your central runner could look:

```python
from concurrent.futures import ThreadPoolExecutor

_executor = ThreadPoolExecutor(max_workers=8)

def run_partitioned(algo, tickers, bot):
    rows = []
    notifs = []
    for t in tickers:
        entry, exit = bot.execute_algorithm(algo, t)
        sig = "BUY" if entry else "SELL" if exit else "DO NOTHING"
        rows.append(SignalResult(algorithm_name=algo, ticker=t, signal=sig, result_message=f"..."))
        notifs.append({"algorithm_name": algo, "ticker": t, "event_type": sig})
    SignalResult.objects.bulk_create(rows, batch_size=200)
    # group_send a single batch message, or loop small groups…

def run_loop():
    bot = get_or_create_master_bot()
    algos = [p["algorithm_id"] for p in bot.get_configuration_settings()['algorithm_preferences']]
    executor = _executor
    while True:
        subscribed = get_subscribed_tickers()  # union of all user prefs
        for algo in algos:
            # You can slice subscribed into N equal partitions to feed each thread
            executor.submit(run_partitioned, algo, list(subscribed), bot)
        time.sleep(120)
```

—
By caching tickers, parallelizing your algorithm execution, bulk‐inserting, and pruning outside your hot path, you’ll drastically reduce both CPU and I/O costs. Let me know if you’d like concrete code snippets for any of these pieces!

---
---

# Notes: Concurrency Within a Container (Docker, ECS/Fargate)
In Docker (and by extension on ECS/Fargate), your containers are just Linux processes constrained by cgroups. Concurrency inside the container works exactly the same way—threads still share a GIL, processes each get their own interpreter and GIL—but the kernel enforces whatever CPU quota or affinity you’ve given the container.

1. **CPU quotas & visible cores**

   * When you run a container with, say, `--cpus=0.5` (or on Fargate you assign 512 CPU units), Docker sets up a cgroup quota so that the container can only ever use one half-core worth of CPU time.
   * By default, Python’s `multiprocessing.cpu_count()` will still return the host’s full core count, but the scheduler will never let your container processes exceed their cgroup quota.
   * If you want your code to detect its actual quota, you can read `/sys/fs/cgroup/cpu/cpu.cfs_quota_us` and `cpu.cfs_period_us` (or use a library like `psutil`), or constrain the visible CPUs via `--cpuset-cpus`.

2. **ThreadPoolExecutor vs ProcessPoolExecutor**

   * **Threads** inside Docker behave just like threads on the host: they contend for the GIL and share the same half-core budget. So with a 0.5-core quota, only one thread can be computing Python bytecode at a time, and all threads together can’t exceed 50% of a core.
   * **Processes** (via `ProcessPoolExecutor` or `multiprocessing`) still get their own interpreter and GIL, but they all draw from that same half-core budget. You can spawn multiple processes, but the sum of their CPU use will be limited by the cgroup.

3. **Practical takeaways**

   * If you’ve given your container 1 vCPU (1,024 units) on ECS, you effectively have one core to play with. Inside Docker, you’ll see full-speed CPU up to that core, and you can run one process (or one thread) at full speed.
   * If you bump your container to 2 vCPUs, you now have two cores worth of quota. A `ProcessPoolExecutor(max_workers=2)` will give you near-linear speed-ups on pure-CPU code.
   * There’s no special Docker “gotcha” beyond remembering that cgroups enforce the limit: your concurrency model (threads vs processes) still behaves the same, but it can’t exceed the CPU time your container is allocated.

In short: Docker simply wraps your Python processes in a cgroup. Your threading vs. multiprocessing strategy doesn’t change, but you size your workers to the vCPUs (or fractional vCPUs) your container has been given.

