# Contributing

This is a **template repository**. The expected workflow is:

1. Click **"Use this template"** on the GitHub repo page to make your own copy.
2. Customize it for your own use — your strategies, your rules, your edits to the slash commands.
3. The strategies you write, the journal entries you generate, and any tweaks specific to your investing style stay in *your* fork. They don't come back here.

So when we say "contributing," we mean **improvements to the template itself** — not new investment strategies.

## What kinds of contributions help

- **Bug fixes** in the slash commands (`.claude/commands/investment/*.md`) where the behavior they describe doesn't match what actually happens, or where they have logical holes.
- **Documentation fixes** — typos, broken links, confusing explanations, missing platform notes (Linux/Windows).
- **Improvements to the example strategies** — corrections to formulas, clearer rule wording, better disclaimers.
- **Safety improvements** — additional guardrails, clearer warnings about risks, better defaults.
- **Setup polish** — better error messages, clearer wizard steps, support for additional OS keyrings.

## What's out of scope

- **New investment strategies.** This template ships three examples to illustrate the *shape* of a strategy file. Add new ones in your own fork; don't submit them as PRs here.
- **Auto-execution features.** The propose-only design is deliberate (see [`docs/safety-and-limits.md`](docs/safety-and-limits.md)). PRs that add automatic order placement won't be merged.
- **Financial advice.** This is a tooling project, not an advisory service.

## How to contribute

1. **Open an issue first** for anything non-trivial. Describe the bug or proposal in plain terms. For a typo or one-line fix, jump straight to a PR.
2. **Fork, branch, push.** Branch names should be short and descriptive (`fix/daily-reconciliation`, `docs/clarify-paper-vs-live`).
3. **Test on paper trading first.** Any change that affects `/investment:daily` or how Alpaca calls are made must be tested with `ALPACA_PAPER_TRADE=true`. Live trading is never required (or expected) for development.
4. **Don't commit secrets.** API keys, account numbers, position sizes — none of these belong in any file or in the git history of a PR.
5. **Update docs alongside behavior.** If you change a command, update the matching section in `docs/` and the slash-commands table in [`README.md`](README.md).
6. **Open a PR** with a clear description: what changed, why, and how you tested it.

## Code review

PRs are reviewed personally by the maintainer. Expect a back-and-forth — anything that touches the safety posture (read-only Alpaca, propose-only, no auto-edits to strategy files) gets extra scrutiny. The goal is to keep the template trustworthy for non-programmer users, so be patient and don't take review questions personally.

## Style

- **Markdown formatting:** match the surrounding file. Hyphen bullets, sentence-case headings, no trailing whitespace.
- **Slash command files:** keep `description:` under ~120 chars and accurate. `allowed-tools:` should be the minimum needed.
- **Frontmatter in strategy examples:** always ship `status: paused`. Never `active`.

## License

By contributing, you agree your contributions are licensed under the same MIT license as the project ([`LICENSE`](LICENSE)).
