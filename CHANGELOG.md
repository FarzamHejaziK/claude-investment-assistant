# Changelog

All notable changes to this template are documented here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.1.0] — 2026-05-13

Initial public template release.

### Added
- Three project-level slash commands: `/setup`, `/daily`, `/new-strategy`
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
