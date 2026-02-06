# SingleSymbol EMA50 M5 One-Position EA (MT5)

A simple MetaTrader 5 Expert Advisor that trades **one symbol at a time** using a **50 EMA crossover on M5**, with **max 1 open position per symbol**, plus basic risk controls:

- Close at **$20 profit** (default)
- Close at **-$50 loss** (default)
- Start trailing after **$10 profit**
- Optional **XAUUSD weekend block**: **Fri 1:00 PM → Sun 4:00 PM (PDT)**

> File: `SingleSymbol_EMA50_M5_OnePos.mq5`

---

## Strategy Summary

### Entry (M5)
Uses the last **two completed** M5 candles and the **EMA(50)**:

- **Buy** when price crosses **above** EMA:
  - previous close ≤ previous EMA  
  - last close > last EMA  

- **Sell** when price crosses **below** EMA:
  - previous close ≥ previous EMA  
  - last close < last EMA  

### Position limit
- **Max 1 open position per symbol**
- Also prevents opening more than one trade on the same M5 bar using a `lastBarTime` gate.

### Management
For any open position on the symbol:

- **Take Profit**: close if `POSITION_PROFIT >= ProfitTarget`
- **Stop Loss (equity-based)**: close if `POSITION_PROFIT <= StopLossLimit`
- **Trailing Stop** (points-based):
  - activates once `profit >= TrailingStart`
  - moves SL using `TrailingStepPoints` (in symbol points)
  - respects broker minimum stop distance (`SYMBOL_TRADE_STOPS_LEVEL`)
- **Reversal protection after $10 profit**:
  - tracks tickets that reached **≥ $10**
  - if such a position drops to **between $0 and $1**, it closes (tries to “lock something” instead of letting it go negative)

---

## Trading Window (PDT)

The EA converts server time to PDT using:

- `tNow = TimeCurrent() + PDT_OFFSET_HOURS * 3600`

Then enforces:

- Trades only between:
  - `TradeStartHourPDT` (inclusive)
  - `TradeEndHourPDT` (exclusive)

Defaults:
- Start: **7 AM PDT**
- End: **24 (midnight) PDT**

> Note: if your broker server time shifts (DST, different timezone), you may need to adjust `PDT_OFFSET_HOURS`.

---

## XAUUSD Weekend Block (Optional)

If enabled (`UseXAUWeekendBlock = true`) and symbol is **XAUUSD**:

- Blocks trading:
  - **Friday 1:00 PM PDT and later**
  - **All Saturday**
  - **Sunday before 4:00 PM PDT**

Also:
- At **exactly Friday 1:00 PM PDT**, it attempts to **close all open XAUUSD positions**.

---

## Inputs

| Input | Default | Meaning |
|------|---------|---------|
| `Lots` | `0.30` | Order size |
| `ProfitTarget` | `20.0` | Close position when profit (USD) reaches this |
| `StopLossLimit` | `-50.0` | Close position when loss (USD) reaches this |
| `TrailingStart` | `10.0` | Start trailing when profit (USD) reaches this |
| `TrailingStepPoints` | `5.0` | Trailing distance in **points** (not pips) |
| `EMAPeriod` | `50` | EMA period |
| `PDT_OFFSET_HOURS` | `-7` | Offset from UTC to PDT (adjust for DST if needed) |
| `TradeStartHourPDT` | `7` | Daily trade start hour (PDT) |
| `TradeEndHourPDT` | `24` | Daily trade end hour (PDT, exclusive) |
| `UseXAUWeekendBlock` | `true` | Enable XAUUSD Fri/Sun block |

---

## Installation (MT5)

1. Open **MetaTrader 5**
2. Go to **File → Open Data Folder**
3. Navigate to:
   - `MQL5/Experts/`
4. Copy `SingleSymbol_EMA50_M5_OnePos.mq5` into that folder
5. Restart MT5 (or right-click **Experts** → Refresh in Navigator)
6. Open **MetaEditor**, compile the file
7. Attach the EA to the chart:
   - Symbol you want to trade
   - Set chart timeframe to **M5** (recommended)

---

## Recommended Setup

- Run on **M5** chart of the symbol you want to trade.
- Enable Algo Trading.
- Test first on **Strategy Tester** (or demo), especially if you tweak trailing / stop rules.

---

## Logging

The EA writes trade exits to a CSV file:

- `TradeLog.csv`

Fields:
- Symbol
- Entry Time, Entry Price
- Exit Time, Exit Price
- Profit/Loss
- Exit Reason (TP Hit, SL Hit, Reversal Exit, Friday Close)

In MT5, you can find the file in:
- **File → Open Data Folder → MQL5/Files/**

---

## Notes / Gotchas

- **TrailingStepPoints is in points**, not pips. On some symbols, 5 points is tiny. Adjust per symbol.
- This EA uses **profit in account currency (USD)** to manage TP/SL/trailing start.
- If your broker has a large `SYMBOL_TRADE_STOPS_LEVEL`, trailing might not move often.
- Time handling uses a fixed offset (`PDT_OFFSET_HOURS`). If you want it to auto-handle DST, you’d need extra logic.

---

## Disclaimer

This is for educational/testing purposes. Trading involves risk. Use at your own discretion and always forward-test before live trading.
