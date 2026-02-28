# Basis Timpi

**NTMPI Cost Basis Calculator** — A zero-dependency, static web tool for tracking transactions, calculating USD cost basis, and monitoring node NFT rewards on the [Neutaro](https://neutaro.com) blockchain.

🌐 **Live:** [basis.clawpurse.ai](https://basis.clawpurse.ai)  
📦 Part of the [ClawPurse](https://github.com/mhue-ai/ClawPurse) ecosystem.

---

## Features

### Cost Basis Tracking
- **Three accounting methods** — FIFO, LIFO, and Average Cost with instant switching
- **Bundled historical prices** — 703+ daily NTMPI/USD prices from 2024-03-28 onward (no API calls for lookups)
- **Manual override** — Edit the per-token cost on any transaction row
- **Real-time stats** — Available balance, staked balance, cost basis, current value, unrealized P&L

### Transaction Classification
- **Smart memo parsing** — Incoming transactions are classified by sender address + memo content
- **Known Timpi address recognition** — Four known payout addresses are mapped to specific categories
- **18 transaction types** with color-coded labels and emoji icons
- **Filter chips** — Toggle visibility by category group (Transfers, Node Rewards, Staking APY, Delegation, Rewards, Lootbox/Adjustments)
- **Memo column** — Raw transaction memo displayed in the table for transparency

### Node Access NFTs
- **On-chain CW721 query** — Reads NFTs owned by the wallet from Neutaro's CosmWasm NFT contract
- **Founders Edition detection** — Parses token names for "Founders Edition" to apply correct reward schedules
- **Estimated monthly rewards** — Per-node and total estimated NTMPI output based on published reward schedules
- **Full reward reference table** — Displays all node types (Regular + Founders) across all timeframes
- **USD value estimates** — Monthly and annual reward projections at current NTMPI price

### Other
- **CSV export** — Download full transaction history with cost basis data
- **Deep-linkable** — Share via `?address=neutaro1...` URL parameter
- **Zero dependencies** — Single-file HTML/CSS/JS, no build step, no backend, no framework
- **Responsive** — Works on desktop and mobile

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | Complete application — HTML, CSS, and all JavaScript in a single file (~1,500 lines) |
| `prices.js` | Static NTMPI/USD daily price history (2024-03-28 → present, ~16 KB) |
| `CNAME` | GitHub Pages custom domain → `basis.clawpurse.ai` |
| `LICENSE` | MIT License |
| `README.md` | This file |

---

## How It Works

1. Enter a `neutaro1...` wallet address (or use the URL parameter)
2. The app fetches from the Neutaro Cosmos REST API:
   - Bank balance (`/cosmos/bank/v1beta1/balances/`)
   - Staking delegations (`/cosmos/staking/v1beta1/delegations/`)
   - Transfer history (incoming + outgoing `MsgSend`)
   - Delegation, undelegation, and redelegation transactions
   - Staking reward withdrawal transactions (`MsgWithdrawDelegatorReward`)
   - CW721 NFT ownership (`/cosmwasm/wasm/v1/contract/.../smart/`)
3. Each incoming transfer is matched against bundled `prices.js` for that date's NTMPI/USD price
4. Transactions are classified by sender address and memo content (see below)
5. Select your accounting method (FIFO, LIFO, or Average Cost)
6. View cost basis, current value, and unrealized gain/loss

---

## Transaction Classification System

Incoming transfers are classified using a two-layer system: **sender address** + **memo content**.

### Known Sender Addresses

| Address | Label | Description |
|---------|-------|-------------|
| `neutaro1q68nu...ukrpdu` | Node Reward Payer | Monthly node reward payouts (per-node, per-type) |
| `neutaro1n0uc8...gja4rg` | Timpi APY / Lootbox | Staking APY distributions, lootbox rewards, synaptron adjustments |
| `neutaro1dq5a4...jrk92x` | Timpi Operations | Bulk transfers, TimpiTransfer, operational payouts |
| `neutaro123rsx...cvya354` | Charity Validator | Gift giveaways |

### Memo-Based Classification

When a transaction comes from a known sender, the memo is parsed to determine the specific type:

| Memo Contains | Classification | Display Label |
|---------------|---------------|---------------|
| `synap` | `node_synaptron` | ⚡ NODE: SYNAPTRON |
| `guardian` | `node_guardian` | 🛡️ NODE: GUARDIAN |
| `collector` | `node_collector` | 🔍 NODE: COLLECTOR |
| `geo` | `node_geocore` | 🌐 NODE: GEOCORE |
| `apy` / `apr` | `staking_apy` | 💰 STAKING APY |
| `missing` / `missed` | `node_adjustment` | 🔧 ADJUSTMENT |
| `lootbox` | `lootbox` | 🎁 LOOTBOX |
| `drip` / `faucet` | `drip` | 💧 DRIP |
| (charity validator) | `gift` | 🎉 GIFT |
| (Timpi ops, no match) | `timpi_transfer` | ⇣ TIMPI XFER |

### All Transaction Types

| Type | Label | Direction | Description |
|------|-------|-----------|-------------|
| `in` | ▼ IN | Inflow | Generic incoming transfer |
| `out` | ▲ OUT | Outflow | Outgoing transfer |
| `delegate` | ⬆ DELEGATE | — | Stake delegation |
| `undelegate` | ⬇ UNDELEGATE | — | Stake undelegation |
| `redelegate` | ↔ REDELEGATE | — | Validator redelegate |
| `reward` | ★ REWARD | Inflow | On-chain staking reward withdrawal |
| `node_synaptron` | ⚡ NODE: SYNAPTRON | Inflow | Synaptron node reward |
| `node_guardian` | 🛡️ NODE: GUARDIAN | Inflow | Guardian node reward |
| `node_collector` | 🔍 NODE: COLLECTOR | Inflow | Collector node reward |
| `node_geocore` | 🌐 NODE: GEOCORE | Inflow | GeoCore node reward |
| `node_reward` | 📦 NODE REWARD | Inflow | Unclassified node reward |
| `node_adjustment` | 🔧 ADJUSTMENT | Inflow | Missed/retroactive reward |
| `staking_apy` | 💰 STAKING APY | Inflow | Timpi staking APY distribution |
| `lootbox` | 🎁 LOOTBOX | Inflow | Lootbox reward |
| `timpi_transfer` | ⇣ TIMPI XFER | Inflow | Bulk Timpi operational transfer |
| `gift` | 🎉 GIFT | Inflow | Charity validator gift |
| `drip` | 💧 DRIP | Inflow | Faucet / drip payout |

### Filter Groups (UI Chips)

| Chip Label | Includes Types |
|------------|---------------|
| Transfers In | `in`, `timpi_transfer`, `gift`, `drip` |
| Transfers Out | `out` |
| Node Rewards | `node_synaptron`, `node_guardian`, `node_collector`, `node_geocore`, `node_reward` |
| Staking APY | `staking_apy` |
| Delegate | `delegate` |
| Undelegate | `undelegate` |
| Redelegate | `redelegate` |
| Rewards | `reward` (on-chain staking) |
| Other Rewards | `lootbox`, `node_adjustment` |

---

## Node NFT Reward Schedules

Rewards are **guaranteed minimum pool contributions per node sold** shared among all active nodes. Actual payouts depend on node uptime and total active nodes, so they fluctuate month to month.

Founders Edition (FE) rates are set fees as published by Timpi.

### Collector & Guardian Nodes

| Node Type | Edition | Till Aug 2025 | Sep 2025 – Aug 2026 | Sep 2026 – Aug 2027 |
|-----------|---------|--------------|---------------------|---------------------|
| Collector | Regular | 375 | 250 | 210 |
| Collector | Founders | 563 | 375 | 315 |
| Guardian | Regular | 667 | 500 | 375 |
| Guardian | Founders | 1,000 | 750 | 563 |

### Synaptron Nodes

| Node Type | Edition | Till Dec 2025 | Jan – Dec 2026 | Jan – Dec 2027 |
|-----------|---------|--------------|----------------|----------------|
| Synaptron T1 (4-6GB) | Regular | 1,400 | 800 | 500 |
| Synaptron T1 (4-6GB) | Founders | 2,100 | 1,200 | 750 |
| Synaptron T2 (8-16GB) | Regular | 1,600 | 1,000 | 700 |
| Synaptron T2 (8-16GB) | Founders | 2,400 | 1,500 | 1,050 |

### GeoCore Nodes

| Node Type | Edition | Till Aug 2026 | Sep 2026 – Aug 2027 | Sep 2027 – Aug 2028 |
|-----------|---------|--------------|---------------------|---------------------|
| GeoCore | Regular | 800 | 400 | 300 |
| GeoCore | Founders | 1,200 | 600 | 450 |

> **Sources:** [Timpi Official Nodes Repo](https://github.com/Timpi-official/Nodes) · [Timpi Knowledge Base](https://timpi.gitbook.io/timpis-knowledge-base/timpis-knowledge-base/timpi-nodes/node-reward-structures)

---

## Price Data Sources

| Date Range | Source | Notes |
|------------|--------|-------|
| 2024-03-28 → 2025-02-28 | BitMart OHLCV (daily close) | Earliest available exchange data |
| 2025-03-01 → present | CoinGecko (aggregated daily) | Multi-exchange average |
| Current price | CoinGecko live API | Fetched on page load |

### Updating `prices.js`

The static price file can be regenerated to include newer data:

```bash
curl -s "https://api.coingecko.com/api/v3/coins/neutaro/market_chart?vs_currency=usd&days=365"
```

Merge new entries with existing `prices.js`. CoinGecko free tier limits historical data to 365 days.

---

## Technical Details

| Property | Value |
|----------|-------|
| **Chain** | Neutaro-1 (Cosmos SDK) |
| **REST/LCD** | `https://api2.neutaro.io` |
| **RPC** | `https://rpc2.neutaro.io` |
| **Native denom** | `uneutaro` (6 decimals) |
| **Display denom** | NTMPI |
| **Address prefix** | `neutaro` |
| **NFT contract** | `neutaro14hj2tavq8fpesdwxxcu44rty3hh90vhujrvcmstl4zr3txmfvw9s9z4e5z` (CW721) |
| **Explorer** | [explorer.neutaro.io](https://explorer.neutaro.io/Neutaro) |
| **Hosting** | GitHub Pages behind Cloudflare |
| **Custom domain** | `basis.clawpurse.ai` (CNAME) |

---

## Deployment

This is a static site hosted on GitHub Pages:

1. All files are in the repo root (`index.html`, `prices.js`, `CNAME`)
2. GitHub Pages deploys from the `main` branch root
3. Cloudflare DNS points `basis.clawpurse.ai` to GitHub Pages
4. No build step required — push to `main` and it deploys

---

## Architecture

```
basis.clawpurse.ai
├── index.html          ← Single-file app (HTML + CSS + JS)
│   ├── Wallet input + URL params
│   ├── Stats cards (balance, staked, cost basis, P&L)
│   ├── NFT section (CW721 query, reward estimates, reference table)
│   ├── Transaction table (classified, filterable, sortable)
│   ├── Cost basis engine (FIFO / LIFO / Avg Cost)
│   └── CSV export
├── prices.js           ← Static daily NTMPI/USD prices
├── CNAME               ← Custom domain
└── README.md           ← Documentation
```

### Data Flow

```
User enters address
       │
       ▼
┌──────────────────┐
│  Neutaro REST API │
│  api2.neutaro.io  │
├──────────────────┤
│ Bank balances     │
│ Staking delegates │
│ Transfer txs      │──► Classify by sender + memo
│ Staking txs       │
│ Reward txs        │
│ CW721 NFT query   │──► Founders Edition detection
└──────────────────┘
       │
       ▼
┌──────────────────┐
│  prices.js        │──► Match date → NTMPI/USD price
└──────────────────┘
       │
       ▼
┌──────────────────┐
│  Cost Basis Calc  │──► FIFO / LIFO / Avg Cost
└──────────────────┘
       │
       ▼
  Dashboard + CSV Export
```

---

## Limitations

- **Pool-based rewards** — The NFT reward table shows guaranteed minimum pool contributions per node sold. Actual payouts vary based on active node count and uptime.
- **No IBC or DEX** — Only tracks native `uneutaro` MsgSend transfers and staking operations. IBC transfers and DEX swaps are not included.
- **Historical prices** — Data starts at 2024-03-28 (token listing date). Earlier transactions won't have price data.
- **Read-only** — This tool never touches private keys or submits transactions.
- **Synaptron tier detection** — Currently classifies all Synaptrons as Tier 1. Tier 2 (8-16GB VRAM) detection would require additional metadata not available on-chain.

---

## Disclaimer

This tool is for informational purposes only and is not financial or tax advice. Always verify data independently and consult a tax professional for your specific situation.

## License

MIT — Copyright (c) 2026 Mhue AI
