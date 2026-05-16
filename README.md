# debank — Claude Code Skill

Give Claude the ability to fetch a wallet's full DeFi portfolio from [DeBank](https://debank.com) — total balance, token holdings, and protocol positions across all chains.

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

- Claude Code with Playwright MCP enabled (for browser rendering)

## License

MIT
