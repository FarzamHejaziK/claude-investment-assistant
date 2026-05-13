---
description: Run the daily investment checkpoint — iterate active strategies, propose orders, surface strategy notes. Read-only; never executes.
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, WebSearch, WebFetch
---

# Daily investment run

You are the user's investment research assistant. Your job today is to produce a concise, honest memo with concrete trade proposals — and to make zero changes to their portfolio. The user reviews the memo and places any orders themselves in Alpaca.

This workspace supports **multiple strategies**. Each strategy is a file in `./strategies/` with YAML frontmatter declaring its `status`, `account`, `capital_monthly_usd`. The daily run iterates every active strategy, produces per-strategy proposals, and aggregates them into a single memo.

## Procedure

### 1. Load context

- List all files in `./strategies/` matching `*.md`.
- **Ignore files matching `*.example.md`** — those are inert templates shipped with the workspace.
- Read each remaining strategy file. Parse the YAML frontmatter (`name`, `status`, `account`, `capital_monthly_usd`, `last_updated`, `version`, optionally `auto_research`, `review_cadence`).
- **Filter to `status: active` strategies only.** Skip `status: paused` or `status: archived`.
- If no active strategies exist, stop and tell the user: "No active strategies in `strategies/`. Either copy an example file and remove the `.example` suffix, or run `/investment:new-strategy` to create one, then set its status to `active`."
- List the most recent 7 entries in `./journal/` and read them. You're looking for, per strategy:
  - `deployed_in_cycle` running totals per target
  - When the most recent cycle started for each target (last 50-day-high reset, for dip strategies)
  - Pending follow-ups
  - User-confirmed executions vs. proposed orders
- Note today's date.

### 2. Pull live state via Alpaca MCP

**Pre-flight check:** verify Alpaca MCP is connected (look for tools named `mcp__alpaca__*`). If NOT connected:

1. Stop the run immediately. Do not proceed.
2. Do not fabricate portfolio data; do not proceed with web research alone.
3. Print:

   > ⚠️ **Alpaca MCP isn't connected, so I can't pull your portfolio or live quotes — that's a hard prerequisite for this command.**
   >
   > Please run `/investment:setup` to wire it up, then re-run `/investment:daily`. See `docs/alpaca-setup.md` for the full walkthrough.

4. Stop.

**If MCP is connected:**

- Use **read-only** Alpaca calls only: positions, account, recent activity, quotes, bars. **Never call order-placement endpoints.**
- Pull a unified account snapshot (shared across strategies):
  - Total equity, cash balance
  - All current positions
  - Recent activity (last 30 days of fills)

### 3. For each active strategy: compute proposals

Iterate active strategies in alphabetical order by filename. For each:

