---
name: zora-bot-update
description: Update the ZoraCLI bot to the latest version from the upstream template. Use when a user says "update the zak bot", "update my bot", "pull latest bot changes", or "check for bot updates". Pulls from the upstream GitHub template, restarts the bot via PM2, and reports what changed.
---

# zora-bot-update

Pull the latest bot code from upstream and restart.

## Update Command

```bash
cd ~/zoraCLI-bot   # or wherever the bot is installed
git pull upstream main
pm2 restart zoraCLI-bot
```

## Full Flow

1. Find the bot directory (check common locations):
```bash
find ~ -name "bot.js" -path "*/zoraCLI*" 2>/dev/null | head -3
# or check pm2
pm2 list | grep zoraCLI
```

2. Check for upstream remote:
```bash
git remote -v | grep upstream
```

If missing, add it:
```bash
git remote add upstream https://github.com/zak802/zoraCLI-bot-template
```

3. Pull and restart:
```bash
git pull upstream main
pm2 restart zoraCLI-bot
```

4. Report what changed:
```bash
git log --oneline HEAD~5..HEAD
```

## What Gets Updated

- `bot.js` — new features, bug fixes
- `setup.sh` — wizard improvements
- `README.md` — docs

## What Never Changes (safe)

- `.env` — your credentials (gitignored)
- `positions.json` — your open trades (gitignored)
- `state.json` — your settings (gitignored)

## Slash Command

```
/update-bot
```

**Examples:**
```
/update-bot
update the zak bot
pull latest bot changes
```
