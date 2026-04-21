---
name: zora-scan
description: Instant token signal card for any Base chain contract address. Use when a user pastes a 0x contract address and wants quick market data — price, market cap, liquidity, volume, buy/sell pressure, and platform detection (Zora, Virtuals, Clanker, Uniswap). Works for any Base DEX token. Also triggers on phrases like "scan this token", "what's this coin", "check this CA", "look up this address".
---

# zora-scan

Instant market snapshot for any Base chain token. One API call, one clean card.

## Zora CLI Check

Before any trading actions, verify Zora CLI is available:

```bash
which zora 2>/dev/null || echo "not found"
```

If not found, install:
```bash
npm install -g @zoralabs/cli --prefix ~/.local
export PATH="$HOME/.local/bin:$PATH"
```

Verify: `zora --version` should output `1.x.x`.

> Note: add `export PATH="$HOME/.local/bin:$PATH"` to `~/.bashrc` or `~/.zshrc` to persist.


## Slash Command

```
/scan <0x...>
```

**Examples:**
```
/scan 0x834f77c66f9042793de053306deb702263eb588a
/scan peptides
Or just paste any 0x address directly
```
## Scanning a Token

Fetch from DexScreener (no API key, works for all Base tokens):

```js
const https = require('https');

function dexFetch(url) {
  return new Promise((resolve, reject) => {
    https.get(url, { headers: { 'User-Agent': 'zora-scan/1.0' } }, res => {
      let data = '';
      res.on('data', c => data += c);
      res.on('end', () => { try { resolve(JSON.parse(data)); } catch { resolve(null); } });
    }).on('error', reject);
  });
}

async function scanToken(address) {
  const res = await dexFetch(`https://api.dexscreener.com/latest/dex/tokens/${address}`);
  const pair = (res?.pairs || []).find(p => p.chainId === 'base');
  if (!pair) return null;
  return pair;
}
```

Rate limit: 1 req/sec. Cache results for 30s to avoid 429s.

## Output Format

```
PEPTIDES (peptides)
0x834f77c...588a

🏭 Zora ✅
📊 Mcap:  $15,297  +7.2% (24h)  +1.1% (1h)
💧 Liq:   $15,618
📦 Vol:   $51,762 (24h)
🔄 Txns:  362 🟢 / 505 🔴
📅 Since: Mar 14 2025
```

Show liquidity warning if `liq < $1,000`: `⚠️ VERY LOW LIQUIDITY`
Show warning if `liq < $5,000`: `⚠️ Low liquidity`

## Factory Detection

```js
function detectFactory(pair) {
  const sites = pair?.info?.websites || [];
  const labels = pair?.labels || [];
  if (sites.some(w => w.url?.includes('zora.co')))     return '🟣 Zora';
  if (sites.some(w => w.url?.includes('virtuals.io'))) return '🤖 Virtuals';
  if (sites.some(w => w.url?.includes('clanker')))     return '🔧 Clanker';
  if (labels.includes('v3')) return '🦄 Uniswap V3';
  if (labels.includes('v2')) return '🦄 Uniswap V2';
  return '❓ Unknown';
}
```

## Fields to Extract

| Field | DexScreener path |
|-------|-----------------|
| Name | `pair.baseToken.name` |
| Symbol | `pair.baseToken.symbol` |
| Market cap | `pair.marketCap` |
| Price USD | `pair.priceUsd` |
| Liquidity | `pair.liquidity.usd` |
| Volume 24h | `pair.volume.h24` |
| Price chg 24h | `pair.priceChange.h24` |
| Price chg 1h | `pair.priceChange.h1` |
| Buys 24h | `pair.txns.h24.buys` |
| Sells 24h | `pair.txns.h24.sells` |
| Created | `pair.pairCreatedAt` (unix ms) |

## After the Scan

Offer quick actions:
- Buy immediately via `zora buy <address> --eth <amount> --yes`
- Add to DCA schedule (see zora-dca skill)
- Deep research (see zora-lore skill)

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

## Error Handling

- No pairs found → "No trading data found on Base for this address"
- Network error → retry once, then surface error
- `marketCap` may be string or number — always `parseFloat()`
- `priceNative` may be null — guard before dividing
