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

## CSV Export

If the user requests a CSV export, write the portfolio to a file after extracting the data.
Two formats are available: **simple** (default) and **full** (richer fields, use when user asks for "full" or "detailed").

### Simple Format

Two sections in one file, separated by a blank line:

```
chain,section,name,type,amount,price_usd,value_usd
Base,Wallet,ETH,Token,5.9992,2183.50,13099.18
Base,Wallet,weETH,Token,2.5750,2389.69,6153.44
...

chain,section,protocol,type,details,value_usd
Base,Protocol,Hydrex,Liquidity Pool,"weETH/WETH — 68.21 weETH + 167.76 WETH",529294.47
Ethereum,Protocol,ether.fi,Locked,"164.05 ETH",358208.59
...
```

### Full Format

All simple fields plus additional metadata extracted from the snapshot where available:

**Wallet tokens** (`debank_{address_short}_full.csv`):
```
chain,section,name,type,amount,price_usd,value_usd,token_address,token_symbol,protocol_adapter
Base,Wallet,ETH,Token,5.9992,2183.50,13099.18,,ETH,
Base,Wallet,weETH,Token,2.5750,2389.69,6153.44,0x04c0599ae5a44757c0af6f9ec3b93da8976c150a,weETH,
...
```

**Protocol positions** (appended after a blank line):
```
chain,section,protocol,type,pool,value_usd,token_address,token_symbol,amount,protocol_url,controller
Base,Protocol,Hydrex,Liquidity Pool,weETH/WETH,529294.47,"0x04c0599ae5a44757c0af6f9ec3b93da8976c150a,0x4200000000000000000000000000000000000006","weETH,WETH","68.2099,167.7555",https://www.hydrex.fi/,
Ethereum,Protocol,ether.fi,Locked,Withdraw Request,358208.59,,ETH,164.0525,https://www.ether.fi/,
Optimism,Protocol,Merkl,Rewards,,339.58,0x4200000000000000000000000000000000000042,OP,2528.5108,https://app.merkl.xyz,
```

**Full format extra fields:**

| Field | Source in snapshot |
|-------|--------------------|
| `token_address` | Token link href — extract contract address from `/token/{chain}/{address}` path; leave empty for native tokens |
| `token_symbol` | Token name text |
| `protocol_adapter` | Protocol name from position header (for wallet tokens held via a protocol) |
| `pool` | Pool column text (for protocol positions) |
| `protocol_url` | href from the external link next to the protocol name |
| `controller` | Controller address if shown in the position (leave empty if not present) |

For multi-token pools (LP positions), use comma-separated values within a single quoted field for `token_address`, `token_symbol`, and `amount`.

### Steps

1. Extract all wallet tokens and protocol positions from the snapshot.
2. Default to simple format unless the user asks for full/detailed.
3. Write to `debank_{address_short}.csv` (simple) or `debank_{address_short}_full.csv` (full), using the Write tool.
4. Confirm the file path to the user.

### Rules

- Column order: `chain` first, then `section`, then asset fields.
- Wrap values containing commas or special characters in double quotes.
- Omit price for protocol positions (leave the field empty).
- Use the short chain name (e.g. `Base`, `Ethereum`, `Optimism`).
- Leave a field empty rather than guessing if the data is not in the snapshot.

## Notes

- DeBank loads data progressively via async requests — always wait before snapshotting
- Chain filter tabs (All / Ethereum / Arbitrum / ...) default to "All" — keep it on All for full portfolio
- Some wallets have 100+ tokens; use "Show more" or scroll to capture the full list
- Protocol positions appear in the lower section below the token list
