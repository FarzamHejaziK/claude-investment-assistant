---
name: active-trading
status: paused
account: alpaca-paper
capital_monthly_usd: 200
horizon: short-term (days to weeks per trade); long-term (1+ year) for strategy evaluation
last_updated: 2026-05-13
version: 1.0
review_cadence: quarterly
---

# Strategy: Active Trading — Mean Reversion (example)

Daily-checkpoint **buy-and-sell** strategy on a tight universe of large-cap names plus broad ETFs. Mean-reversion entries (oversold-bounce), explicit exit rules (take-profit / stop-loss / time-stop). Quarterly review for strategy effectiveness.

> **This is a paused example.** To activate: change `status: paused` to `status: active`, adjust `capital_monthly_usd` and the universe if you want, and run `/investment:daily`.

> **⚠️ Active trading typically underperforms buy-and-hold for retail investors over multi-year periods.** Multiple academic studies (Barber & Odean, FINRA, SPIVA) consistently find 70–90% of retail active traders underperform. **Treat this strategy as an experiment with capital you can afford to lose.** Quarterly review is the formal kill-switch checkpoint.

## Profile

- **Capital:** $200/month (suggest starting small for active trading)
- **Per-trade horizon:** days to weeks
- **Strategy horizon:** 1+ year before evaluating whether to continue
- **Style:** Daily-checkpoint mean reversion
- **Risk tolerance:** **High.** Capital can lose up to 100%. Quarterly review is the only formal checkpoint.
- **Tax note:** Short-term holds = ordinary-income-rate taxes on gains. Active trading creates more tax drag than buy-and-hold.

## Universe (10 names + 3 ETFs)

### Mag 10 (large-cap individual stocks)

AAPL, MSFT, GOOGL, AMZN, META, NVDA, TSLA, AVGO, AMD, ORCL

### ETFs (broad market)

SPY (S&P 500), IWM (Russell 2000), DIA (Dow)

**Total universe: 13 tickers.** This is the COMPLETE list — strategy never buys outside it.

## Entry rules — buy on oversold-bounce

Daily checkpoint scans the 13-name universe. A BUY signal fires when **EITHER** condition holds:

1. **Oversold + below trend:** RSI(14) closes below 30 AND price closes below 20-day moving average.
2. **Single-day capitulation:** price closes ≥4% below the prior day's close on above-average volume (>1.5× 20-day average daily volume).

If multiple names fire the same day, prefer the one with the **largest %-drop from its 50-day high**.

## Exit rules — sell on whichever fires FIRST

For each open position:

1. **Take profit:** close at or above the 20-day moving average. SELL.
2. **Stop loss:** close ≥5% below entry. SELL.
3. **Time stop:** 10 trading days since entry. SELL regardless of P&L.

## Position sizing

- **Per trade:** $100 at the starter $200/mo allocation (scales upward as strategy proves itself)
- **Maximum concurrent positions:** **1**. Don't stack mean-reversion bets — they're correlated.
- **Idle cash:** parked in SGOV between trades. Sell SGOV to fund a trade entry.

## Quarterly review (every Feb 1, May 1, Aug 1, Nov 1)

The `/investment:daily` run on those dates produces an extended Strategy notes section with:

1. **Trade log:** number of trades, win rate, average win, average loss, profit factor, max drawdown.
2. **Vs. buy-and-hold benchmark:** what did equal-weight DCA across the same 13 tickers return over the same period?
3. **Verdict question:** is the active strategy beating buy-and-hold? **If NO for 2 consecutive quarters → propose pausing the strategy.**
4. **Rule-tuning suggestions:** anything in entry/exit rules that's miscalibrated.

## Risk rules

- **Buy AND sell.** Exit rules drive sells.
- No leverage, no options, no margin.
- No instruments outside the universe above.
- **Loss budget:** capital allocated to this strategy can fall to $0. Quarterly review is the only formal checkpoint.
- **`/investment:daily` proposes orders only; you execute every trade manually in Alpaca.**

## Open questions / things to revisit

- [ ] **First quarterly review:** is the strategy beating buy-and-hold of the same universe?
- [ ] **Universe expansion:** when (if ever) to add more names?
- [ ] **Entry rule tuning:** are signals firing too rarely / too often?
- [ ] **Tax tracking:** short-term gains accumulate; year-end review should report cumulative realized P&L.

## Change log

- **2026-05-13 — v1.0.** Initial example strategy. RSI(14)<30 + below-20d-MA OR -4% capitulation entry. Exit on 20d-MA / -5% stop / 10d time-stop. Max 1 concurrent position. Quarterly review.
