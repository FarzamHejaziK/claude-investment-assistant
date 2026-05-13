# investment-assistant

A workspace template for designing, tracking, and running personal investment strategies with [Claude Code](https://claude.com/claude-code) and [Alpaca](https://alpaca.markets/).

You define your rules in markdown files. Each morning you run one command (`/daily`). It reads your portfolio, checks your rules, and tells you exactly what to buy or sell today. **You execute every trade yourself** — this tool proposes; it doesn't trade on your behalf.

---

> ⚠️ **This is not financial advice.** This is a tool for organizing and tracking *your own* investment decisions. It does not place trades automatically. Past performance does not guarantee future results. **Use paper trading until you've tested for at least a month.** Read [`docs/safety-and-limits.md`](docs/safety-and-limits.md) before going live.

---

## What it does

1. You write strategy rules in `./strategies/<name>.md` files — using plain English plus a YAML header. Three working examples are included.
2. Each market day, you run `/daily` in Claude Code. The command:
   - Pulls your live Alpaca portfolio (read-only)
   - Evaluates each active strategy's rules
   - Writes a dated memo to `./journal/<today>.md`
   - Tells you "BUY $50 of VTI today" or "no action needed" — with reasoning
3. You open Alpaca and place the proposed orders yourself.
4. Next day, the tool reconciles what you actually placed and proposes the next action.

That's the whole loop. Patient, mechanical, and yours to customize.

## What it does NOT do

- ❌ Place trades for you (intentionally — see [`docs/safety-and-limits.md`](docs/safety-and-limits.md))
- ❌ Pick stocks or strategies for you (it follows YOUR rules)
- ❌ Provide financial advice
- ❌ Auto-modify your strategy files (it can suggest changes, you decide)

## Quick start

### What you need
- An [Alpaca account](https://alpaca.markets/) (paper trading is free, no funding required)
- [Claude Code](https://claude.com/claude-code) installed
- About 30 minutes for one-time setup

### 5 steps

1. **Click the green "Use this template" button** at the top of this repo, create your own copy (make it private if you want).
2. **Clone it** locally (replace `<your-github-username>` and `<your-repo-name>` with whatever you used when you clicked "Use this template"):
   ```bash
   git clone https://github.com/<your-github-username>/<your-repo-name>.git
   cd <your-repo-name>
   ```
3. **Open the folder in Claude Code** (File → Open).
4. **Run `/setup`** in the chat. The wizard walks you through: generating Alpaca API keys → storing them securely in your OS keyring → installing the connector → verifying the connection. Strongly recommended: start with paper trading.
5. **Activate a strategy**:
   - Look in `./strategies/`. Three example strategies ship paused.
   - Pick one. Rename it (drop the `.example` suffix). Edit the frontmatter to set `status: active` and adjust `capital_monthly_usd` to your real number.
   - Run `/daily`. Done.

Or create your own: run `/new-strategy` for an interactive builder.

Full walkthrough for non-programmers: [`docs/getting-started.md`](docs/getting-started.md).

## The three example strategies

All ship `status: paused` for safety. Activate by renaming and editing the frontmatter.

| File | Style | Risk | Good for |
|---|---|---|---|
| [`dip-buying.example.md`](strategies/dip-buying.example.md) | Continuous-formula dip-buying on broad ETFs (VTI, QQQ). Buy-only. | Low | Long-term accumulation with opportunistic dip overlay |
| [`ai-value-chain.example.md`](strategies/ai-value-chain.example.md) | Monthly equal-weight DCA across 8 AI infrastructure picks-and-shovels names. Buy-only. Daily research. | Medium (basket concentration) | Themed long-term exposure beyond mega-cap names |
| [`active-trading.example.md`](strategies/active-trading.example.md) | Daily-checkpoint mean reversion on Mag 10 + ETFs. Buy AND sell. Quarterly review. | **High** | Experimentation — most retail active traders underperform |

Read each file to understand what it does before activating.

## Slash commands

| Command | When to run | What it does |
|---|---|---|
| `/setup` | Once, after cloning | First-time wizard: Alpaca API keys, OS keyring storage, MCP wiring, connection verification |
| `/daily` | Every market morning | Reads strategies → pulls portfolio → proposes today's orders → writes journal entry |
| `/new-strategy` | Whenever you want a new strategy | Interactive Q&A; outputs a strategy file in `./strategies/` |

These commands live in `./.claude/commands/`. They're plain markdown — read them to understand what each does, or modify them to change behavior.

## File layout

```
your-investment-workspace/
├── .claude/
│   ├── commands/                  (the three slash commands)
│   │   ├── daily.md
│   │   ├── setup.md
│   │   └── new-strategy.md
│   └── settings.json              (tool permissions — denies Alpaca order endpoints by default)
├── strategies/                    (your investment rules — one file per strategy)
│   ├── *.example.md               (paused examples that ship with the template)
│   └── (your active strategies)
├── journal/                       (daily memos — auto-generated by /daily)
├── docs/                          (documentation)
│   ├── getting-started.md
│   ├── alpaca-setup.md
│   ├── designing-a-strategy.md
│   ├── faq.md
│   └── safety-and-limits.md
├── README.md                      (you are here)
├── LICENSE
├── CHANGELOG.md
└── .gitignore
```

## Documentation

- **[Getting started](docs/getting-started.md)** — 15-minute walkthrough for non-programmers
- **[Alpaca setup](docs/alpaca-setup.md)** — Paper vs. live, generating keys, OS keyring storage, troubleshooting
- **[Designing a strategy](docs/designing-a-strategy.md)** — Anatomy of a strategy file, the three archetypes, tips
- **[FAQ](docs/faq.md)** — Common questions
- **[Safety and limits](docs/safety-and-limits.md)** — What the tool will and won't do, risks per strategy archetype, disclaimer

## How it stays safe

Three layers:

1. **Settings layer.** `./claude/settings.json` explicitly denies the Alpaca MCP's order-placement tools (`mcp__alpaca__place_*`, `close_position`, `cancel_order`, etc.). Claude Code's permission system enforces this — the tool literally can't call them.
2. **Prompt layer.** Every slash command's prompt explicitly says "never execute trades; read-only Alpaca calls only." If Claude tried to violate this, the prompt itself would catch it.
3. **Strategy layer.** Each strategy file's "Hard rules" section enforces strategy-specific limits (buy-only, universe constraints, position caps).

If you want to remove these safety layers (e.g., to enable auto-execution), you can — but you'd be doing so deliberately, with your eyes open. See [`docs/safety-and-limits.md`](docs/safety-and-limits.md) for why we don't recommend it.

## License

MIT. See [`LICENSE`](LICENSE). You can fork, modify, sell, or do anything else permitted by MIT.

## Contributing

This is a template. The best way to "contribute" is to fork and adapt for yourself. If you find bugs in the template itself (the commands, docs, or example strategies), open an issue or PR on this repo.

Security issues: please report privately first.

## Built with

- [Claude Code](https://claude.com/claude-code) — the CLI that runs the slash commands
- [Alpaca](https://alpaca.markets/) — the brokerage and market data API
- [alpaca-mcp-server](https://github.com/alpacahq/alpaca-mcp-server) — the MCP connector for Alpaca

---

*This template is a starting point. The strategies are illustrative. Read each file before activating, customize freely, and remember: every trade is your decision.*
