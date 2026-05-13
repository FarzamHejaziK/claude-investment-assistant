# Getting started

A 15-minute walkthrough from "I just cloned this repo" to "I just ran my first daily checkpoint." Aimed at people with little or no programming background.

## What this is

This is a **template repository** for managing investment strategies using Claude Code. You clone it to your machine, fill in your strategies, and run a daily command that:

1. Reads your strategy files
2. Pulls your live portfolio from Alpaca
3. Tells you what to buy or sell **today**, based on the rules you wrote
4. Writes a dated memo to `journal/` so you have a record

It does **not** place trades for you. It tells you what to do; you click the buy button in Alpaca yourself. (See `safety-and-limits.md` for why.)

## What you need

| Thing | Why | Cost |
|---|---|---|
| **A computer** | macOS, Linux, or Windows | — |
| **An Alpaca brokerage account** | This is where your money lives and where trades happen | Free to open; paper trading is free |
| **Claude Code installed** | The tool that runs the slash commands | Free; install from https://claude.com/claude-code |
| **About 30 minutes** | One-time setup | — |
| **~$0** to start | Use paper trading (fake money) until you trust the system | — |

That's it. No programming knowledge required.

## The 5 steps

### 1. Clone this template

On the GitHub page for this repo, click the green **"Use this template"** button → "Create a new repository." Pick a name (e.g., `my-investments`), make it **private** if you want your data hidden, and click create.

Then on your computer, in Terminal:

```bash
git clone https://github.com/<your-github-username>/my-investments.git
cd my-investments
```

(Replace `<your-github-username>` with your actual GitHub username, and `my-investments` with whatever you named your repo.)

(If you've never used Terminal: it's an app on your computer. On macOS it's called Terminal; on Windows it's PowerShell or "Command Prompt." Open it, then copy/paste the lines above one at a time and press Enter.)

### 2. Open the workspace in Claude Code

Launch Claude Code. Use **File → Open** (or whatever your version calls it) to open the folder you just cloned. Once open, Claude Code automatically detects the `.claude/commands/` directory and makes the slash commands available: `/setup`, `/daily`, `/new-strategy`.

### 3. Run `/setup`

In the Claude Code chat, type `/setup` and press Enter. The wizard walks you through:
- Generating Alpaca API keys (in your Alpaca dashboard)
- Storing them securely (in macOS Keychain, on Mac — other options for Linux/Windows)
- Installing `uv` (a tool that runs the Alpaca connector)
- Wiring everything up
- Verifying the connection by reading your Alpaca account (read-only — no trades placed)

When it prints "✅ Setup complete," you're ready.

### 4. Activate a strategy

In `./strategies/` there are three example strategies:
- `dip-buying.example.md` — buys broad ETFs (VTI / QQQ) when they dip
- `ai-value-chain.example.md` — monthly equal-weight DCA on 8 AI infrastructure picks
- `active-trading.example.md` — buy-and-sell on mean-reversion signals (riskier)

**All three ship `status: paused`** — they don't do anything until you activate one. To activate:

1. Pick the one whose style matches you (most beginners pick dip-buying; it's the simplest)
2. Open the file
3. Change the filename: remove `.example` → `dip-buying.md` (or rename via Finder/your file manager)
4. Edit the frontmatter at the top: change `status: paused` to `status: active`
5. Change `capital_monthly_usd: 500` to whatever you actually plan to deposit each month
6. Save the file

That's it. The strategy is now live.

Or, if you want a custom strategy, run `/new-strategy` — it'll ask you questions and generate a file for you.

### 5. Run `/daily`

Type `/daily` in Claude Code chat. The command:
- Reads your strategy
- Pulls your Alpaca portfolio
- Calculates whether any rules fire today
- Writes a memo to `journal/<today's date>.md`
- Shows you a summary in chat

If a rule fires (e.g., "VTI dipped 3% from its 50-day high"), the memo will propose specific orders like "BUY $24 VTI." You then open the Alpaca app, place that exact order, and reply back to confirm. Tomorrow's run reconciles what you actually placed.

If nothing fires (markets are calm or rising), the memo will say "No new orders today." That's normal. The strategies are designed to be patient.

## What to do next

- **Keep running `/daily`** each market morning (Monday–Friday, ~9:30 AM ET or any time after).
- **Read the daily memo** in `journal/`. Over time these become your trading diary.
- **Don't change the strategy file every day.** Let rules play out for at least a month before tuning.
- **Review your strategy quarterly** (every 3 months) and decide if rules need adjustment. The `Open questions` section in each strategy file is where you list things to revisit.

## Common first-week questions

**Q: I ran `/daily` and it said "Alpaca MCP isn't connected." What now?**
A: The setup didn't finish or Claude Code needs a restart. Quit Claude Code, re-open, and try `/daily` again. If still broken, re-run `/setup`.

**Q: Does this work with brokers other than Alpaca?**
A: Not out of the box. Alpaca is what the MCP connector talks to. You could adapt the strategy logic to other brokers manually (read the memo, place the same trade in Fidelity/Schwab/etc.) but the auto-portfolio-pull won't work.

**Q: Is paper trading really safe? Can I lose money?**
A: Paper trading uses fake money in a simulated Alpaca account. You cannot lose real money in paper mode. Switch to live trading (real money) only after you've tested for at least a month. See `alpaca-setup.md` for how to switch.

**Q: Can the AI place trades for me automatically?**
A: **No.** This is a deliberate design choice — see `safety-and-limits.md`. The tool proposes; you execute. This prevents both accidental large trades and unauthorized money movement.

**Q: What if I want to take a break and not run `/daily` every day?**
A: That's fine. The strategies are designed to be patient. If you skip a week, the next run reads the prior journal entries and figures out what you missed. No data is lost.

## Where to go from here

- `docs/alpaca-setup.md` — Paper vs. live, key rotation, troubleshooting
- `docs/designing-a-strategy.md` — How strategy files are structured, anatomy of each section
- `docs/safety-and-limits.md` — What this tool will and won't do (read before going live)
- `docs/faq.md` — Frequently asked questions

When in doubt, ask Claude Code directly — it has the full context of this workspace and can answer specific questions about your strategy.
