---
name: zora-limit-orders
description: Configure take-profit tiers and stop-loss orders on Zora coin positions. Monitors prices every 5 minutes and auto-executes sells when targets are hit. Supports multi-tier TPs (sell X% at +Y%, more at +Z%), per-coin custom overrides, and global defaults. Use when a user wants to set price targets, automate profit-taking, protect against losses, or manage an open position. Triggers on phrases like "set a take profit", "set a stop loss", "sell when it hits", "take profits at", "protect my position", "set TP", "set SL", or "limit order".
---

# zora-limit-orders

Automated TP/SL execution for Zora positions. Monitors prices, sells when targets hit.

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
/orders <set-tp|set-sl|list|clear>
```

**Examples:**
```
/orders set-tp 0x834f77c... 25 33
/orders set-sl 0x834f77c... 15
/orders list
/orders clear 0x834f77c...
```
## Default Configuration

```json
{
  "tpOrders": [
    { "pct": 25, "sellPct": 33 },
    { "pct": 50, "sellPct": 33 },
    { "pct": 100, "sellPct": 34 }
  ],
  "stopLossPct": 15
}
```

Applies to all positions unless overridden per-coin.

## Price Monitoring Loop

Run every 5 minutes. Use Zora SDK for fast parallel fetching:

```js
// Find SDK dynamically (works on any install)
const sdkPath = require('child_process')
  .execSync("find ~/.local -path '*/coins-sdk/dist/index.cjs' 2>/dev/null | head -1")
  .toString().trim();
const { getCoin } = require(sdkPath);

async function fetchPriceBulk(addresses) {
  const results = await Promise.all(addresses.map(a => getCoin({ address: a })));
  const map = {};
  addresses.forEach((a, i) => {
    const c = results[i]?.data?.zora20Token;
    if (c) map[a] = parseFloat(c.marketCap) / 1_000_000_000;
  });
  return map;
}
// ~334ms for 4 coins in parallel
```

## TP/SL Execution Logic

```js
for (const address of openPositions) {
  const currentPrice = priceMap[address];
  const ref = pos.avgBuyPrice || pos.buyPriceUsd;
  if (!ref || !currentPrice) continue;

  const changePct = (currentPrice - ref) / ref * 100;
  const activeTps = pos.customTpOrders || globalTpOrders;

  // Take profit (fire one tier per cycle)
  for (let i = 0; i < activeTps.length; i++) {
    if (pos.tpHits[i]) continue;
    if (changePct >= activeTps[i].pct) {
      await executeSell(address, activeTps[i].sellPct);
      pos.tpHits[i] = true;
      break;
    }
  }

  // Stop loss
  if (!pos.fullyExited && changePct <= -stopLossPct) {
    await executeSell(address, 100);
    pos.fullyExited = true;
  }
}
```

## Sell Command

```bash
zora sell <address> --percent <pct> --yes --json
```

`--yes` skips interactive confirmation. `--json` returns machine-readable output with `txHash`.

## Per-Coin TP Overrides

Store custom TPs in `positions.json`:
```json
{
  "0x...": {
    "customTpOrders": [
      { "pct": 10, "sellPct": 50 },
      { "pct": 30, "sellPct": 50 }
    ],
    "tpHits": [false, false]
  }
}
```

When `customTpOrders` is `null`, global defaults apply.
When set, only per-coin orders fire for that position.

## Sell Initials

Calculates the % to sell to recover original investment:
```js
const currentMultiple = currentPrice / ref;
const pctToSell = Math.min(100, Math.round(100 / currentMultiple));
// e.g. coin 2x'd → sell 50% to get initial back, hold 50% free
```

## Strategy Notes

- Break-even win rate (TP1 +25% vs SL -15%): ~26.7% on blue chips
- Only fire one TP tier per 5-min check cycle (avoids race conditions)
- Reset TP hit flags if position is added to after a partial sell
- Keep `fullyExited` positions in history — don't delete, just filter from active checks

## Related Skills

- `zora-dca` — creates positions this skill monitors
- `zora-portfolio` — view positions and configure TPs interactively
