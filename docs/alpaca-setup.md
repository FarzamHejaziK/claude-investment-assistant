# Alpaca setup

This doc covers: opening an Alpaca account, generating API keys, storing them securely, wiring up the MCP connection, and switching between paper and live.

For first-time setup, run `/setup` in Claude Code instead — it walks you through this interactively. Use this doc as reference if something goes wrong, or to refresh your memory later.

## Paper vs. live — which to pick

| | Paper | Live |
|---|---|---|
| **Money** | Fake ($100K simulated balance by default) | Real money you deposit |
| **Risk** | Zero — you can't lose anything | Real — you can lose what you deposit |
| **API endpoint** | `https://paper-api.alpaca.markets` | `https://api.alpaca.markets` |
| **API key prefix** | `PK...` | `AK...` |
| **`ALPACA_PAPER_TRADE` env var** | `true` | `false` |
| **Use for** | Learning, testing strategies, validating rules | Running tested strategies with real capital |

**Always start with paper.** Run for at least one month, see how the strategies behave through real market conditions, then decide whether to flip to live.

## Opening an Alpaca account

1. Go to https://alpaca.markets/
2. Click **Sign Up**. You'll need to provide ID and answer some KYC questions (US residents).
3. For paper trading, you don't need to fund the account — paper accounts start with $100K simulated balance automatically.
4. For live trading, you need to fund the account via ACH transfer or wire after KYC approval (usually takes 1–2 business days).

## Generating API keys

API keys are how the MCP connects to Alpaca to read your portfolio. **Paper and live have separate key panels** — they don't share keys.

### For paper

1. Log in at https://app.alpaca.markets/
2. Make sure you're in **paper** mode (URL contains `/paper/`)
3. On the right sidebar, find **API Keys**
4. Click **Generate New Key**
5. **Copy both** the `Key ID` (starts with `PK`) and `Secret Key` (long random string) immediately — the secret is shown **only once**

### For live

Same flow but ensure you're in live mode (URL contains `/live/`). The key prefix will be `AK`.

### If you lose the secret

You can't recover it. Click **Generate New Key** again to create a fresh pair (which invalidates the old one). Then re-store and re-wire the MCP per the steps below.

## Storing keys securely

**Never put keys into files in this repo.** Not in `.env`, not in strategy files, not in journal entries. They live in your OS's secret store.

### macOS — Keychain

In Terminal, run these one at a time. The `read -s` flag means typing is silent — the value won't appear on screen and won't be saved to shell history.

```bash
read -s "k?Paste ALPACA_API_KEY: " && echo && \
  security add-generic-password -a "$USER" -s alpaca-api-key -w "$k" -U && unset k

read -s "s?Paste ALPACA_SECRET_KEY: " && echo && \
  security add-generic-password -a "$USER" -s alpaca-secret-key -w "$s" -U && unset s
```

The `-U` flag means "update if exists" — safe to re-run when rotating keys.

**Verify (without exposing full keys):**

```bash
echo "API key prefix: $(security find-generic-password -a "$USER" -s alpaca-api-key -w | cut -c1-2)"
echo "Secret length: $(security find-generic-password -a "$USER" -s alpaca-secret-key -w | wc -c)"
```

Should print `PK` (paper) or `AK` (live), and length ~44.

### Linux — `secret-tool` (libsecret)

```bash
secret-tool store --label="Alpaca API key" service alpaca-api-key
# (paste key when prompted; press Enter then Ctrl+D)
secret-tool store --label="Alpaca Secret" service alpaca-secret-key
```

Retrieve later with `secret-tool lookup service alpaca-api-key`.

### Windows — Credential Manager

In PowerShell:

```powershell
Install-Module -Name CredentialManager -Scope CurrentUser -Force
New-StoredCredential -Target "alpaca-api-key" -Username "$env:USERNAME" -Password (Read-Host -AsSecureString)
New-StoredCredential -Target "alpaca-secret-key" -Username "$env:USERNAME" -Password (Read-Host -AsSecureString)
```

