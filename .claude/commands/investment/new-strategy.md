---
description: Interactive strategy builder — ask questions, generate a new strategy file in ./strategies/.
allowed-tools: Read, Write, Glob
---

# Interactive strategy builder

You are helping the user create a new investment strategy file. Walk them through it like a conversation — **one question at a time**, wait for their answer, then move on. Don't dump all the questions at once. Don't write the file until the user has answered everything and approved the draft.

## Procedure

### 1. Greet and orient

> "Let's build a new strategy. I'll ask you a few questions, then generate a strategy file in `./strategies/`. I won't run it for you — once it's written, you decide when to set `status: active` and run `/investment:daily`.
>
> First question: **what do you want to call this strategy?** Use lowercase, hyphens for spaces, no extension. Example: `my-tech-basket` or `bond-ladder`. This becomes the filename `strategies/<name>.md` and the `name:` in frontmatter."

**Validate the name:**
- Must be unique (not already a file in `./strategies/`)
- Lowercase, alphanumeric + hyphens only
- No `.example` suffix
- If they pick a taken name, ask for a different one

### 2. Strategy type

> "What type of strategy is this? Pick one:
>
> 1. **Dip-buying** — formula-based, fires when a target dips from its 50-day high. Buy-only. Good for: long-term accumulation on broad ETFs.
> 2. **DCA basket** — monthly mechanical buys across a basket of names. Buy-only. Good for: themed exposure (e.g., AI value chain, dividend growers, sector tilts).
> 3. **Active trading** — buy AND sell on signals (RSI, capitulation, etc.). Risky. Good for: experimentation with capital you can afford to lose.
>
> Type the number."

Branch based on their choice. Each branch has its own follow-ups.

### 3a. Dip-buying branch

Ask in order:
1. **Universe** — which tickers? Suggest examples (VTI, QQQ, SCHG, etc.). 1–3 names usually best.
2. **Capital per month** — `capital_monthly_usd`. (Remind them: this strategy is fully tactical; the budget is just the cap on monthly deployment.)
3. **Anchor calibration:** "At what dip level do you want a meaningful bite? Example: 'at 5% dip, I want a $50 bite' or 'at 10% dip, $500.'" Use their answer to derive formula coefficients via `target_total = a × dip% + b × dip%²` calibrated to match at 5% AND 20%. Show them the derived curve before continuing.
4. **Minimum bite floor** — usually 2% of `capital_monthly_usd` is a good default. Skip orders below.
5. **Idle cash policy** — park in SGOV (recommended) or hold as cash?
6. **Horizon** — long-term (5+ yrs)? Capital allocation can rebalance later if so.

### 3b. DCA basket branch

Ask in order:
1. **Universe** — which tickers, and conviction tier? List them as "high" or "medium" — high gets active rotation; medium gets monitor-only.
2. **Capital per month** — `capital_monthly_usd`.
3. **Buy day of month** — typically 1st (or any day; calendar-driven).
4. **Sizing** — equal-weight across high-conviction names (simple, recommended), or conviction-weighted (50% high / 35% medium / 15% reserve)?
5. **Daily research** — `auto_research: daily`? If yes, the daily run will do per-name news scans and append findings to a "Live research notes" section. If the basket is themed (e.g., AI, biotech), this is valuable. If it's broad-market (e.g., dividend-aristocrats), probably overkill.
6. **Horizon** — typically 5+ years for basket strategies.

### 3c. Active-trading branch

**Before asking specifics, warn them:**

> "Heads up: active trading typically underperforms buy-and-hold for retail investors over multi-year periods. Multiple academic studies (Barber & Odean, FINRA, SPIVA) consistently find 70–90% of retail active traders lose to a simple index. **Treat this strategy as an experiment with capital you can afford to lose.** Quarterly review is the formal checkpoint. Still want to proceed?"

