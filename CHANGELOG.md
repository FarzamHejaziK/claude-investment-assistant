# Changelog

All notable changes to this template are documented here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Changed
- Paper trading is no longer the recommended "default" mode. `/investment:setup` now asks **paper or live** up front, with both presented as first-class options. Docs (README, getting-started, alpaca-setup, faq, safety-and-limits) updated to remove paper-first prescriptive language while keeping factual descriptions of both modes. Strategy file `account:` field clarified as informational only — the actual paper/live mode is controlled by the `ALPACA_PAPER_TRADE` env var in the MCP config.

### Added
- `/investment:setup` now walks the user through **each example strategy** and asks for a **per-strategy monthly budget** as part of the setup flow. Previously users had to manually rename `.example.md` files and edit `capital_monthly_usd`; now the wizard does it for them (or accepts the example default, or skips). All strategies still default to `status: paused`; user must explicitly opt in to activate. Active-trading gets an extra confirmation step that re-surfaces the underperformance warning before activation.

## [0.1.0] — 2026-05-13

Initial public template release.

### Added
- Three project-level slash commands: `/investment:setup`, `/investment:daily`, `/investment:new-strategy`
- Three example strategies, all shipped as `status: paused` for safety:
  - `dip-buying.example.md` — continuous-formula dip-buying on broad ETFs
  - `ai-value-chain.example.md` — DCA basket of AI infrastructure picks-and-shovels
  - `active-trading.example.md` — daily-checkpoint mean-reversion on a curated universe
- Documentation:
  - `getting-started.md` — 15-minute walkthrough for non-programmers
  - `alpaca-setup.md` — paper/live keys, Keychain storage, MCP wiring
  - `designing-a-strategy.md` — anatomy of a strategy file
  - `faq.md`
  - `safety-and-limits.md` — propose-only design, financial disclaimer
- `.gitignore` to keep sensitive files out of git
- `.claude/settings.json` with sensible default tool permissions
