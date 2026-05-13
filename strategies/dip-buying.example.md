---
name: dip-buying
status: paused
account: alpaca-paper
capital_monthly_usd: 500
horizon: long-term (10+ years)
last_updated: 2026-05-13
version: 1.0
---

# Strategy: Dip-Buying (example)

Continuous-formula dip-buying on broad US ETFs. Idle cash parked in SGOV. Buy-only, long horizon, mechanical rule.

> **This is a paused example.** To activate: change `status: paused` to `status: active` in the frontmatter above, adjust `capital_monthly_usd` to your real amount, and run `/investment:daily`.

## Profile

- **Capital:** $500/month (adjust to your reality)
- **Horizon:** Long term (10+ years)
- **Style:** Continuous-formula dip-buying on broad market ETFs
- **Risk tolerance:** Moderate. No margin, no options, no leverage.

## Dip targets

| Target | Role |
|---|---|
| **VTI** | Broad-market dip target (US total market) |
| **QQQ** | High-beta dip target (Nasdaq-100, more volatile) |

## Dip-buy formula

For each target, computed daily:

```
dip%         = max(0, 100 × (1 − current_price / 50_day_high))
target_total = 5 × dip%  +  1 × dip%²
bite_today   = max(0, target_total − deployed_in_cycle)
```

Where `deployed_in_cycle` is the running sum of buys into this target since the cycle started.

**Note:** the coefficients `(5, 1)` are calibrated for a $500/month budget. If you scale up, scale the coefficients proportionally — for $5,000/mo use `(50, 10)`, etc. The shape of the curve stays the same; only the magnitude changes.

### Anchor values

| Dip % | Bite total |
|---|---|
| 1% | $6 |
| 2% | $14 |
| 3% | $24 |
| **5%** | **$50** |
| 7% | $84 |
| **10%** | **$150** |
| **15%** | **$300** |
| **20%** | **$500** |
| 25% | $750 (taps SGOV reserves) |

## Cycle & delta-firing mechanic

- A **cycle** starts when the target's price closes at a new 50-day high. `deployed_in_cycle = 0`.
- Each day, compute `target_total`. Then `bite_today = max(0, target_total − deployed_in_cycle)`.
- If `bite_today ≥ $10` (minimum floor), fire it.
- After execution, `deployed_in_cycle += bite_today`.
- **If bite ≤ 0 (price rebounded), fire $0. Do not sell.**
- **Cycle reset:** when price closes at a new 50-day high → `deployed_in_cycle = 0` for that target.

## Guardrails

- **Monthly cap:** total deployed across all targets in a calendar month must not exceed `capital_monthly_usd`. Cap resets on the 1st of each calendar month.
- **Minimum bite floor:** $10. Skip orders below this.
- **Same-day cap across targets:** max 2 fires in a single day.
- **Per-target per-day:** max 1 fire per target per day.
- **Reserve check:** before firing, confirm `cash + SGOV value ≥ bite_today`. If short, sell SGOV first.
- **News override:** if the dip stems from a thesis-breaking event, skip and wait for clarity.

## Idle cash policy

- All idle cash → **SGOV** (iShares 0–3 Month Treasury ETF, ~4.5% yield).
- When dip fires: sell SGOV → buy target. Same-day usable on margin accounts; T+1 on cash accounts.
- No reserve cap. SGOV can grow indefinitely. The strategy embraces extended quiet periods.

## Hard rules

- **Buy-only.** Never sells VTI, QQQ, or any equity position.
- No margin, no options, no leverage.
- No instruments outside VTI / QQQ / SGOV.
- Never average down on a thesis break.
- The `/investment:daily` command proposes orders only; **you execute every trade manually** in Alpaca.

## Open questions / things to revisit

- [ ] Add a 3rd dip target (sector ETF — SCHG / VGT / SMH)? Revisit after 6 months of data.
- [ ] Tax-loss-harvesting policy (if any).
- [ ] Behavior in sustained near-0% rate environment (SGOV yield collapse).

## Change log

- **2026-05-13 — v1.0.** Initial example strategy.
