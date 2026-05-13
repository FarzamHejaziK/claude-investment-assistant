---
name: ai-value-chain
status: paused
account: alpaca-paper
capital_monthly_usd: 500
horizon: long-term (5+ years)
last_updated: 2026-05-13
version: 1.0
auto_research: daily
---

# Strategy: AI Value Chain — Picks-and-Shovels (example)

Long-horizon basket targeting layers of the AI value chain **beyond** consensus mega-cap names (NVDA, MSFT, GOOGL, META, AMZN). Picks-and-shovels providers in power, cooling, advanced packaging, connectivity, specialty materials where AI capex is real but multiples haven't fully priced it in.

> **This is a paused example.** To activate: change `status: paused` to `status: active`, adjust `capital_monthly_usd`, customize the watchlist if you want different names, and run `/daily`.
>
> **The watchlist below is research-based as of 2026-05-13.** Theses, names, and rankings will go stale. Re-validate before activating.

## Profile

- **Capital:** $500/month (equal-weight DCA = $62.50 / name / month)
- **Horizon:** 5+ years, buy-and-hold
- **Style:** Equal-weight monthly DCA across a curated watchlist. Mechanical accumulation; no dip-timing.
- **Why exclude the obvious mega-caps:** Implicit exposure already exists via any index fund (VTI/VOO/SPY/QQQ) the user already holds. This basket is for *differentiated* exposure.

## Watchlist (HIGH conviction — 8 names, active rotation)

| Ticker | Layer | One-line thesis |
|---|---|---|
| **GEV** | Power generation | Diversified gas turbines + grid + SMR |
| **TLN** | Nuclear | Direct hyperscaler PPAs; fleet-scale nuclear with co-location economics |
| **NVT** | Cooling | Liquid-cooling pure-play; less crowded than VRT |
| **CAMT** | Packaging metrology | HBM bump + hybrid-bond inspection |
| **ONTO** | Packaging metrology | Advanced-packaging inspection ramp |
| **FORM** | Wafer test | Sole-source MEMS probe cards for high-stack HBM |
| **CRDO** | Connectivity | AECs + silicon photonics |
| **ENTG** | Specialty materials | CMP slurries / filtration / gas purifiers |

Equal-weight allocation: each name receives **1/8 = 12.5%** of monthly contribution.

## Watchlist (MED conviction — monitor only)

These came out of initial research as medium-conviction. They enter active rotation only by **explicit user edit** of this file.

| Ticker | Layer | Why monitor |
|---|---|---|
| BWXT | Nuclear / SMR | Naval reactor monopoly |
| LEU | Nuclear fuel | Only US uranium enricher |
| POWL | Power / switchgear | Custom MV switchgear backlog |
| MOD | Cooling | DC sales growth, legacy auto drag |
| ALAB | Connectivity | PCIe/CXL retimers |
| VECO | Equipment niche | EUV mask blank monopoly |
| IRM | Data center REIT | DC growth at REIT discount |

## Buy strategy — equal-weight monthly DCA

On or near the 1st of each calendar month:

1. Confirm `capital_monthly_usd` is available (or SGOV reserve has been built up).
2. Allocate equally across the 8 HIGH-conviction watchlist names.
3. Place 8 fractional-share market orders (Alpaca supports fractional).
4. **No timing.** Buy regardless of dip / rally / sideways action.

If a name has a thesis-break signal in a given month, **skip that name** and redistribute its slice to the other 7. Never auto-remove from the watchlist; only the user does that.

## Sizing rules

- **Per-name max position:** 15% of total strategy AUM. If a name appreciates past the cap, redirect that month's allocation to underweight names; do not exceed.
- **No automatic rebalancing.** Annual review (Q1) decides if a manual rebalance is warranted.

## Risk rules

- **Buy-only, no leverage, no options, no margin.**
- **No instruments outside the HIGH-conviction watchlist.** MED names enter only by user edit.
- **Thesis-break override:** daily research surfaces thesis-breaking events; skip that month's buy for the affected name. The plugin will NOT auto-remove the name — the user decides.

## Daily research protocol (because `auto_research: daily`)

The `/daily` command will, for this strategy:

1. **Per-name news scan** — for each of the 8 HIGH names, web-search for material news in the last 24h (earnings, guidance, M&A, customer wins/losses, regulatory action, accounting events).
2. **Broader chain landscape:**
   - Hyperscaler capex announcements
   - HBM cycle news
   - Power grid / data center power
   - Advanced packaging
3. **Crowdedness drift check** — note if any HIGH name's profile is shifting toward consensus-mega-cap status.

Findings appended to "Live research notes" below (this is the only auto-edit exception).

## Live research notes

(Auto-appended by `/daily`. Each entry dated. After 30 entries, oldest move to `journal/<strategy-name>-research-archive.md`.)

### 2026-05-13 — initial seed

- Initial watchlist context: 5 of 8 are semi-related (CAMT, ONTO, FORM, ENTG, CRDO). 3 are power/cooling (GEV, TLN, NVT).
- ALAB and CRDO carry customer-concentration risk (hyperscaler / NVDA platform).
- CAMT, ONTO, FORM all correlated to HBM build-out cadence — if HBM oversupplies, these correlate down.
- No active thesis-break signals at activation.

## Open questions / things to revisit

- [ ] **MED-conviction additions:** threshold for adding a name to active rotation?
- [ ] **Annual rebalance (Q1):** review allocation drift after first 12 months.
- [ ] **Sector concentration:** consider rebalancing toward power/cooling/REIT if semis dominate.
- [ ] **Max position cap (15%):** may need adjusting if a name materially outperforms.

## Change log

- **2026-05-13 — v1.0.** Initial example strategy.