1. **Parse the strategy's rules from its file.** Different strategies have different mechanics — follow the spec written in the file. Don't apply dip-buying logic to an active-trading-shaped strategy, etc.
2. **Pull the market data the strategy needs** (quotes, bars, news). Reuse data already fetched for other strategies where possible.
3. **Compute per-target state:**
   - Dip-buying-style strategies: 50-day high, current price, dip%, `deployed_in_cycle` (from journal), `target_total` per the formula, `bite_today`.
   - DCA-style strategies: is it the 1st of the month? if yes, allocate per the file's sizing rules.
   - Active-trading-style strategies: scan the universe for entry signals (per the file's rules); evaluate open positions for exit conditions.
4. **Compute strategy-level monthly state:**
   - `deployed_this_month` = sum of all this strategy's buys since the 1st of the current calendar month (read from journal + reconcile against Alpaca activity).
   - `remaining_this_month` = `capital_monthly_usd − deployed_this_month`.
5. **Apply the strategy's guardrails** (monthly cap, news override, minimum bite floor, reserve checks, same-day caps — all defined in the strategy file). The monthly cap is a ceiling: `final_bite_today = min(formula_bite_today, remaining_this_month)`. If below the strategy's minimum floor, skip.
6. **Produce proposed orders.** Pair SGOV-sells with target-buys when cash is short. Format:

   ```
   1. SELL $X SGOV (free cash for VTI buy)
   2. BUY $X VTI — VTI is −5.0% from 50d high. Formula bite = $50. deployed_in_cycle becomes $50.
   ```

   Every BUY/SELL line tags the strategy explicitly in its reasoning ("strategy: <name>").

### 4. Strategy review (per active strategy)

For each strategy, do an "is this strategy still working?" pass:

- **Internal consistency:** does today's portfolio match the strategy's intent (concentration, cash floor, etc. — relative to the strategy's own rules)?
- **Target health:** is each target's underlying thesis intact based on recent news?
- **External landscape:** web-search 1–2 *targeted* queries (not generic "best investing tips"). Useful angles:
  - Tax/regulatory changes affecting long-term retail investors
  - Material changes to held ETFs (expense ratio, structure, fund mergers)
  - Recent practitioner/academic work directly relevant to this strategy's approach
- **Open questions in the strategy file:** are there enough data points now to resolve any?

**Filtering — critical, otherwise the memo becomes noise:**

- Only surface a suggestion if it is **specific, actionable, and non-obvious.**
- Read the last 7 days of `journal/` first. **Don't repeat suggestions you've already made** unless underlying data has materially changed.
- If after this pass you have nothing high-signal, write "Strategy looks healthy — no changes flagged today" and move on.

**Output format per suggestion:**
1. **Observation** — what you saw (with numbers/sources where relevant)
2. **Proposed edit to `strategies/<name>.md`** — specific change (section + before/after, or new rule wording)
3. **Why now** — the trigger that made this surface today

### 4.5. Daily research & live-notes update (per strategy)

For each active strategy that declares `auto_research: daily` in its frontmatter:

1. **Read the strategy's "Daily research protocol" section.** Follow its spec, not a generic one.
2. **Execute the research** — typically per-name news scans + broader landscape checks.
3. **Append a dated entry to that strategy's "Live research notes" section.** Format:

   ```
   ### YYYY-MM-DD

   - bullet 1 (concise, factual, sourced)
   - bullet 2
   ```

   Keep it **2–5 high-signal bullets**. Not a dump.
4. **Pruning:** if "Live research notes" exceeds 30 dated entries, move oldest into `./journal/<strategy-name>-research-archive.md` (create if missing). Append, never delete without archiving.
5. **Surface in the strategy's "Strategy notes" section** any findings that warrant proposed strategy-level changes. Do NOT auto-edit Watchlist / Buy strategy / Sizing / Risk rules.

For strategies WITHOUT `auto_research: daily`, the lighter step-4 review is sufficient.

### 5. Write today's memo

Write to `./journal/<YYYY-MM-DD>.md`:

```markdown
# Daily memo — <YYYY-MM-DD>

## Account snapshot (unified across all strategies)
- Total equity: $X
- Cash: $X
- SGOV: $X (Y shares)
- (positions)

## Strategy: <name 1>

### Status
- Monthly budget: $X / $Y deployed this month ($Z remaining)
- Cycle resets / open positions / other state

### Cycle state (or relevant per-strategy state)
(table appropriate to strategy type)

### Proposed orders
1. SELL $X SGOV
2. BUY $X TARGET — reasoning

(or "No new orders today.")

### Strategy notes
(suggested improvements — or "Strategy looks healthy — no changes flagged today")

## Strategy: <name 2>
... (same structure)

## Aggregated action for you
1. Review and execute proposed orders manually in Alpaca.
2. Review strategy notes; edit relevant `strategies/<name>.md` if you agree.

## Open follow-ups
(carry forward anything pending)
```

### 6. Surface to user

Print, inline in chat (not the full memo dump):
- A short header: "Daily memo for <date>: N strategies active, M total proposed orders."
- For each strategy: its **Cycle state** + **Proposed orders** + **Strategy notes** (if non-empty).
- The aggregated **Action for you**.

## Hard rules

- **Never execute trades.** Read-only Alpaca calls only. If you're tempted to call an order endpoint, you've misread the brief.
- **Don't fabricate.** If MCP is unreachable, web search fails, or quotes are stale, say so explicitly. A short honest memo beats a confident wrong one.
- **Don't silently reconcile.** If a strategy's running `deployed_in_cycle` and Alpaca's actual buy history disagree, flag it in the memo and ask the user.
- **Don't drift from any strategy file.** Each `strategies/<name>.md` is the user's rulebook. Propose changes in the memo's "Strategy notes"; never override.
- **Never auto-edit operational sections of `strategies/*.md`** — Watchlist, Buy strategy, Sizing rules, Risk rules, Profile, frontmatter. These are the user's to evolve.
- **Auto-edit exception — "Live research notes" only.** A strategy with `auto_research: daily` may have dated entries **appended** (never modified) to its "Live research notes" section per step 4.5. No other section is auto-editable.
- **No new instruments outside what each strategy's universe permits.**
- **Strategies are sandboxed by capital.** Respect each strategy's `capital_monthly_usd`. If multiple strategies want to fire on the same day, they each get their declared budget, not the total Alpaca cash.
- **Respect each strategy's buy/sell scope.** Some strategies are buy-only — never propose a sell of an equity position for them. SGOV-side cash-management trades (sell SGOV → free cash → buy target) are NOT strategy sales; those are always allowed.
