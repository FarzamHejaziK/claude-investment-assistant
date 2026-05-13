# Designing a strategy

Every strategy in this workspace is a single markdown file in `./strategies/`. The `/investment:daily` command reads each file with `status: active`, follows its rules, and proposes today's orders.

This doc explains the anatomy of a strategy file so you can write your own or modify the examples.

For an interactive builder, use `/investment:new-strategy` — it asks questions and generates a file for you.

## File structure at a glance

```yaml
---
name: my-strategy             # filename should match: strategies/my-strategy.md
status: paused                # paused | active | archived
account: alpaca-paper         # alpaca-paper | alpaca-live
capital_monthly_usd: 500      # how much to deploy per calendar month
horizon: long-term (5+ years) # free text — your intended hold
last_updated: 2026-05-13
version: 0.1
auto_research: daily          # optional — daily run does deep research per this strategy
review_cadence: quarterly     # optional — for active-trading; daily run produces extended summary on quarterly dates
---

# Strategy: <Name>

(one-paragraph plain-English description)

## Profile
## Targets / Watchlist / Universe
## Buy strategy / Entry rules
## Sizing rules
## Risk rules
## Daily research protocol      (only if auto_research: daily)
## Live research notes          (only if auto_research: daily — auto-appended)
## Quarterly review protocol    (only if review_cadence: quarterly)
## Open questions / things to revisit
## Change log
```

## The frontmatter (the YAML block at the top)

These keys control how `/investment:daily` treats the strategy.

| Field | Required? | Values | What it does |
|---|---|---|---|
| `name` | Yes | lowercase-hyphenated | Used for logging and tagging journal entries |
| `status` | Yes | `paused` / `active` / `archived` | `/investment:daily` only processes `active` strategies |
| `account` | Yes | `alpaca-paper` / `alpaca-live` | Tells the daily run which mode is appropriate. Mostly informational; the actual paper/live config lives in your MCP setup. |
| `capital_monthly_usd` | Yes | integer | Monthly budget cap. Total deployed in a calendar month can't exceed this. |
| `horizon` | Yes | free text | Your stated hold horizon. Informational. |
| `last_updated` | Yes | YYYY-MM-DD | When you last edited the file |
| `version` | Yes | semver | Bump when you change rules |
| `auto_research` | No | `daily` | If set, `/investment:daily` does per-name news scans and appends to "Live research notes" |
| `review_cadence` | No | `quarterly` | If set, `/investment:daily` on the 1st of Feb/May/Aug/Nov produces extended trade-log summary |

## The body sections

The body is markdown — write it however you want, but `/investment:daily` looks for certain section names. The required ones are:

### `## Profile`

A short paragraph: who is this for, what's the horizon, what's the risk tolerance, what's the strategy's role in your overall portfolio. Helps future-you remember why you set this up.

### `## Targets` / `## Watchlist` / `## Universe`

The list of tickers this strategy is allowed to trade. **Be explicit and exhaustive** — `/investment:daily` enforces "no instruments outside this list."

For dip-buying: usually 1–3 broad ETFs.
For DCA basket: usually 5–15 names.
For active trading: usually 5–15 names + a few ETFs.

If you have conviction tiers (high / medium / low), break them into separate tables. The daily run treats only the "active rotation" tier as in-scope by default.

### `## Buy strategy` / `## Entry rules`

The mechanics. Tell the daily command exactly when to buy.

For dip-buying: a formula (e.g., `target_total = 5 × dip% + 1 × dip%²`), plus a cycle definition (typically "starts at a new 50-day high"), plus a minimum bite floor.

For DCA basket: when to buy (typically 1st of the month) and how to size each position (typically equal-weight or conviction-tiered).

For active trading: entry signal conditions (e.g., "RSI(14) < 30 AND below 20-day MA"). Should be unambiguous.

### `## Sizing rules`

Position-level constraints. Maximum % per name, maximum concurrent positions, fractional vs. whole shares.

### `## Risk rules`

Hard limits the daily run will enforce: no margin, no options, no leverage, buy-only vs. buy-and-sell, what universes are off-limits, news-override clauses, etc.

### `## Daily research protocol` (optional — `auto_research: daily` only)

Tells the daily run *what kind* of research to do for this strategy. Examples:

> For each watchlist name, web-search for material news in the last 24h: earnings, guidance, M&A, regulatory action.
>
> Also check: hyperscaler capex announcements, HBM cycle news, power-grid news.

The daily command will follow this spec and append findings to the "Live research notes" section.

### `## Live research notes` (optional — `auto_research: daily` only)

Starts empty. The daily run appends dated entries here:

```markdown
### 2026-05-13

- bullet 1
- bullet 2
```

**This is the ONLY section the daily run is allowed to auto-edit.** Everything else is yours.

After ~30 entries, the daily run prunes the oldest into `journal/<strategy-name>-research-archive.md` (move, not delete).

### `## Quarterly review protocol` (optional — `review_cadence: quarterly` only)

Tells the daily run what to do on the 1st of Feb/May/Aug/Nov. Example:

> Produce: trade log summary (count, win rate, avg win/loss, profit factor), comparison vs. buy-and-hold benchmark, verdict on whether the strategy is beating the benchmark. If under-performing for 2 consecutive quarters, propose pausing.

### `## Open questions / things to revisit`

A running checklist. Use it to capture things you'll think about later: "should I add another name?" "is my minimum bite floor too high?" The daily run reads this and may try to resolve items if new data warrants.

### `## Change log`

Every time you bump the `version` in frontmatter, add an entry here explaining what changed and why. This is your strategy's history.

## Three style choices to make

### 1. Buy-only or buy-and-sell?

**Buy-only** (dip-buying and DCA strategies): the strategy never proposes selling equity positions. Cash management (SGOV-side trades) is allowed but not a "strategy sell."

**Buy-and-sell** (active trading): exit rules drive sell proposals. Position turnover is the point.

Set this in the `## Risk rules` section as a hard rule. The daily command enforces it.

### 2. Mechanical or signal-driven?

**Mechanical**: dollar-cost averaging at a fixed cadence. No discretion. Buy on the 1st of every month regardless of price.

**Signal-driven**: rules-based but contingent. "Buy only when dip% > 1.5%." Daily checkpoint evaluates the signal and acts only when it fires.

Both can coexist. The example `dip-buying.example.md` is signal-driven; `ai-value-chain.example.md` is mechanical.

### 3. Universe-isolated or shared?

If you have multiple active strategies, make their universes **non-overlapping** to avoid one strategy accidentally selling another's position. Example:

- `dip-buying.md` → VTI, QQQ
- `ai-value-chain.md` → 8 specific picks-and-shovels names (none overlapping with VTI/QQQ)
- `active-trading.md` → Mag 10 + SPY/IWM/DIA (explicitly NOT VTI/QQQ)

SGOV (cash management) is shared across all strategies — that's fine, since SGOV trades don't affect equity positions.

## Tips

- **Start small** with `capital_monthly_usd`. You can always raise it once a strategy proves itself.
- **Always start `status: paused`** for new strategies. Read the file once or twice before activating.
- **Keep change logs honest.** Future-you will thank past-you for explaining why a rule changed.
- **The "Open Questions" section is for you, not me.** I'll occasionally propose changes via daily memos, but you decide.
- **Don't make 10 small changes.** Make one change, run for a month, see what happens, decide.
