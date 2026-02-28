# Basis Timpi

**NTMPI Cost Basis Calculator** — A static web tool for calculating your USD cost basis on the [Neutaro](https://neutaro.com) blockchain.

Part of the [ClawPurse](https://github.com/mhue-ai/ClawPurse) ecosystem.

## Features

- **Neutaro chain native** — Queries `api2.neutaro.io` Cosmos REST API directly (no API key needed)
- **Auto-fetch historical prices** — Pulls USD price at time of each incoming transfer via CoinGecko
- **Manual override** — Edit any per-token cost for transactions where auto-price isn't available
- **Three cost basis methods** — FIFO, LIFO, and Average Cost
- **Real-time stats** — Balance, cost basis, current value, and unrealized P&L
- **CSV export** — Download full transfer history with cost data
- **Deep-linkable** — Share via `?address=neutaro1...` URL parameter
- **Zero dependencies** — Single HTML file, no build step, no backend

## How It Works

1. Enter your `neutaro1...` wallet address
2. The app fetches your balance and all bank transfer history from the Neutaro chain
3. For incoming transfers within the last 365 days, it auto-fetches the NTMPI/USD price on that date from CoinGecko
4. You can manually enter or override the cost per token for any transfer
5. Select your accounting method (FIFO, LIFO, or Average Cost)
6. View your total cost basis, current value, and unrealized gain/loss

## Data Sources

| Data | Source | API Key? |
|------|--------|----------|
| Balance & transfers | [Neutaro REST API](https://api2.neutaro.io) | No |
| Current NTMPI price | [CoinGecko](https://www.coingecko.com/en/coins/neutaro) | No |
| Historical prices | CoinGecko `/history` endpoint | No (365-day limit on free tier) |

## Deploy to GitHub Pages

1. Add `index.html` (and optionally this `README.md`) to the repo root
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

- CoinGecko free API only provides historical prices for the last 365 days. Older transfers require manual price entry.
- CoinGecko rate limits (~10-30 requests/minute). The app pauses automatically between batches.
- Only tracks `MsgSend` bank transfers of `uneutaro`. Staking rewards, IBC transfers, and DEX swaps are not included.
- This is a read-only tool — it never touches your private keys.

## Disclaimer

This tool is for informational purposes only and is not financial or tax advice. Always verify data independently and consult a tax professional for your specific situation.

## License

MIT
