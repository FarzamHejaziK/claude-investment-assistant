---
description: First-time setup wizard — wires up the Alpaca MCP, stores keys in the OS keyring, verifies the connection, and walks through configuring a monthly budget for each example strategy.
allowed-tools: Bash, Read, Write, Edit, Glob, WebFetch
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

### 8. Configure your strategies — per-strategy monthly budget

Now walk the user through each `.example.md` file in `./strategies/`. Process them in this order: **dip-buying → ai-value-chain → active-trading** (lowest-risk to highest-risk, which matches the introduction order). For each one:

#### 8a. Summarize the strategy

Read the strategy file's `## Profile` section plus any prominent warning callouts at the top. Print a 3–5 line summary for the user:
- What the strategy does (one plain-English sentence)
- The example file's current `capital_monthly_usd` default
- Any risk warning baked into the file (e.g., active-trading's "70–90% of retail active traders underperform" notice — surface this verbatim before asking the user about activating it)

#### 8b. Ask whether to use it, and at what budget

```
Want to use this strategy? Three options:
  1. Yes — and let me set my own monthly budget
  2. Yes — use the example default of $<value>/month
  3. No / skip for now
```

If **option 1**:
- Ask: "What's your monthly budget for this strategy (USD, positive integer)?"
- Validate: must be > 0. If unusually small (< $50) or unusually large for the strategy type, confirm with the user — don't silently accept a typo.
- For **active-trading specifically**: if budget > $500, re-state the underperformance warning and ask the user to confirm they accept the risk at that size.

If **option 2**: use the example file's existing `capital_monthly_usd` value as-is.

If **option 3**: leave the `.example.md` file untouched. Note "skipped" and move to the next strategy.

#### 8c. Ask whether to activate or keep paused

If the user picked option 1 or 2, ask:

```
Activate now (status: active) or keep paused (status: paused)?

  - Paused (recommended) — strategy is configured but won't run until you flip the switch. Read the file once, understand the rules, then activate.
  - Active — /investment:daily will start proposing orders for this strategy on the next run.
```

**Default to paused.** Only activate if the user explicitly chooses active.

**For active-trading specifically:** even if the user picks active, surface the warning one more time and confirm: "Active trading has high underperformance odds. Still activate?" If they hesitate or pick paused, that's the safer outcome.

#### 8d. Apply the changes

For each strategy the user chose to use (option 1 or 2):

1. Copy `./strategies/<name>.example.md` to `./strategies/<name>.md`. **Leave the `.example.md` file untouched** — it stays as a reference template in the repo.
2. In the new `<name>.md`, edit the YAML frontmatter:
   - Set `capital_monthly_usd:` to the user's chosen value
   - Set `last_updated:` to today's date
   - Set `version:` to `1.0` (this is the user's first version of their strategy)
   - If the user chose "activate now," set `status: active`
   - If the user chose "keep paused," leave `status: paused`
   - If the user picked live mode in step 1, set `account: alpaca-live`. If paper, leave as `alpaca-paper`.

#### 8e. Summary

After all 3 strategies have been processed, print a recap:

```
Strategies configured:
  ✓ dip-buying — $X/month, <status>
  ✓ ai-value-chain — $X/month, <status>
  ✗ active-trading — skipped
  ...
Active: N | Paused: M | Skipped: K
```

If at least one strategy is active, the user is ready to run `/investment:daily`. If everything is paused, the user needs to flip a status to `active` first (open the file, edit, save).

### 9. Print "you're set up"

Tell the user:

> ✅ **Setup complete.** Your Alpaca MCP is connected and verified.
>
> **Your strategies:** [summary from step 8e]
>
> **Next steps:**
> 1. If any strategies are `status: paused`, open them and change to `status: active` when ready.
> 2. Run `/investment:daily` each market morning to get today's proposals.
> 3. Run `/investment:new-strategy` anytime to build additional strategies.
> 4. Run `/investment:help` if you have questions about anything.
>
> Read `docs/getting-started.md` for the full walkthrough and `docs/safety-and-limits.md` for what this tool will and won't do.

## Hard rules

- **Never put API keys into any file in this workspace** — not in `.env`, not in strategy files, not in journal entries. Keychain (or your OS equivalent) is the only sanctioned storage.
- **Never execute trades during setup.** This wizard is read-only on Alpaca; the only `mcp__alpaca__*` call to make is `get_account_info` for verification.
- **If the user pastes a key into the chat by accident**, tell them to:
  1. Regenerate the key in the Alpaca dashboard (invalidates the leaked one).
  2. Re-run the Keychain storage step with the new key.
  3. Re-run `claude mcp add` with the refreshed values.
