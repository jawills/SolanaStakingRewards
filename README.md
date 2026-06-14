# Solana Staking Rewards to Koinly CSV

A web-based application that fetches Solana staking rewards and exports them in Koinly-compatible CSV format for tax reporting.

## Features

- **Smart Epoch Processing**: Finds all stake accounts for your wallet and scans epochs for inflation rewards
- **Full History by Default**: Scans from the current epoch back to epoch 150 (when mainnet inflation rewards began)
- **Local Database Storage**: Uses IndexedDB to store results locally, building complete history incrementally across sessions
- **Smart Retry System**: Categorizes failures (rate limits, network, RPC errors) and retries only what is needed
- **Manual Epoch Check**: Target specific epochs (useful for finding missing rewards)
- **Multiple Time Ranges**: From recent epochs (~3 weeks) to full historical data
- **Multi-Provider RPC Fallback**: Rotates across Alchemy, Helius, and public endpoints automatically
- **Stop/Resume**: Stop long-running scans anytime; the database picks up where you left off
- **Koinly CSV Export**: Direct export in Koinly-compatible format for tax reporting

## How It Works

1. Enter your **Alchemy** and/or **Helius** API keys (recommended for browser use)
2. Enter your **Solana wallet address** (the withdrawer authority for your stake accounts)
3. Select **network** (Mainnet or Devnet)
4. Choose **time range** (default: Full History) or enter specific epochs manually
5. Click **Generate Report** — the app will:
   - Find all stake accounts associated with your wallet
   - Scan epochs for staking rewards (newest to oldest)
   - Store results in the local database
   - Display rewards in a table
6. **Download CSV** for import into Koinly

Full History scans can take 30–60+ minutes depending on RPC speed. Use **Stop** and re-run later; only unprocessed or failed epochs are scanned on subsequent runs.

## RPC Providers

The app tries endpoints in order until one succeeds. **Keyed providers work best in the browser** because many public RPCs block cross-origin requests (CORS).

| Provider | Signup | Notes |
|----------|--------|-------|
| [Helius](https://helius.dev) | Free (1M credits/mo) | Recommended backup; tried first when configured |
| [Alchemy](https://alchemy.com) | Free tier available | Primary provider |
| Alchemy Demo | No signup | Built-in fallback; lower limits |
| Solana Tracker | No signup | `rpc.solanatracker.io/public` |
| AeX402 | No signup | `rpc.aex402.com` |
| PublicNode | No signup | May fail from browser due to CORS |
| Solana Official | No signup | Often CORS-blocked in browser |

**Recommendation:** Add both an Alchemy and a Helius API key. When one is rate-limited or returns `-32001` (server overloaded), the app automatically switches to the other.

## Configuration

Settings are saved automatically in browser `localStorage`. You can also download a `config.json` via the **Save Config** button.

| Field | Description |
|-------|-------------|
| Alchemy API Key | Free key from [alchemy.com](https://alchemy.com) |
| Helius API Key | Free key from [helius.dev](https://helius.dev) |
| Wallet Address | Your Solana public key (withdrawer authority) |
| Network | Mainnet or Devnet |
| Time Range | Default is **Full History**; use "Last 90 epochs" for ~6 months only |

Example `config.json`:

```json
{
  "defaultWalletAddress": "YourWalletAddress...",
  "alchemyApiKey": "your-alchemy-key",
  "heliusApiKey": "your-helius-key"
}
```

## Database Storage

- **Local only**: All data stays in your browser via IndexedDB
- **Persistent**: Survives browser restarts; private to your browser
- **Incremental**: Only processes new or failed epochs — does not re-scan completed epochs
- **Location**: Browser → Developer Tools → Application → Storage → IndexedDB → `SolanaRewardsDB`

Use **Clear DB** to start fresh. Use **Retry Failed** to re-process epochs that failed due to RPC issues.

## Smart Retry System

- Tracks per-epoch success/failure with error type and retry count
- Distinguishes zero-reward epochs from failed scans (avoids infinite re-processing)
- Retries failed epochs on subsequent runs or via the **Retry Failed** button
- Excludes the most recent 2 epochs (data often not available yet)
- Uses estimated epoch timestamps when block time is unavailable for older slots

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Only ~6 months of data | Time range set to "Last 90 epochs" | Select **Full History** |
| Same epochs scanned repeatedly | Old database records | Hard refresh; app now skips completed epochs correctly |
| `Failed to fetch` | Browser CORS block on public RPC | Add **Helius** and/or **Alchemy** API keys |
| HTTP 429 / compute units | Rate limit on Alchemy | Add Helius key; app pauses and rotates providers |
| `-32001` / "Unable to complete request" | Alchemy temporarily overloaded | Normal; app skips Alchemy for 5 min and uses Helius/public RPCs |
| Scan stops early | Too many consecutive hard failures | Click **Retry Failed** later when RPCs recover |
| Missing rewards for specific epochs | RPC failure during scan | Use **Check Specific** and enter epoch numbers |

## Files

- `SolanaStaking.html` — Main application (single HTML file, no build step)
- `CursorChat.md` — Development log and conversation history

## Usage

1. Open `SolanaStaking.html` in your web browser
2. Enter your **Alchemy** and **Helius** API keys
3. Enter your Solana wallet public key
4. Confirm **Full History** is selected (or choose a shorter range)
5. Click **Generate Report**
6. Download the CSV when complete (or after multiple incremental runs)

## Technical Details

- **Frontend**: Pure HTML/CSS/JavaScript (no build process required)
- **Storage**: IndexedDB for local data persistence
- **API**: Solana JSON-RPC (`getInflationReward`, `getProgramAccounts`, `getBlockTime`)
- **Export**: CSV format compatible with Koinly tax software

## Branch Structure

- `main` — Production-ready releases
- `develop` — Development branch for integration
- `feature/*` — Feature development branches

## Contributing

1. Fork the repository
2. Create a feature branch from `develop`
3. Make your changes
4. Submit a pull request to `develop`

## License

This project is open source and available under the MIT License.

## Support

For issues or questions, open an issue on GitHub or check `CursorChat.md` for development history and troubleshooting notes.
