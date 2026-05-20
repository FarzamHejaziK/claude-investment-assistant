---
description: First-time setup wizard — wires up the Alpaca MCP, stores keys in macOS Keychain, verifies the connection.
allowed-tools: Bash, Read, WebFetch
---

# First-time setup

You are walking a non-programmer through setting up this workspace. Be patient, ask one thing at a time, confirm progress before moving on, and explain *why* at each step.

The goal of this wizard: get the user from "fresh clone" to "able to run `/investment:daily` successfully."

## Procedure

### 1. Confirm the basics

Ask the user, one at a time:
- Are they on **macOS, Linux, or Windows**? (Affects Keychain availability — macOS has it natively; on Linux/Windows we'll use a different storage option.)
- Do they already have an **Alpaca brokerage account**? If no, point them to https://alpaca.markets/ and pause here.
- **Paper or live?** Quick framing — both are fully supported, neither is the "default":
  - **Paper** — simulated money, simulated account, zero financial risk. Great for: learning the tool's behavior, validating a strategy you wrote, getting comfortable with the daily routine.
  - **Live** — real money in a real Alpaca account. Real returns, real losses. Great for: deploying a strategy you've already validated, or going direct if you have prior conviction and accept the risk.

Whichever they pick, the only differences in the rest of the wizard are: (a) the Alpaca dashboard URL where keys are generated, and (b) the `ALPACA_PAPER_TRADE` env var value (`true` for paper, `false` for live). Otherwise the flow is identical.

**If they pick live**, briefly confirm they understand:
- Proposed orders, when executed by the user in Alpaca, move real money.
- This is propose-only — Claude never auto-trades — but the user still clicks "buy" and lives with the outcome.
- They can later switch modes by generating new keys in the other Alpaca panel, updating Keychain, and re-running `claude mcp add` with the opposite `ALPACA_PAPER_TRADE` value.

Once confirmed, store their choice as `<mode>` (`paper` or `live`) and use it consistently in the steps below.

### 2. Generate Alpaca API keys

Walk them to the right Alpaca dashboard page:
- **Paper:** https://app.alpaca.markets/paper/dashboard/overview
- **Live:** https://app.alpaca.markets/live/dashboard/overview

In the right sidebar, find the **"API Keys"** panel. Tell them to click **"Generate New Key"**. The modal shows two strings:

1. **Key ID** — looks like `PK...` (paper) or `AK...` (live). This is `ALPACA_API_KEY`.
2. **Secret Key** — long alphanumeric. This is `ALPACA_SECRET_KEY`.

**Warn them clearly:** the Secret Key is shown **only once**. If they close the modal without copying both, they have to regenerate (which invalidates the old pair).

### 3. Store the keys securely

For **macOS** users — store in Keychain. Run these commands one at a time; each prompts for the value silently (no echo, no shell history):

```bash
read -s "k?Paste ALPACA_API_KEY: " && echo && \
  security add-generic-password -a "$USER" -s alpaca-api-key -w "$k" -U && unset k

read -s "s?Paste ALPACA_SECRET_KEY: " && echo && \
  security add-generic-password -a "$USER" -s alpaca-secret-key -w "$s" -U && unset s
```

Tell the user to run each in their **own terminal** (not via Claude Code). The `-s` flag means typing is hidden; the `-U` flag means "update if exists."

After both are stored, verify (this is safe to run via Claude — it doesn't echo full values):

```bash
echo "alpaca-api-key prefix: $(security find-generic-password -a "$USER" -s alpaca-api-key -w | cut -c1-2)"
echo "alpaca-secret-key length: $(security find-generic-password -a "$USER" -s alpaca-secret-key -w | wc -c)"
```

Expected: prefix `PK` (paper) or `AK` (live), secret length ~44 chars.

For **Linux / Windows** users — the Keychain commands won't work. Use one of these instead:
- **Linux:** `secret-tool` (libsecret) or `pass` (gpg-based password store)
- **Windows:** Credential Manager via PowerShell `New-StoredCredential`
- **Cross-platform fallback:** put the values in `.env` (gitignored — never commit this) and source it before `claude mcp add`

Adjust the wizard's next step accordingly.

### 4. Install `uv` (if not already present)

The Alpaca MCP server runs via `uvx`, which ships with `uv`. Check if installed:

```bash
which uv || echo "uv not found"
```

If not found:

- **macOS (Homebrew):** `brew install uv`
- **Linux/macOS (curl installer):** `curl -LsSf https://astral.sh/uv/install.sh | sh`
- **Windows (PowerShell):** `powershell -c "irm https://astral.sh/uv/install.ps1 | iex"`

After install, also install a managed Python (the macOS system Python is often broken on Apple Silicon):

```bash
uv python install 3.12
```

### 5. Wire up the Alpaca MCP

For **macOS** users:

```bash
claude mcp add alpaca \
  --scope user \
  --transport stdio \
  --env ALPACA_API_KEY="$(security find-generic-password -a "$USER" -s alpaca-api-key -w)" \
  --env ALPACA_SECRET_KEY="$(security find-generic-password -a "$USER" -s alpaca-secret-key -w)" \
  --env ALPACA_PAPER_TRADE=true \
  -- uvx --python 3.12 alpaca-mcp-server
```

**Important** — the `ALPACA_PAPER_TRADE` env var must match the key type:
- Paper keys (`PK...` prefix) → `ALPACA_PAPER_TRADE=true`
- Live keys (`AK...` prefix) → `ALPACA_PAPER_TRADE=false`

If it doesn't match, the MCP will 401 on every account-level call.

### 6. Verify

```bash
claude mcp list
```

Expected: `alpaca: ... ✓ Connected`. If it shows ✗ Failed, run the diagnostic in `docs/alpaca-setup.md`.

Then **restart Claude Code** (close and re-open) so this session picks up the new MCP. After restart, in any project, `/mcp` should show alpaca connected.

### 7. Test by calling Alpaca

Once Claude Code is restarted and the MCP is loaded, call `mcp__alpaca__get_account_info` (read-only) and show the user:
- Their account number (last 4 digits only)
- Their equity / buying power
- Their cash balance

This confirms the keys work and the connection is live.

### 8. Print "you're set up"

Tell the user:

> ✅ **Setup complete.** Your Alpaca MCP is connected and verified.
>
> Next steps:
> 1. Open one of the example strategies in `./strategies/`. Read it.
> 2. If you want to use it as-is, change `status: paused` to `status: active` in the frontmatter, and adjust `capital_monthly_usd` to your real number.
> 3. Or run `/investment:new-strategy` to build a custom one interactively.
> 4. Once you have at least one `status: active` strategy, run `/investment:daily` to start the loop.
>
> Read `docs/getting-started.md` for the full walkthrough and `docs/safety-and-limits.md` for what this tool will and won't do.

## Hard rules

- **Never put API keys into any file in this workspace** — not in `.env`, not in strategy files, not in journal entries. Keychain (or your OS equivalent) is the only sanctioned storage.
- **Never execute trades during setup.** This wizard is read-only on Alpaca; the only `mcp__alpaca__*` call to make is `get_account_info` for verification.
- **If the user pastes a key into the chat by accident**, tell them to:
  1. Regenerate the key in the Alpaca dashboard (invalidates the leaked one).
  2. Re-run the Keychain storage step with the new key.
  3. Re-run `claude mcp add` with the refreshed values.
