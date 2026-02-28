# Basis Timpi

**NTMPI Cost Basis Calculator** — A static web tool for calculating your USD cost basis on the [Neutaro](https://neutaro.com) blockchain.

Part of the [ClawPurse](https://github.com/mhue-ai/ClawPurse) ecosystem.

## Features

- **Neutaro chain native** — Queries `api2.neutaro.io` Cosmos REST API directly (no API key needed)
- **Bundled historical prices** — 703 daily NTMPI/USD prices from 2024-03-28 to 2026-02-28 in `prices.js` (instant lookup, no API calls)
- **Manual override** — Edit any per-token cost for transactions where you have a different acquisition price
- **Three cost basis methods** — FIFO, LIFO, and Average Cost
- **Real-time stats** — Balance, cost basis, current value, and unrealized P&L
- **CSV export** — Download full transfer history with cost data
- **Deep-linkable** — Share via `?address=neutaro1...` URL parameter
- **Zero dependencies** — Static HTML + JS, no build step, no backend

## Files

| File | Purpose |
|------|---------|
| `index.html` | The complete application (single-file HTML/CSS/JS) |
| `prices.js` | Static NTMPI/USD daily prices (2024-03-28 → 2026-02-28, 703 entries, ~16KB) |
| `README.md` | This file |

## How It Works

1. Enter your `neutaro1...` wallet address
2. The app fetches your balance and all bank transfer history from the Neutaro chain
3. For each incoming transfer, it looks up the NTMPI/USD price on that date from the bundled `prices.js`
4. You can manually enter or override the cost per token for any transfer
5. Select your accounting method (FIFO, LIFO, or Average Cost)
6. View your total cost basis, current value, and unrealized gain/loss

## Price Data Sources

| Date Range | Source | Notes |
|------------|--------|-------|
| 2024-03-28 → 2025-02-28 | BitMart OHLCV (daily close) | Earliest available exchange data |
| 2025-03-01 → 2026-02-28 | CoinGecko (aggregated daily) | More accurate multi-exchange average |
| Current price | CoinGecko live API | Fetched on page load |

### Updating prices.js

The static price file can be regenerated to include newer data. Fetch from CoinGecko:

```bash
curl -s "https://api.coingecko.com/api/v3/coins/neutaro/market_chart?vs_currency=usd&days=365"
```

Then merge with existing `prices.js` entries. CoinGecko free tier limits historical data to 365 days.

## Deploy to GitHub Pages

1. Add `index.html`, `prices.js`, and `README.md` to the repo root
2. Go to **Settings → Pages → Source → Deploy from a branch → main → / (root)**
3. Your site will be live at `https://mhue-ai.github.io/basis-timpi/`

## Technical Details

- **Chain:** Neutaro-1 (Cosmos SDK)
- **RPC:** `https://rpc2.neutaro.io`
- **REST/LCD:** `https://api2.neutaro.io`
- **Native denom:** `uneutaro` (6 decimals)
- **Display denom:** NTMPI
- **Address prefix:** `neutaro`
- **Explorer:** [explorer.neutaro.io](https://explorer.neutaro.io/Neutaro)

These endpoints match the [ClawPurse](https://github.com/mhue-ai/ClawPurse) wallet configuration.

## Limitations

- Only tracks `MsgSend` bank transfers of `uneutaro`. Staking rewards, IBC transfers, and DEX swaps are not included.
- Historical prices before 2024-03-28 are not available (token listing date on exchanges).
- This is a read-only tool — it never touches your private keys.

## Disclaimer

This tool is for informational purposes only and is not financial or tax advice. Always verify data independently and consult a tax professional for your specific situation.

## License

MIT
