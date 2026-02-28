# SYSTEM.md — Basis Timpi Internal Development Reference

> Internal documentation for maintainers and AI assistants working on this project.

---

## Quick Reference

| Item | Value |
|------|-------|
| **Live URL** | https://basis.clawpurse.ai |
| **Repo** | https://github.com/mhue-ai/basis-timpi |
| **Hosting** | GitHub Pages (main branch root) → Cloudflare |
| **Stack** | Single-file HTML/CSS/JS, zero dependencies |
| **Lines** | ~1,500 (index.html) |

---

## Repository Structure

```
basis-timpi/
├── index.html      # Complete app (HTML + embedded CSS + JS)
├── prices.js       # Static NTMPI/USD daily prices, loaded via <script>
├── CNAME           # basis.clawpurse.ai
├── LICENSE         # MIT
├── README.md       # Public documentation
└── SYSTEM.md       # This file — internal dev reference
```

---

## Key Code Sections (index.html)

The file is organized in this order:

### 1. CSS Styles (lines ~1-340)
- CSS custom properties (dark theme with coral/cyan/yellow accents)
- Stats card grid, NFT grid, transaction table styles
- Filter chip styles, responsive breakpoints
- Custom class: `.td-node`, `.td-apy`, `.td-lootbox`, etc. for type labels

### 2. HTML Structure (lines ~340-660)
- Wallet address input + action buttons
- Five stats cards: Available Balance, Staked, Cost Basis, Total Value, Rewards Earned
- Unrealized P&L card
- NFT section: summary stats → NFT grid → reward reference tables → disclaimer
- Transaction history: filter chips → cost basis method tabs → sortable table → CSV export

### 3. JavaScript (lines ~660-end)

#### Configuration Block (~660-680)
```javascript
const NEUTARO = {
  rest: 'https://api2.neutaro.io',
  rpc: 'https://rpc2.neutaro.io',
  denom: 'uneutaro',
  decimals: 6,
  nftContract: 'neutaro14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9s9z4e5z',
};
```

#### NFT Reward Schedules (~682-750)
`NFT_REWARDS` object — keys are `'Collector'`, `'Collector FE'`, `'Guardian'`, `'Guardian FE'`, etc.
Each key maps to an array of `{ from, to, ntmpi }` period objects.

#### NFT Classification (~752-770)
`classifyNFT(tokenId)` — Parses token name to determine:
- Category: Collector, Guardian, Synaptron, GeoCore
- Edition: "Founders Edition" or "Regular" (checks for "founders edition" in name)
- Returns: `{ category, edition, rewardKey, icon, css }`

#### State Management (~770-790)
```javascript
const state = {
  address: '', transfers: [], nfts: [],
  currentPrice: 0, costBasisMethod: 'fifo',
  activeFilters: new Set([...all types...]),
};
```

#### Transaction Fetching (~800-960)
- `fetchAllTransactions(address)` — Orchestrator, fetches all tx types
- `fetchTxByEvent(eventKey, eventValue, filterAddress)` — Generic paginated Cosmos tx query
- Captures memo from `tx.body.memo` for classification

#### Transaction Classification (~824-895)
Four known Timpi addresses:
```javascript
const TIMPI_NODE_PAYER  = 'neutaro1q68nu5lwczjepjtt6v7fek8vu2m7jxnaukrpdu';
const TIMPI_APY_PAYER   = 'neutaro1n0uc8wwaxez3xzw8722kh2c9kuvqxtamgja4rg';
const TIMPI_OPS         = 'neutaro1dq5a4x9f8ksp0y062z7v5cn3sa99tgfmjrk92x';
const CHARITY_VALIDATOR = 'neutaro123rsx3yhd8qts0aduw7zacpwtk6m94ucvya354';
```

Classification logic: `sender address` → narrow by `memo.toLowerCase()` content.

#### NFT Fetching (~960-1000)
`fetchNFTs(address)` — Paginated CW721 `tokens` query, 30 per page.

#### Rendering (~1000-1400)
- `renderStats()` — Updates stat cards
- `renderNFTs()` — NFT grid + summary totals
- `renderTable()` — Transaction table with filtering, sorting, memo column
- `TYPE_LABELS` — Maps tx type → HTML label with icon + color
- `FILTER_GROUPS` — Maps chip filter → array of tx types

#### Cost Basis Engine (~1150-1250)
- FIFO: First-in-first-out lot matching
- LIFO: Last-in-first-out lot matching
- Average Cost: Running weighted average
- All operate on the `transfers` array, assigning `costPerToken` and `totalCost`

#### CSV Export (~1450-1480)
Exports: Date, Type, Amount, Counterparty, Cost/NTMPI, Total Cost, Memo, TX Hash

