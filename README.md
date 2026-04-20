# ZoraCLI Skills

> Claude Code / OpenClaw agent skills for advanced systems in the Zora CLI. 

---

## Skills

| Skill | Description | Install |
|-------|-------------|---------|
| **zora-scan** | Instant token signal — drop any Base CA, get market data | [Download](./dist/zora-scan.skill) |
| **zora-lore** | Deep token research: market + creator + social signals | [Download](./dist/zora-lore.skill) |
| **zora-dca** | Recurring buys into leaderboard or custom coins + Growth Mode | [Download](./dist/zora-dca.skill) |
| **zora-limit-orders** | Automated take-profit tiers and stop-loss execution | [Download](./dist/zora-limit-orders.skill) |
| **zora-portfolio** | Live P&L, position management, buy/sell from one panel | [Download](./dist/zora-portfolio.skill) |
| **zora-follow** | Watch wallets — alerts on creates/buys/sells, copy-trade | [Download](./dist/zora-follow.skill) |

---

## Requirements

- [OpenClaw](https://openclaw.ai) — AI agent platform
- [Zora CLI](https://cli.zora.com) — command-line trading interface
- Node.js 20+
- ETH on Base chain (fund your wallet after setup)

---

## Quick Start

```bash
# 1. Install Zora CLI
npm install -g @zoralabs/cli

# 2. Create a trading wallet
zora setup --create
# → saves wallet to ~/.config/zora/wallet.json

# 3. Fund your wallet
# Send ETH on Base to the address shown above

# 4. Verify
zora balance --json
```

---

## Installing a Skill

**Option A: Download and install**
1. Download a `.skill` file from the `dist/` folder above
2. In OpenClaw: `/skill install path/to/zora-scan.skill`

**Option B: Install from URL**
```
/skill install https://raw.githubusercontent.com/zak802/ZoraCLI_skills/main/dist/zora-scan.skill
```

---

## Slash Commands

| Skill | Command |
|-------|---------|
| zora-scan | `/scan <0x...>` or just paste a contract address |
| zora-lore | `/lore <0x... or @handle>` |
| zora-dca | `/dca setup` · `/dca add <address> <eth>` · `/dca growth` |
| zora-limit-orders | `/orders set-tp <address> <pct> <sell%>` · `/orders set-sl` |
| zora-portfolio | `/portfolio` · `/portfolio <address>` |
| zora-follow | `/follow add <address>` · `/follow copy <address> <eth>` |

---

## About

These skills were built while developing a Zora trading bot and extracted into reusable agent skills. They work together — scan a token, lore it, DCA in, set limit orders, track in portfolio, follow the creator.

---

## License

MIT
