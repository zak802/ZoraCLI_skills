---
name: zora-portfolio
description: View and manage all open Zora coin positions with live P&L in ETH and USD. Shows market cap movement, liquidity, volume, and buy/sell pressure per coin. Allows manual buys, partial sells, sell-to-initials, and TP configuration from a single interface. Use when a user wants to see their holdings, check profit/loss, manage a position, or take action on an existing investment. Triggers on phrases like "show my positions", "how am I doing", "my portfolio", "check my holdings", "what do I own", or "manage my coins".
---

# zora-portfolio

Unified view of all Zora positions. Live P&L, market data, and one-tap management.

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
/portfolio <list|detail|sell|buy>
```

**Examples:**
```
/portfolio
/portfolio 0x834f77c...
/portfolio sell 0x834f77c... 50
/portfolio sell-initials 0x834f77c...
```
## Listing Positions

Load from `positions.json`. Sort: open first, exited at bottom.

```js
const allPositions = JSON.parse(fs.readFileSync('positions.json'));
const open   = Object.entries(allPositions).filter(([, p]) => !p.fullyExited);
const exited = Object.entries(allPositions).filter(([, p]) =>  p.fullyExited);
```

Show each as: `🟢 peptides  +12.3%` or `🔴 balajis  -2.1%` or `⚪ jacob  (exited)`

## Coin Detail Panel

Fetch market data from DexScreener + Zora SDK in parallel:

```js
const [sdkData, dexData] = await Promise.all([
  getCoin({ address }),  // SDK for holder count, Zora-specific fields
  dexFetch(`https://api.dexscreener.com/latest/dex/tokens/${address}`)
]);
```

Display format:
```
PEPTIDES  🟢 OPEN
🟢 +12.34%  +0.00062 ETH  (~+$1.48)
In: 0.00500 ETH

Mcap: $15,297  *+7.2%*  1h: +1.1%
Liq: $15,618  |  Vol: $51,762
362🟢 505🔴 (24h)

TPs: ⬜+25%  ⬜+50%  ⬜+100%  |  SL: -15%
```

## P&L Calculation

```js
const ref = pos.avgBuyPrice || pos.buyPriceUsd;
const currentPrice = parseFloat(sdkData.data.zora20Token.marketCap) / 1_000_000_000;

const changePct = (currentPrice - ref) / ref * 100;
const currentValueEth = ethSpent * (currentPrice / ref);
const pnlEth = currentValueEth - ethSpent;

// Estimate ETH price from DexScreener pair
const ethPrice = pair?.priceUsd && pair?.priceNative
  ? parseFloat(pair.priceUsd) / parseFloat(pair.priceNative)
  : 2400; // fallback

const pnlUsd = pnlEth * ethPrice;
```

Only display P&L line when `|changePct| > 0.01%` (suppress noise at entry).

## Buy More

```bash
zora buy <address> --eth <amount> --yes --json
```

Update position after buy:
- Add ETH to `totalEthSpent`
- Recalculate `avgBuyPrice`
- Increment `buyCount`

## Sell Actions

```bash
# Sell percentage
zora sell <address> --percent <pct> --yes --json

# Sell initials (recover original investment)
pctToSell = Math.min(100, Math.round(100 / (currentPrice / ref)))
zora sell <address> --percent <pctToSell> --yes --json
```

Mark `fullyExited = true` when selling 100%.

## Wallet Routing

Positions have a `source` field. Use the right wallet:
```js
const walletEnv = pos.source === 'dca'
  ? { ZORA_PRIVATE_KEY: dcaWalletKey }
  : { ZORA_PRIVATE_KEY: buyerWalletKey };
```

## Address Lookup (Telegram / short IDs)

Telegram `callback_data` is limited to 64 bytes. Use short keys:
```js
const addrLookup = {};
function addrKey(addr) {
  const k = 'a' + addr.slice(2, 10).toLowerCase();
  addrLookup[k] = addr;
  return k;
}
// Rebuild from positions.json on startup:
Object.keys(positions).forEach(a => addrKey(a));
```

## SDK Path (portable)

```js
const { execSync } = require('child_process');
const sdkCjs = execSync(
  "find ~/.local -path '*/coins-sdk/dist/index.cjs' 2>/dev/null | head -1"
).toString().trim();
const { getCoin } = require(sdkCjs);
```

This works regardless of where `npm install -g` placed the CLI.

## Related Skills

- `zora-limit-orders` — set/edit TP and SL on any position
- `zora-dca` — configure recurring buys
- `zora-scan` — quick scan before buying more
