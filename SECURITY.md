# Security policy

This project is a **personal investment workspace template**. It connects to a brokerage API (Alpaca) and handles financial data, so security matters here even though the codebase is small.

## What this repo touches

- **Alpaca API keys** — stored in your OS keyring (macOS Keychain by default), never in the repo.
- **Live brokerage account data** — positions, balances, activity. Read-only by design (order-placement tools are denied in `.claude/settings.json`).
- **Trade decisions** — proposed in `journal/<date>.md` memos, never executed automatically.

## Threat model

- **API key leaks** are the highest-impact issue. If your keys leak, an attacker can call any Alpaca endpoint with your account credentials.
- **Strategy file or journal tampering** could mislead future runs. Strategy files are version-controlled in your fork; review the diff before merging anything.
- **Supply-chain risk** from MCP servers (Alpaca's MCP, plus any others you wire up). Pin versions where you can.
- **Hallucinated market data.** This is mitigated by the propose-only design — a wrong number can mislead a memo but cannot move real money. Reports about misleading or fabricated data are still welcome.

## Reporting a vulnerability

**Please do not file a public issue for security problems.**

Preferred channel: **GitHub Security Advisories** on this repo — the **"Security" tab → "Report a vulnerability"** button. This creates a private discussion with the maintainer.

If you can't use Security Advisories, contact the maintainer directly via GitHub (`@FarzamHejaziK`) and request a private channel.

## What to include in a report

- A clear description of the issue and its impact.
- Steps to reproduce (commands, file contents, expected vs. observed behavior).
- The commit SHA you tested against.
- Whether it affects paper accounts, live accounts, or both.
- Any suggested mitigation, if you have one.

## What to expect

- Acknowledgement within a few days (this is a side project; not a 24/7 response).
- A back-and-forth to reproduce and scope the issue.
- A patch released as a regular commit, with security context documented in [`CHANGELOG.md`](CHANGELOG.md) once the fix is public.
- Credit in the changelog if you'd like (or anonymous if you'd prefer).

## What is NOT a security issue

- **Losing money on a strategy you wrote.** This template proposes; you execute. Trading losses are not a vulnerability.
- **A strategy file with weird math.** Bugs in example strategies are bugs — open a normal issue.
- **General disagreement with the propose-only design.** That's a design discussion, not a security report. The reasoning lives in [`docs/safety-and-limits.md`](docs/safety-and-limits.md).

## Recommendations for users

- **Use paper trading until you've run at least a month** without surprises.
- **Don't commit your own strategies to a public fork** if they reveal portfolio composition or net worth.
- **Rotate Alpaca keys** if your Keychain or local machine is ever compromised — regenerate in the Alpaca dashboard and re-run the Keychain storage step.
- **Audit `.claude/settings.json`** before each release. The `deny` list there is part of the safety story; don't merge changes that remove entries from it without understanding why.
