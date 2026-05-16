---
name: debank
description: Use when fetching a wallet's DeFi portfolio from DeBank — total balance, token holdings, and protocol positions across all chains.
---

# DeBank Portfolio

Fetch a wallet's full DeFi portfolio from `https://debank.com/profile/{address}`.

## How to Fetch

DeBank requires real browser rendering — always use Playwright browser tools, never curl.

### Steps

1. **Navigate** to the profile page:
   ```
   mcp__playwright__browser_navigate: https://debank.com/profile/{address}
   ```

2. **Wait** for portfolio data to render (loads asynchronously):
   ```
   mcp__playwright__browser_wait_for: text="Total Balance"
   ```
   If that times out, fall back to `browser_wait_for time=5`.

3. **Snapshot** the rendered page to extract data:
   ```
   mcp__playwright__browser_snapshot
   ```

4. **Scroll / paginate** if needed — token list and protocol positions may have "Show more" buttons. Click them before taking the final snapshot.

## Data Extracted

| Section | What to look for in snapshot |
|---------|------------------------------|
| **Total Balance** | USD value shown prominently at top of profile |
| **Token Holdings** | Token name, amount, USD value, chain label |
| **Protocol Positions** | Protocol name, position type (Staking/Lending/LP/Perps), USD value |

## Example

```
Address: 0x754F3FD759f41cc8757CB37Ad4e319E390e6AB47
URL:     https://debank.com/profile/0x754F3FD759f41cc8757CB37Ad4e319E390e6AB47
```

Expected output summary:
```
Total Balance: $X,XXX,XXX

Tokens:
  ETH    X.XX    $X,XXX   [Ethereum]
  USDC   X,XXX   $X,XXX   [Arbitrum]
  ...

Protocols:
  Aave v3     Lending     $X,XXX
  Uniswap v3  LP          $X,XXX
  ...
```

## Notes

- DeBank loads data progressively via async requests — always wait before snapshotting
- Chain filter tabs (All / Ethereum / Arbitrum / ...) default to "All" — keep it on All for full portfolio
- Some wallets have 100+ tokens; use "Show more" or scroll to capture the full list
- Protocol positions appear in the lower section below the token list
