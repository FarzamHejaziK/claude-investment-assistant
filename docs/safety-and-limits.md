# Safety and limits

This doc covers what this tool **will not** do, why, and what that means for you. Read it before activating any strategy with real money.

## What this tool is

A workspace template for organizing investment strategies. The `/investment:daily` command reads your strategy files, pulls your Alpaca portfolio (read-only), and proposes specific orders based on the rules you wrote.

## What this tool is NOT

- **Not a robo-advisor.** It doesn't pick strategies for you. You design them.
- **Not financial advice.** It doesn't tell you whether to invest, what to invest in, or how much. It just executes the rules YOU wrote.
- **Not an autotrader.** It does not place orders. You do, manually, in Alpaca.
- **Not regulated.** It's open-source software. There's no licensing, oversight, or accountability beyond what's in the MIT license.

## The propose-only design

This is the most important design choice. Every order is a 3-step process:

1. **The strategy file declares a rule** (e.g., "buy $50 of VTI when it dips 5% from its 50-day high")
2. **The `/investment:daily` command proposes an order** when the rule fires (writes it to today's journal entry, shows it in chat)
3. **You execute the order** by opening Alpaca and placing it yourself

The tool **does not skip step 3**. Even though the underlying Alpaca API supports placing orders, this workspace's settings explicitly deny those tools (`./claude/settings.json` blocks `mcp__alpaca__place_*` and related write endpoints).

### Why propose-only?

1. **Safety**: a buggy strategy file, a bad market move, or a misunderstanding of the rules could trigger oversized or wrong trades. Manual execution forces a human checkpoint.
2. **Learning**: the value of this tool is that you build investment discipline by following rules. Auto-execution makes you a passive observer of your own portfolio. Manual execution makes you a participant.
3. **Compliance**: software that places trades on a user's behalf operates under different legal and regulatory expectations than software that produces recommendations. Propose-only stays on the safer side.
4. **Recoverability**: if anything ever does go wrong, the consequence is a memo you ignore — not a trade you can't unwind.

## What if I want to auto-execute?

You technically could:
1. Remove the `deny` entries for order-placement tools in `.claude/settings.json`
2. Modify `.claude/commands/daily.md` to call `mcp__alpaca__place_stock_order` after generating proposals

**Don't do this.** Specifically:
- The strategy files have not been tested for auto-execution risks (timing, slippage, order sizing edge cases)
- Mistakes in a strategy file would result in real trades, not just bad memos
- You lose the daily checkpoint with reality
- If your account is hacked or your machine is compromised, an attacker could trigger trades through your MCP

If you genuinely want algorithmic auto-execution, use a purpose-built platform (Alpaca's own SDK, QuantConnect, etc.) — not this tool.

## What can go wrong even with propose-only?

The propose-only design prevents *unintended trades*, but it doesn't prevent:

- **Bad strategies losing money.** A poorly-designed strategy will lose money even if you execute every proposed order correctly. The tool just follows the rules you wrote; it doesn't validate that those rules are good.
- **Behavioral errors during execution.** If you skip rules you don't like, or place trades that weren't proposed, the tool can't help. The journal logs what you actually did vs. what was proposed, but it doesn't enforce.
- **Market risk.** Any equity can drop 50%+ in a year. Broad indexes have had 50%+ drawdowns (2008, 2020 briefly). Your account value will fluctuate.
- **MCP/API outages.** If Alpaca's API is down, `/investment:daily` can't pull live data. If Claude Code has issues, you can't run the command. In both cases, you fall back to manual portfolio tracking.
- **Stale strategy files.** If you wrote a strategy six months ago and the market has changed, the rules may no longer make sense. Quarterly review is the official checkpoint, but ultimately YOU need to keep the strategies sensible.

## Specific risks per strategy archetype

### Dip-buying

Mostly safe in mechanics but:
- **Catching falling knives.** A "dip" can become a much bigger drop (e.g., 2008 had a 56% drawdown over 17 months). Without a stop-loss (and dip-buying is buy-only by default), you keep accumulating into deepening drawdowns. The rules in the example expect this — long horizon, accept volatility.
- **Drying up the budget.** If your monthly cap is small and dips deepen fast, you'll exhaust your budget before reaching the bottom. The tier-sizing in the formula mitigates this somewhat.

### DCA basket

Mostly safe:
- **Concentration risk.** If your basket is themed (AI, biotech, etc.), it correlates more than a diversified index. A thesis-break for one name might infect the whole basket.
- **Stale names.** If you don't review the watchlist quarterly, you'll keep DCA'ing into names whose thesis has died.

### Active trading

Highest risk:
- **70–90% of retail active traders lose to a simple index** (per Barber & Odean, FINRA, SPIVA). Your odds aren't great.
- **Tax drag.** Short-term gains are taxed at 2–3× the long-term rate. Compounding effect over years is huge.
- **Behavioral failure.** Rules look great on paper. Following them through 5+ losing trades in a row is psychologically hard.
- **Hard to know if the strategy works.** Coin flips win sometimes too. Distinguishing skill from luck requires 100+ trades and statistical analysis. Most retail traders pull the plug before having data.

The example active-trading strategy ships with a quarterly review protocol that compares the strategy to buy-and-hold of the same universe. **Take that comparison seriously.** If your active strategy loses to buy-and-hold for 2 consecutive quarters, the design recommends pausing.

## Financial disclaimer

This software is not financial advice. Investment decisions are your responsibility. Past performance does not guarantee future results. All investments carry risk, including the risk of partial or total loss of principal.

The author(s) are not licensed financial advisors. The example strategies in this template are intended for educational and illustrative purposes only. They are not endorsements of any particular trading approach or set of securities.

**If you are unsure about an investment decision, consult a qualified financial advisor.**

## What this tool's "I won't do that" rules look like in practice

When Claude is running `/investment:daily` or `/investment:setup` or `/investment:new-strategy`, it will refuse to:

- Place an order on your behalf (even if you tell it to)
- Modify a strategy's operational sections (Watchlist, Buy strategy, Sizing, Risk rules) without your explicit edit request
- Fabricate data when Alpaca is unreachable (it'll stop and tell you instead)
- Silently reconcile when its expected portfolio state disagrees with Alpaca's actual state (it flags discrepancies)
- Recommend specific securities outside the universe each strategy declares

These constraints are baked into `.claude/commands/*.md` and `./claude/settings.json`. They're enforceable; the tool's permissions don't include the ability to do these things.

## Reporting issues

If you find a bug, security issue, or design flaw in this template, open a GitHub issue at https://github.com/FarzamHejaziK/claude-investment-assistant/issues. If it's a security issue specifically (e.g., a way the tool could leak keys or place unintended trades), please report privately first.
