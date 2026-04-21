---
name: zora-dca
description: Recurring automated buys into Zora coins on a configurable schedule. Supports leaderboard auto-selection (top blue chips by market cap and holders) and custom coins by contract address. Includes Growth Mode for rapid TWAP-style accumulation with randomized timing. Use when a user wants to set up recurring buys, dollar-cost average into a coin or basket, or execute a timed buy sequence. Triggers on phrases like "DCA into", "buy every", "set up recurring buys", "accumulate", "growth engine", "TWAP", or "buy X ETH over Y minutes".
---

# zora-dca

Recurring automated buys into Zora coins. Leaderboard mode, custom coins, or rapid Growth Mode.

## Zora CLI Check

```bash
which zora 2>/dev/null || echo "not found"
```

If missing:
```bash
npm install -g @zoralabs/cli --prefix ~/.local
export PATH="$HOME/.local/bin:$PATH"
zora setup --create   # generates wallet, saves to ~/.config/zora/wallet.json
```

Verify: `zora balance --json` should return your ETH balance on Base.


## Slash Command

```
/dca <setup|start|stop|status|add|growth>
```

**Examples:**
```
/dca setup
/dca add 0x834f77c... 0.002
/dca growth 0x834f77c... 0.01 10 1min
/dca status
/dca stop
```
## Setup Wizard

Run through these questions before starting:

1. **Wallet** — `~/.config/zora/wallet.json` (default) or set `ZORA_PRIVATE_KEY` env var
2. **Mode** — `leaderboard` | `custom` | `both`
3. **Leaderboard filters** (if leaderboard mode):
   - Min market cap (default: $100,000)
   - Min holders (default: 1,000)
   - Max coins to buy per cycle (default: 5)
4. **Custom coins** (if custom mode) — paste addresses + ETH per cycle each
5. **ETH per coin per cycle** (default: 0.002 ETH)
6. **Interval** — `1h` / `4h` / `6h` / `12h` / `24h` / `1 week` / `1 month` / custom hours
7. **Confirm** → start paused, enable when ready

## Running a DCA Cycle

### Leaderboard Selection

```bash
# Get top coins by mcap
zora explore --sort mcap --json
```

```js
const coins = JSON.parse(output).coins;
const selected = coins
  .filter(c => c.marketCap >= minMcap)
  .filter(c => c.uniqueHolders >= minHolders)
  .filter(c => !c.platformBlocked)
  .slice(0, maxCoins);
```

Anti-bot filter: skip coins with `uniqueHolders < 10`.

### Executing Buys

```bash
zora buy <address> --eth <amount> --yes --json
```

Response contains `txHash`. Record it.

### Price Tracking at Buy Time

Fetch price immediately after buy to enable P&L tracking:

```js
// SDK path — find dynamically
const sdkPath = require('child_process')
  .execSync("find ~/.local -path '*/coins-sdk/dist/index.cjs' 2>/dev/null | head -1")
  .toString().trim();
const { getCoin } = require(sdkPath);

const r = await getCoin({ address });
const priceUsd = parseFloat(r.data.zora20Token.marketCap) / 1_000_000_000;
```

DCA-average across multiple buys:
```js
const totalEth = pos.totalEthSpent + thisEth;
pos.avgBuyPrice = ((pos.avgBuyPrice * pos.totalEthSpent) + (priceUsd * thisEth)) / totalEth;
pos.totalEthSpent = totalEth;
```

## Growth Mode (Short-Interval TWAP)

Rapid accumulation with ±15% jitter on amount and timing:

```js
function jitter(value, variance = 0.15) {
  return Math.max(0.0001, value * (1 + (Math.random() * 2 - 1) * variance));
}

// Example: 0.01 ETH over 10 min in 10 buys
// Each buy: ~0.00085-0.00115 ETH every ~51-69 seconds
```

Wizard questions for Growth Mode:
1. Coin address (or pick from open positions)
2. Total ETH to spend — presets: `0.005` / `0.01` / `0.02` / `0.05` / `0.1` / Custom
3. Number of buys — presets: `3` / `5` / `10` / `20` / `50` / Custom
4. Interval — presets: `30s` / `1 min` / `2 min` / `5 min` / `10 min` / `30 min` / Custom
5. Confirm: "Buy ~0.001 ETH every ~1 min × 10 = ~0.01 ETH total. 🎲 ±15% jitter. Start?"

## Position Schema

Store in `positions.json`:
```json
{
  "0x...": {
    "coinName": "jessepollak",
    "address": "0x...",
    "buyPriceUsd": 0.00144,
    "avgBuyPrice": 0.00142,
    "totalEthSpent": 0.008,
    "buyCount": 4,
    "boughtAt": 1713456789000,
    "source": "dca",
    "tpHits": [false, false, false],
    "customTpOrders": null,
    "fullyExited": false
  }
}
```

## Config Schema

```json
{
  "enabled": false,
  "intervalHours": 6,
  "ethPerCoin": 0.002,
  "maxCoins": 5,
  "minMcap": 100000,
  "minHolders": 1000,
  "customCoins": [
    { "address": "0x...", "name": "peptides", "ethPerCycle": 0.002 }
  ],
  "nextDcaAt": null,
  "lastDcaAt": null
}
```

## Strategy Notes

- Blue chip DCA EV: slightly positive at TP1 +25% vs SL -15% (break-even win rate: ~27%)
- 4x/day DCA = sqrt(4) ≈ 50% reduction in timing variance vs lump sum
- Recommend starting with 0.001 ETH/coin to observe before scaling
- Separate DCA wallet from main wallet to track performance cleanly

## Token Selection

The Zora CLI supports buying with ETH, USDC, or $ZORA:

```bash
zora buy <address> --eth 0.005 --token eth    # default
zora buy <address> --eth 0.005 --token zora   # spend $ZORA
zora buy <address> --usd 10    --token usdc   # spend USDC
```

**Smart selection:** check wallet balances and use whichever token has the highest USD value:

```js
function getBestToken(walletBalance) {
  const eth  = walletBalance.find(t => t.symbol === 'ETH');
  const zora = walletBalance.find(t => t.symbol === 'ZORA');
  if (zora && parseFloat(zora.usdValue) > parseFloat(eth?.usdValue || 0)) return 'zora';
  return 'eth';
}
```

## Related Skills

- `zora-limit-orders` — configure TP/SL on DCA positions
- `zora-portfolio` — view all DCA positions with live P&L
- `zora-scan` — research a coin before adding to DCA
