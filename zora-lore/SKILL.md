---
name: zora-lore
description: Deep token research combining on-chain data, market data, and social signals for any Base chain token. Use when a user wants to research a coin before buying — pulls from Zora SDK (holders, creator, social accounts), DexScreener (price, liquidity, volume), and Farcaster/Neynar (creator following, recent posts). Triggers on phrases like "research this coin", "tell me about this token", "who made this", "is this legit", "full report on", or when a user shares a CA and asks for analysis rather than an immediate buy.
---

# zora-lore

Full token intelligence report. Combines market data, creator identity, and social signals.

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
/lore <0x... or @handle>
```

**Examples:**
```
/lore 0x834f77c...
/lore @jessepollak
/lore peptides
```
## Data Sources

### 1. Zora SDK (creator + holder data)

The SDK is bundled with the CLI. Find it:
```bash
# Find SDK location
find ~/.local -name "coins-sdk" -type d 2>/dev/null | head -1
# Typically: ~/.local/lib/node_modules/@zoralabs/cli/node_modules/@zoralabs/coins-sdk
```

```js
const sdkPath = require('child_process')
  .execSync("find ~/.local -path '*/coins-sdk/dist/index.cjs' 2>/dev/null | head -1")
  .toString().trim();
const { getCoin } = require(sdkPath || '@zoralabs/coins-sdk');

const result = await getCoin({ address });
const coin = result?.data?.zora20Token;
// coin.name, coin.uniqueHolders, coin.creatorHandle, coin.creatorAddress
// coin.socialAccounts.twitter.username, coin.socialAccounts.twitter.followerCount
// coin.socialAccounts.farcaster.username, coin.socialAccounts.farcaster.followerCount
// coin.marketCap (string), coin.createdAt
```

### 2. DexScreener (market data)

```js
// Same as zora-scan — see that skill for dexFetch() implementation
const pair = await dexFetch(`https://api.dexscreener.com/latest/dex/tokens/${address}`);
```

### 3. Neynar / Farcaster (optional, richer social)

Requires `NEYNAR_API_KEY` env var (free tier: neynar.com).

```js
const handle = coin?.socialAccounts?.farcaster?.username;
if (handle && process.env.NEYNAR_API_KEY) {
  const res = await fetch(
    `https://api.neynar.com/v2/farcaster/user/by_username?username=${handle}`,
    { headers: { 'api_key': process.env.NEYNAR_API_KEY } }
  );
  const user = (await res.json()).user;
  // user.follower_count, user.power_badge, user.profile.bio.text
}
```

If no Neynar key, fall back to follower count from Zora SDK.

## Report Format

```
🔍 PEPTIDES (peptides)
`0x834f77c66f9042793de053306deb702263eb588a`

🏭 Factory: Zora ✅  |  Created: Mar 14 2025

📊 Market
  Mcap:    $15,297  (+7.2% 24h)
  Liq:     $15,618  (102% of mcap) 🟢
  Vol 24h: $51,762
  Txns:    362 buys / 505 sells

👤 Creator: @peptides_xyz
  Twitter:   2.3k followers
  Farcaster: 890 followers  ⚡ Power Badge

⚠️ Risk Signals:
  - Low holder count (142) — thin distribution
  - Vol/Mcap ratio 338% — speculative

🔗 zora.co/trend/peptides
```

## Risk Signal Logic

| Condition | Flag |
|-----------|------|
| `liq < $1,000` | 🚨 Very low liquidity |
| `liq < $5,000` | ⚠️ Low liquidity |
| `holders < 50` | ⚠️ Thin distribution |
| `holders < 10` | 🚨 Likely wash trading |
| `vol/mcap > 200%` | ⚠️ High speculation |
| No social accounts linked | ⚠️ Anonymous creator |
| Created < 7 days ago | ℹ️ New coin |

## priceUsd Calculation for Zora Coins

Zora coins have 1 billion total supply:
```js
const priceUsd = parseFloat(coin.marketCap) / 1_000_000_000;
```

## After the Report

Offer:
- Buy (see zora-scan/zora-dca)
- Add to follow list (see zora-follow)
- Set limit orders (see zora-limit-orders)

## Notes

- Twitter follower count comes from Zora SDK — no X API key needed
- SDK `getCoin()` may return null for non-Zora tokens — fall back to DexScreener only
- Always `parseFloat()` on `marketCap` — it's a string in SDK responses
- Rate limit DexScreener at 1 req/sec
- Cache Neynar responses (100 req/day free tier)
