# Institutional Opening Range Breakout (ORB) Backtesting Engine

An institutional-grade quantitative backtesting implementation of the **Opening Range Breakout (ORB)** intraday day-trading strategy. This project builds upon foundational retail implementations (such as *Hacking The Markets*) and significantly modifies the core logic to align with the rigorous academic standards and risk parameters outlined in the benchmark SSRN research paper by Dr. Andrew Aziz and Carlo Zarattini.

---

## Core Strategy & Academic Alignment

The strategy evaluates the initial volatility of the US equity market morning session to establish an institutional trading framework. This repository processes **1-minute intraday data** for highly liquid assets (e.g., `TQQQ`) and evaluates trades strictly against the structural boundaries of the opening session.

### Strategy Rules & Parameters

| Parameter | Value | Description / Academic Context |
| :--- | :--- | :--- |
| **Asset Under Test** | `TQQQ` | Ultra-liquid, high-beta leveraged ETF matching the high-volatility criteria of the paper. |
| **Opening Range Window** | `5 Minutes` | The structural price boundaries are established between `09:30 AM` and `09:35 AM` EST. |
| **Signal Monitoring** | `09:36 AM onwards` | True breakout tracking begins immediately after the opening range window closes. |
| **Risk Management (R)** | `1% of Total Equity` | Strict risk allocation per trade ($1\%$ constant multiplier on dynamic compounding equity). |
| **Take Profit Target** | `2x or 3x Multiple` | Aligns with realistic professional reward-to-risk (2R/3R) targets defined in the paper. |
| **Intraday Leverage Cap** | `4x Buying Power` | Strictly models **Regulation T** retail/institutional intraday margin limitations. |
| **Session Closeout** | `15:59 PM EST` | Pure intraday execution. All open positions are systematically flattened before the closing bell. |

---

## Advanced Risk & Position Sizing Architecture

Position sizes are calculated dynamically on every single trade using a strict **Capital-at-Risk** model rather than a fixed cash allocation model. 

### 1. The Dynamic Sizing Formula
The system determines the exact number of shares to purchase by dividing the maximum allowable cash risk by the absolute distance to the structural invalidation point (the stop-loss):

$$Position\ Size\ (Shares) = \lfloor\frac{\text{Account Equity} \times \text{Risk Percent (0.01)}}{\lvert\text{Entry Price} - \text{Stop Loss Price}\rvert}\rfloor$$

* **Low Volatility (Narrow Range):** Results in tighter stops, automatically scaling up share size to maximize capital efficiency.
* **High Volatility (Wide Range):** Results in wider stops, automatically downsizing share size to insulate the account from outsized losses.

### 2. Dual-Constraint Safety Net
To prevent mathematically valid but physically impossible position sizes on hyper-compressed opening ranges, the position-sizing engine enforces a strict safety boundary:

$$\text{Final Position Size} = \min(\text{Shares by Risk}, \text{Shares by Leverage})$$

Where $\text{Shares by Leverage}$ enforces the maximum regulatory intraday margin buying power ($4 \times \text{Equity}$).

---

## Critical Quantitative Code Fixes Implemented

This repository fixes several mechanical bugs and edge cases found in standard tutorial videos to satisfy strict production and research requirements:

### 1. Resolution of the Over-Trading / Multiple Trade Loophole
Standard retail implementations allow the backtester to re-enter trades indefinitely throughout the afternoon if a position gets stopped out early. This code implements a strict `self.traded_today` state lock. 
* **The Fix:** The moment a long or short breakout condition triggers, the day is immediately locked (`self.traded_today = True`). The system permits **at most 1 trade per session**, eliminating commission-churning whipsaws in choppy markets.

### 2. Zero-Target Value Exception Handling (`ValueError` Patch)
When running backtests with low profit targets (e.g., `take_profit_multiple = 2`) on low-volatility mornings, sudden gapping candles on the execution tick can cause the market entry price to overshoot and match the calculated take-profit price. `backtesting.py` crashes with a `ValueError` when an execution entry equals its exit target.
* **The Fix:** Integrated an operational pre-flight safety check:
  ```python
  # Long Entry Guard
  if take_profit_price > current_close:
      self.buy(...)
  
  # Short Entry Guard
  if take_profit_price < current_close:
      self.sell(...)

---

## Getting Started 

## Prerequisites
Ensure you have the following libraries installed within your Python environment:
```bash 
pip install pandas backtesting bokeh talib-binary data-science-types dotenv
```

## Data Pipeline Configuration
1. Secure a free API tier or premium market data key from an institutional provider (e.g., Alpaca API, Polygon.io).
2. Create a `.env` file in the root directory of this workspace:
```
ALPACA_API_KEY=your_actual_public_key_here
ALPACA_SECRET_KEY=your_actual_secret_key_here
```
3. Run the data ingestion script to download and clean the historical 1-minute OHLCV candles, ensuring timezone localization is shifted cleanly to America/New_York.

---

## References & Acknowledgments
1. Academic Paper: Can Day Trading Really Be Profitable? Evidence of Sustainable Long-Term Profits from Opening Range Breakout (ORB) Day Trading Strategy vs. Benchmark in the US Stock Market by Dr. Andrew Aziz & Carlo Zarattini (SSRN No. 4416622) (https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4416622) 
2. Tutorial Frameworks: Hacking The Markets ORB Architecture Series (https://hackingthemarkets.com/backtesting-py-tutorial/) and Backtesting the Opening Range Breakout (ORB) Strategy in Python using Polygon.io (https://concretumgroup.com/backtesting-the-opening-range-breakout-orb-strategy-using-polygon-io/)
3. YouTube Reference: Part Time Larry (https://youtu.be/HXxHunu_Hkk?si=El5Y63VaIs1xTyE9, https://youtu.be/T3PT4eV8xFU?si=AihGY5sU-gUWHI1E, https://youtu.be/Jmv0xvOg-RQ?si=sMnt8Wchu1Ksjk6H)