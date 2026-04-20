---
name: zora-follow
description: Watch Zora wallets and users — get notified whenever they create a coin, buy, or sell. Configurable per-wallet event filters (creates/buys/sells/all). Optional copy-trade mode auto-buys when a watched wallet buys. Use when a user wants to track alpha wallets, follow a creator's activity, copy-trade a specific address, or get alerts on wallet activity. Triggers on phrases like "follow this wallet", "watch this address", "alert me when", "copy trade", "track this user", or "notify me when @handle buys".
---

# zora-follow

Wallet activity alerts. Follow any address or handle — get notified on creates, buys, and sells.

## Zora CLI Check

```bash
which zora 2>/dev/null || echo "not found"
```

If missing:
```bash
npm install -g @zoralabs/cli --prefix ~/.local
export PATH="$HOME/.local/bin:$PATH"
```


## Slash Command

```
/follow <add|remove|list|pause>
```

**Examples:**
```
/follow add 0x... buys
/follow add @jessepollak creates,buys
/follow list
/follow remove 0x...
/follow copy 0x... 0.005
```
## Setup Wizard

1. **Wallet or handle** — paste `0x...` address or Zora `@handle`
2. **Event types** — `creates` / `buys` / `sells` / `all`
3. **Copy-trade?** — Yes (specify ETH amount per copy) | No
4. **Confirm** → begin watching

## Alert Format

```
🔔 @jessepollak bought PEPTIDES
  0.05 ETH  |  Mcap: $15,297  (+7.2%)
  zora.co/trend/peptides

[🟢 Copy Buy 0.005 ETH]  [📊 Scan]
```

## Polling Architecture

Poll every 2 minutes using Zora SDK profile activity:

```js
// Find SDK path dynamically
const { execSync } = require('child_process');
const sdkCjs = execSync(
  "find ~/.local -path '*/coins-sdk/dist/index.cjs' 2>/dev/null | head -1"
).toString().trim();
const { getProfile } = require(sdkCjs);
```

Check recent activity:
```js
async function checkWallet(address, lastSeenAt) {
  const result = await getProfile({ identifier: address });
  const activity = result?.data?.profile?.activity?.edges || [];

  const newEvents = activity
    .filter(e => new Date(e.node.timestamp).getTime() > lastSeenAt)
    .map(e => e.node);

  return newEvents;
}
```

Update `lastSeenAt` after each check to avoid duplicate alerts.

## Event Types

| SDK event type | Filter key |
|---------------|------------|
| Coin created | `creates` |
| Buy / swap in | `buys` |
| Sell / swap out | `sells` |

## Watch List Config

```json
{
  "watchList": [
    {
      "address": "0x...",
      "handle": "jessepollak",
      "events": ["buys", "creates"],
      "copyTrade": false,
      "copyTradeEth": 0.005,
      "lastSeenAt": 1713456789000,
      "paused": false
    }
  ],
  "alertChannels": {
    "telegram": true,
    "discord": false,
    "discordChannelId": null
  }
}
```

## Copy-Trade Logic

When a watched wallet buys and `copyTrade: true`:

```bash
zora buy <coinAddress> --eth <copyTradeEth> --yes --json
```

Guards:
- Skip if position already open for that coin (no doubling)
- Skip if buyer wallet balance < copyTradeEth
- Log all copy trades with original wallet + txHash

## Enriching Alerts with Market Data

After detecting a buy event, fetch quick market data:
```js
// Get coin address from event, then DexScreener scan
const pair = await dexFetch(`https://api.dexscreener.com/latest/dex/tokens/${coinAddress}`);
const mcap = pair ? `$${parseFloat(pair.marketCap).toLocaleString()}` : '?';
const chg = pair?.priceChange?.h24;
```

## Rate Limiting

- Poll every 2 min max
- Stagger polls if > 5 wallets (don't hit all at once)
- Cache profile data for 60s
- Zora API is public — be respectful, don't hammer

## Notes

- Zora SDK `getProfile()` may have 1-5 min delay on activity
- For faster alerts: index on-chain `Transfer` events via Alchemy/Helius RPC (advanced)
- Filter out dust buys < 0.001 ETH to reduce noise
- `@handle` identifiers must be valid Zora usernames (not Twitter handles)

## Related Skills

- `zora-scan` — quick scan on coins the followed wallet is buying
- `zora-lore` — deep research on a coin from the alert
- `zora-dca` — set up recurring buys into coins a wallet keeps buying