If yes:
1. **Universe** — what names/ETFs? Suggest tight (5–15 names) of liquid large-caps.
2. **Capital per month** — `capital_monthly_usd`. **Suggest starting small** — $100–$200/month max.
3. **Edge** — what asymmetric thing are you trying to capture? Options:
   - Mean reversion (RSI / oversold-bounce)
   - Momentum (12-1 month or breakout)
   - News/earnings reactions
   - User-defined
4. **Entry rules** — translate the edge into concrete conditions (e.g., "RSI(14) < 30 AND price below 20-day MA").
5. **Exit rules** — take-profit, stop-loss, time-stop. **All three are required** (no open-ended trades).
6. **Position size and concurrency** — $/trade and max concurrent positions.
7. **Risk budget** — what %-loss of allocated capital triggers a pause? Quarterly review is standard.
8. **Tax acknowledgement** — short-term gains taxed at ordinary income rate; confirm user accepts.

### 4. Common follow-ups for ALL branches

After the type-specific questions:

1. **`account:`** — `alpaca-paper` or `alpaca-live`? (Informational metadata about which Alpaca account this strategy is designed for; the actual paper/live mode is set by the `ALPACA_PAPER_TRADE` env var in your MCP config, chosen during `/investment:setup`.)
2. **`auto_research: daily`?** (already asked for DCA; ask for the other types here too)
3. **Final sanity check:** show the user the proposed values they've given. Confirm.

### 5. Draft the file

Generate the strategy file content based on their answers. Use the structure of the matching `.example.md` file in `./strategies/` as a template — copy the section structure, fill in user-specific values, write a one-paragraph "Profile" describing the strategy in plain terms.

**Default frontmatter:**
```yaml
---
name: <user choice>
status: paused        # ALWAYS start paused — user activates manually
account: <user choice>
capital_monthly_usd: <user choice>
horizon: <user choice>
last_updated: <today>
version: 0.1
auto_research: <user choice; omit if false>
review_cadence: <if active-trading, "quarterly"; else omit>
---
```

**Always start `status: paused`.** Never write `status: active` automatically. The user activates by manual edit.

**Always include these sections** (adapted to strategy type):
- `# Strategy: <name>` (one-line description)
- `## Profile`
- `## Watchlist` / `## Targets` / `## Universe` (label per type)
- `## Buy strategy` / `## Entry rules`
- `## Sizing rules`
- `## Risk rules`
- `## Open questions / things to revisit`
- `## Change log` (seeded with v0.1 entry)

For active-trading strategies, also add `## Quarterly review protocol`.

For strategies with `auto_research: daily`, also add `## Daily research protocol` and an empty `## Live research notes` section.

### 6. Show the draft

Print the generated file content to chat. **Do not write it yet.** Ask the user to review:

> "Here's the draft. Read through it. Anything wrong? Anything to add to the Open Questions section? Tell me 'looks good' to write it, 'change X' to revise, or 'cancel' to scrap."

If they say "looks good," write the file. If they ask for changes, iterate. If they cancel, don't write anything.

### 7. Write and confirm

After they approve:
1. Write to `./strategies/<name>.md`.
2. Confirm:

   > "✅ Strategy `<name>` written to `strategies/<name>.md`. It's `status: paused` — open the file and change to `status: active` when you're ready to run it via `/investment:daily`. Edit any rules in the file freely; the daily run will respect what's there."

## Hard rules

- **Always start `status: paused`.** Never auto-activate.
- **Validate filename:** lowercase, hyphens, no `.example` suffix, must be unique.
- **Never write to an existing file** — refuse and ask for a different name.
- **Always include a clear disclaimer** in the generated file (in the Profile or at the bottom): "This strategy is a tool for tracking the user's own decisions. The `/investment:daily` command proposes orders; the user executes manually."
- **Never propose violating the strategy's own rules.** If the user describes an active-trading strategy without exit rules, push back and require them.
- **Show the user the draft before writing.** No surprise files.
