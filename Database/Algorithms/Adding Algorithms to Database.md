# Adding Algorithms to the Boritrader Database
**Boritrade, LLC**
**Last updated:** February 11, 2026

## Overview

This guide explains how to add a trading algorithm record in the Boritrader admin UI, including where to find the algorithm’s HTML description and how to correctly set the display and internal names.

---

## Prerequisites

* Boritrader running locally
* Access to the admin UI

---

## Step-by-step: Add an Algorithm

### 1) Log in

Open the app and sign in:

* `http://localhost:8000/`

### 2) Open the Algorithm table

Go directly to the Algorithm dashboard:

* `http://localhost:8000/sys-admin/dashboard/algorithm/`

### 3) Create a new Algorithm

Click **Add Algorithm**.

### 4) Locate the algorithm documentation file

In the repository, navigate to:

* `Documentation Repository/Database/Algorithms`

Select the applicable **HTML algorithm file**.

---

## Populate the Algorithm fields

### Description

Copy **only the HTML portion** of the algorithm file and paste it into the **Description** field.

Example (HTML portion):

```html
<h3>Description:</h3>
<p>
    Also commonly known as a moving average crossover (MAC), this strategy identifies potential market uptrends using the crossover of short-term (<strong>MA9</strong>) and long-term (<strong>MA50</strong>) moving averages. An <strong>RSI</strong> check is added to confirm trend strength.
</p>

<h4>Entry Conditions (AND):</h4>
<ul>
    <li>MA9 crosses above MA50.</li>
    <li>RSI is greater than 55.</li>
</ul>

<h4>Exit Conditions (Profit Indicator):</h4>
<ul>
    <li>MA9 crosses below MA50.</li>
</ul>

<h4>Purpose and Best Use Case:</h4>
<p>
    The MAC strategy is ideal for traders looking to enter markets at the beginning of a new uptrend. It works well in trending markets where momentum is consistent. The RSI filter helps reduce false signals by ensuring there is sufficient buying pressure to sustain the uptrend. 
</p>
<p>
    This strategy is not optimal in choppy or sideways markets as crossovers can produce whipsaws, leading to frequent but unprofitable trades. For new traders, understanding that this strategy performs best when combined with additional market context, such as overall trend direction or key support and resistance levels, is critical. It allows for informed decision-making when deciding to hold or exit based on external market indicators.
</p>

<h4>Best Case Example:</h4>
<p>
    A stock has been in a consolidation phase for several weeks, then the MA9 crosses above the MA50, and the RSI is at 60. The price begins an uptrend, continuing to rise steadily over the next few weeks, providing the trader with significant gains.
</p>

<h4>Worst Case Example:</h4>
<p>
    The market is experiencing low volatility and price action remains range-bound. The MA9 crosses above the MA50 multiple times, but each time RSI hovers around 55 without strong follow-through. The trader is caught in multiple small trades that result in minimal profits or small losses.
</p>
```

### Display Name and Internal Name

The **last two lines** of the algorithm documentation file must be used to fill in the naming fields:

Example (last two lines):

* `Golden Crossover`
* `golden_crossover`

Set:

* **Display Name:** `Golden Crossover`
* **Internal Name:** `golden_crossover`

---

## Important notes for custom algorithms

* You may choose any **Display Name** and write your own **Description**.
* The **Internal Name must match** the internal key defined in the `strategy_functions` map in the Algorithms file.
* See **`Using Your Own Algorithms.md`** (same directory as this document) for details on registering and mapping custom algorithms.
