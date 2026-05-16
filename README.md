# debank — Claude Code Skill

Give Claude the ability to fetch a wallet's full DeFi portfolio from [DeBank](https://debank.com) — total balance, token holdings, and protocol positions across all chains. Supports plain display and CSV export (simple or full).

## What you say → what Claude does

**"Show me the portfolio of 0x754F...AB47"**

↓ Claude navigates to `https://debank.com/profile/0x754F3FD759f41cc8757CB37Ad4e319E390e6AB47`, waits for data to render, and returns:

```
Total Balance: $X,XXX,XXX

Tokens:
  ETH    X.XX    $X,XXX   [Ethereum]
  USDC   X,XXX   $X,XXX   [Arbitrum]
  ...

Protocols:
  Aave v3     Lending    $X,XXX
  Uniswap v3  LP         $X,XXX
  ...
```

**"Export it as CSV"** → writes `debank_0x754fab47.csv` (simple) or `debank_0x754fab47_full.csv` (full, with token addresses, protocol URLs, etc.)

## How it works

DeBank requires real browser rendering — the skill uses Playwright to navigate the profile page, wait for async data to load, and extract the portfolio snapshot.

## Installation

### Via Claude Code plugin marketplace

```bash
/plugin marketplace add jsvisa/claude-marketplace
/plugin install debank@jsvisa-marketplace
```

### Manual

```bash
mkdir -p ~/.claude/skills/debank
curl -fsSL https://raw.githubusercontent.com/jsvisa/debank-skill/main/skills/debank/SKILL.md \
  -o ~/.claude/skills/debank/SKILL.md
```

## Requirements

### Playwright MCP

This skill requires the [Playwright MCP server](https://github.com/microsoft/playwright-mcp) for browser rendering. Install it with:

```bash
npm install -g @playwright/mcp
```

Then add it to your Claude Code MCP config (`~/.claude/mcp.json` or via `/mcp add`):

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

### Using a different browser

If the default Playwright browser doesn't work (e.g. missing system Chromium), point it at a browser you already have installed via `~/.mcp.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/mcp@latest",
        "--browser", "chromium",
        "--executable-path", "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
      ],
      "env": {},
      "type": "stdio"
    }
  }
}
```

Common paths:

| Browser | Path |
|---------|------|
| Google Chrome (Mac) | `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome` |
| Brave (Mac) | `/Applications/Brave Browser.app/Contents/MacOS/Brave Browser` |
| Chromium (Linux) | `/usr/bin/chromium-browser` |
| Google Chrome (Linux) | `/usr/bin/google-chrome` |

## License

MIT