---

## Known Timpi Payout Patterns

Based on on-chain analysis of actual transactions:

### Node Reward Payer (`neutaro1q68nu...ukrpdu`)
- Pays **per node, per type** monthly
- Memo format: `"{Month}{Year} {NodeType}"` — e.g. `"Jan2026 Synap"`, `"Dec2025 Guardian"`
- Variations: `"July Synap"`, `"Sept2025 GeoCore"`, `"Aug2025 Collector"`
- Also: `"Sept Missed Rewards"`, `"Apr Missing Rewards"`
- One transaction per node (so 5 Synaptron FE nodes = 5 separate txs with same memo)

### Timpi APY Payer (`neutaro1n0uc8...gja4rg`)
- Pays monthly staking APY
- Memo format: `"Timpi APY{Month}{Year}"` — e.g. `"Timpi APYDec2025"`, `"Timpi APYSept2025"`
- Also: `"Lootbox Reward"`, `"Lootbox rewards"`, `"Timpi Synaptron Adjustment"`, `"APY Rewards November"`
- Single transaction per month for APY

### Timpi Operations (`neutaro1dq5a4...jrk92x`)
- Large bulk transfers
- Memo: `"TimpiTransfer"`, `"SynaptronDec"`, `"SynaptronJan"`, `"Movement"`, or empty
- Often 5-figure or 6-figure amounts

### Charity Validator (`neutaro123rsx...cvya354`)
- Gift giveaways
- Memo: `"Gift Milestone 4 Giveaway from Charity Validator"`, `"Gift X Charity Validator Moo"`

### Other Observed Senders
- Personal wallets with memos like `"Test Out"`, `"Closing out this wallet"`
- Drip faucet: memo `"Timpi Drip - trusted tier"`
- Small random transfers (faucet tests, 0.01-9 NTMPI)

---

## Actual vs Published Reward Rates

**Important:** Published reward rates are pool contributions per node sold, not guaranteed payouts. Actual per-node payouts vary based on:
- How many total nodes of that type have been sold
- How many are currently active/online
- Pool distribution mechanics

Observed actual per-node monthly payouts (as of Feb 2026):

| Node Type | Published Rate | Actual Range | Notes |
|-----------|---------------|--------------|-------|
| Collector (Regular) | 250 (Sep 25+) | 496-774 | Often higher than published |
| Guardian FE | 750 (Sep 25+) | 1,072-1,194 | Consistently ~50% above published |
| Synaptron T1 FE | 1,200 (Jan 26+) | 1,633-1,793 | ~35% above published |
| GeoCore FE | 1,200 (pre Aug 26) | 800-889 | Below published rate |

---

## Deployment Notes

- **Push to `main`** → GitHub Pages auto-deploys (usually 1-2 minutes)
- **Cloudflare cache** — May serve stale version for up to 5 min. Hard refresh (Ctrl+Shift+R) bypasses browser cache but Cloudflare edge cache may persist.
- **CNAME file** must remain as `basis.clawpurse.ai`
- **No build step** — The HTML file IS the app

### GitHub API Deployment (used in dev sessions)

```bash
# Get current file SHA
SHA=$(curl -s -H "Authorization: token $PAT" \
  "https://api.github.com/repos/mhue-ai/basis-timpi/contents/index.html" \
  | jq -r '.sha')

# Base64 encode and push
CONTENT=$(base64 -w 0 index.html)
curl -X PUT -H "Authorization: token $PAT" \
  "https://api.github.com/repos/mhue-ai/basis-timpi/contents/index.html" \
  -d "{\"message\":\"commit message\",\"content\":\"$CONTENT\",\"sha\":\"$SHA\"}"
```

---

## Future Improvements

- [ ] Synaptron Tier 2 detection (requires VRAM metadata not on-chain)
- [ ] Historical reward tracking — compare actual payouts against published rates over time
- [ ] IBC transfer tracking
- [ ] Multi-address support
- [ ] Dark/light theme toggle
- [ ] Staking validator details display
- [ ] Price chart visualization
- [ ] Tax year summary reports

---

## Related Projects

| Project | URL | Description |
|---------|-----|-------------|
| ClawPurse | [clawpurse.ai](https://clawpurse.ai) | Main wallet platform |
| ClawPurse Gateway | [gateway.clawpurse.ai](https://gateway.clawpurse.ai) | HTTP 402 micropayment gateway |
| Drip Faucet | [drip.clawpurse.ai](https://drip.clawpurse.ai) | NTMPI token faucet |
| Neutaro Explorer | [explorer.neutaro.io](https://explorer.neutaro.io) | Chain explorer (source of truth) |
