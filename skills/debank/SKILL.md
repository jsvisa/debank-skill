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

Extracted from the page snapshot. Two sections in one file, separated by a blank line:

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

The full format is sourced from the DeBank API response, not the snapshot.
After page load, intercept the network call to get the raw JSON:

```
browser_network_requests  →  find URL matching /portfolio/project_list
browser_network_request(index, part="response-body")  →  parse JSON
```

The API endpoint is:
```
https://api.debank.com/portfolio/project_list?user_addr={address}
```

Flatten to **one row per token per position** so the output is query-friendly. The columns are:

```
chain,section,protocol_id,protocol_name,protocol_url,protocol_tvl,
position_type,detail_types,position_description,position_index,
adapter_id,controller,pool_id,pool_created_at,
asset_usd_value,debt_usd_value,net_usd_value,
token_address,token_symbol,token_display_symbol,token_name,
token_amount,token_price_usd,token_value_usd,token_decimals,token_chain,token_verified
```

**Field mapping from JSON response:**

| CSV column | JSON path |
|------------|-----------|
| `chain` | `data[].chain` |
| `section` | always `"Protocol"` |
| `protocol_id` | `data[].id` |
| `protocol_name` | `data[].name` |
| `protocol_url` | `data[].site_url` |
| `protocol_tvl` | `data[].tvl` |
| `position_type` | `data[].portfolio_item_list[].name` |
| `detail_types` | `data[].portfolio_item_list[].detail_types` joined with `\|` |
| `position_description` | `data[].portfolio_item_list[].detail.description` (may be absent) |
| `position_index` | `data[].portfolio_item_list[].position_index` |
| `adapter_id` | `data[].portfolio_item_list[].pool.adapter_id` |
| `controller` | `data[].portfolio_item_list[].pool.controller` |
| `pool_id` | `data[].portfolio_item_list[].pool.id` |
| `pool_created_at` | `data[].portfolio_item_list[].pool.time_at` (unix timestamp) |
| `asset_usd_value` | `data[].portfolio_item_list[].stats.asset_usd_value` |
| `debt_usd_value` | `data[].portfolio_item_list[].stats.debt_usd_value` |
| `net_usd_value` | `data[].portfolio_item_list[].stats.net_usd_value` |
| `token_address` | `asset_token_list[].id` (`"eth"` = native, no contract) |
| `token_symbol` | `asset_token_list[].symbol` |
| `token_display_symbol` | `asset_token_list[].optimized_symbol` |
| `token_name` | `asset_token_list[].name` |
| `token_amount` | `asset_token_list[].amount` |
| `token_price_usd` | `asset_token_list[].price` |
| `token_value_usd` | `token_amount × token_price_usd` (compute) |
| `token_decimals` | `asset_token_list[].decimals` |
| `token_chain` | `asset_token_list[].chain` |
| `token_verified` | `asset_token_list[].is_verified` |

**Example output** (one row per token):
```
chain,section,protocol_id,protocol_name,protocol_url,protocol_tvl,position_type,detail_types,position_description,position_index,adapter_id,controller,pool_id,pool_created_at,asset_usd_value,debt_usd_value,net_usd_value,token_address,token_symbol,token_display_symbol,token_name,token_amount,token_price_usd,token_value_usd,token_decimals,token_chain,token_verified
base,Protocol,base_hydrexfi,Hydrex,https://www.hydrex.fi/,66680431.66,Liquidity Pool,common,,29949,scribe_v3_liquidity,0xc63e9672f8e93234c73ce954a1d1292e4103ab86,0xc63e9672f8e93234c73ce954a1d1292e4103ab86,1750087429,528374.09,0,528374.09,0x04c0599ae5a44757c0af6f9ec3b93da8976c150a,weETH,weETH,Wrapped eETH,68.20993550,2386.62,162790.02,18,base,true
base,Protocol,base_hydrexfi,Hydrex,https://www.hydrex.fi/,66680431.66,Liquidity Pool,common,,29949,scribe_v3_liquidity,0xc63e9672f8e93234c73ce954a1d1292e4103ab86,0xc63e9672f8e93234c73ce954a1d1292e4103ab86,1750087429,528374.09,0,528374.09,0x4200000000000000000000000000000000000006,WETH,WETH,Wrapped Ether,167.75546436,2179.26,365584.06,18,base,true
eth,Protocol,etherfi,ether.fi,https://www.ether.fi/,4555487713.56,Locked,locked,Withdraw Request,,etherfi_locked,0x7d5706f6ef3f89b3951e23e557cdfbc3239d4e2c,0x7d5706f6ef3f89b3951e23e557cdfbc3239d4e2c,1699344455,357513.01,0,357513.01,eth,ETH,ETH,ETH,164.05248049,2179.26,357513.01,18,eth,true
op,Protocol,op_merkl,Merkl,https://app.merkl.xyz,12857.91,Rewards,common,,,merkl_reward,0x3ef3d8ba38ebe18db133cec108f4d14ce00dd9ae,0x3ef3d8ba38ebe18db133cec108f4d14ce00dd9ae,1676997987,339.07,0,339.07,0x4200000000000000000000000000000000000042,OP,OP,Optimism,2528.51079384,0.1341,339.07,18,op,true
```

### Steps

1. Extract wallet tokens and protocol positions from the page snapshot (both formats).
2. For full format, additionally call `browser_network_requests` filtered to `/portfolio/project_list`, then `browser_network_request` to read the response body JSON.
3. Default to simple format unless the user asks for full/detailed.
4. Write to `debank_{address_short}.csv` (simple) or `debank_{address_short}_full.csv` (full), using the Write tool.
5. Confirm the file path to the user.

### Rules

- Column order: `chain` first, then `section`, then protocol fields, then position fields, then token fields.
- Wrap values containing commas or special characters in double quotes.
- Use raw chain IDs from the API (e.g. `base`, `eth`, `op`) — do not normalize to display names in full format.
- Simple format uses display chain names (e.g. `Base`, `Ethereum`, `Optimism`).
- Leave a field empty rather than guessing if the data is absent.

## Notes

- DeBank loads data progressively via async requests — always wait before snapshotting
- Chain filter tabs (All / Ethereum / Arbitrum / ...) default to "All" — keep it on All for full portfolio
- Some wallets have 100+ tokens; use "Show more" or scroll to capture the full list
- Protocol positions appear in the lower section below the token list