### Fallback — `.env` file (gitignored)

If nothing else works, create `.env` in this workspace:

```
ALPACA_API_KEY=PKxxxxxxxxxxxxxxxxx
ALPACA_SECRET_KEY=your_long_secret_here
```

Then `source .env` before running `claude mcp add`. **The `.gitignore` in this repo excludes `.env`**, so you won't accidentally commit it.

## Installing `uv`

The Alpaca MCP server is a Python program that runs via `uvx` (part of `uv`).

```bash
# macOS (recommended)
brew install uv

# macOS / Linux (curl installer)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

After install, install a managed Python — system Python is often broken on Apple Silicon:

```bash
uv python install 3.12
```

## Wiring up the MCP

This is the command that tells Claude Code where to find your Alpaca connection. Adjust based on paper vs. live.

### macOS (paper) — keys from Keychain

```bash
claude mcp add alpaca \
  --scope user \
  --transport stdio \
  --env ALPACA_API_KEY="$(security find-generic-password -a "$USER" -s alpaca-api-key -w)" \
  --env ALPACA_SECRET_KEY="$(security find-generic-password -a "$USER" -s alpaca-secret-key -w)" \
  --env ALPACA_PAPER_TRADE=true \
  -- uvx --python 3.12 alpaca-mcp-server
```

### macOS (live) — keys from Keychain

Same as paper, but set `ALPACA_PAPER_TRADE=false`:

```bash
claude mcp add alpaca \
  --scope user \
  --transport stdio \
  --env ALPACA_API_KEY="$(security find-generic-password -a "$USER" -s alpaca-api-key -w)" \
  --env ALPACA_SECRET_KEY="$(security find-generic-password -a "$USER" -s alpaca-secret-key -w)" \
  --env ALPACA_PAPER_TRADE=false \
  -- uvx --python 3.12 alpaca-mcp-server
```

### Verify

```bash
claude mcp list
```

Expected: `alpaca: ... ✓ Connected`.

**After wiring**, restart Claude Code so the current session picks up the new MCP. After restart, in any project: `/mcp` should show alpaca connected, and the `mcp__alpaca__*` tools should be available.

## Switching paper → live (or vice versa)

When you're ready to switch, three things happen:

1. **Generate new keys in the OTHER panel** (paper if you were on live, vice versa).
2. **Update Keychain** with the new values (re-run the `read -s` commands; the `-U` flag overwrites).
3. **Re-add the MCP** with the new `ALPACA_PAPER_TRADE` value:
   ```bash
   claude mcp remove alpaca --scope user
   claude mcp add alpaca ... --env ALPACA_PAPER_TRADE=<true|false> ...
   ```
4. **Restart Claude Code.**

**Critical:** the `ALPACA_PAPER_TRADE` value must match the key prefix. Paper keys (`PK`) → `true`. Live keys (`AK`) → `false`. Mismatch = 401 errors on every account-level call.

## Troubleshooting

**`claude mcp list` shows alpaca ✗ Failed**

Run the MCP command directly to see the actual error:
```bash
uvx --python 3.12 alpaca-mcp-server --help
```
If it fails with "Failed to inspect Python interpreter," you have a broken system Python. The `--python 3.12` flag tells uvx to use uv's managed Python instead. Make sure `uv python install 3.12` succeeded.

**HTTP 401 on `get_account_info` but `get_clock` and `get_stock_bars` work**

Paper/live mismatch. Market data uses different auth than account API. Verify:
- Your key prefix (`security find-generic-password -a "$USER" -s alpaca-api-key -w | cut -c1-2`)
- Your `ALPACA_PAPER_TRADE` value (check `claude mcp get alpaca` or re-add)

Match them. PK + true, or AK + false.

**MCP shows connected but tools aren't available in Claude Code session**

Restart Claude Code. MCPs are loaded at session start; mid-session changes don't always propagate.

**Forgot which keys I have**

```bash
security find-generic-password -a "$USER" -s alpaca-api-key -w | cut -c1-2
# PK = paper, AK = live, anything else = wrong
```
