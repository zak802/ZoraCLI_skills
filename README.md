# Zora Skills

OpenClaw agent skills for trading on Zora — the attention market on Base.

Built by [@zak_krevitt](https://zora.co/@zak)

---

## Skills

| Skill | Description |
|-------|-------------|
| [zora-scan](./zora-scan/SKILL.md) | Instant token signal card — drop any Base CA, get market data |
| [zora-lore](./zora-lore/SKILL.md) | Deep token research: market + creator + social signals |
| [zora-dca](./zora-dca/SKILL.md) | Recurring buys into leaderboard or custom coins + Growth Mode |
| [zora-limit-orders](./zora-limit-orders/SKILL.md) | Automated take-profit tiers and stop-loss execution |
| [zora-portfolio](./zora-portfolio/SKILL.md) | Live P&L, position management, buy/sell from one panel |
| [zora-follow](./zora-follow/SKILL.md) | Watch wallets — alerts on creates/buys/sells, optional copy-trade |

---

## Requirements

- [OpenClaw](https://openclaw.ai) agent
- [Zora CLI](https://cli.zora.com) (`npm install -g @zoralabs/cli --prefix ~/.local`)
- Node.js 20+
- A funded Base wallet (ETH on Base chain)

---

## Installation

1. Download a `.skill` file from [ClaWHub](https://clawhub.ai)
2. Install via OpenClaw: `/skill install zora-scan.skill`
3. Or clone this repo and use locally

---

## Quick Start

```bash
# Install Zora CLI
npm install -g @zoralabs/cli --prefix ~/.local
export PATH="$HOME/.local/bin:$PATH"

# Create a wallet
zora setup --create

# Fund your wallet (send ETH on Base to the displayed address)

# Start scanning
# In your OpenClaw chat: paste any Base contract address
```

---

## License

MIT
