---
name: Trade on Delphi prediction markets
description: Operate the Gensyn Delphi information (prediction) market as an agent — discover markets, quote prices, buy/sell shares with slippage protection, and redeem winning positions.
api: Delphi API
package: gensyn-delphi-skills
sdk: "@gensyn-ai/gensyn-delphi-sdk"
source: https://github.com/gensyn-ai/gensyn-delphi-skills
operations:
  - health
  - listMarkets
  - getMarket
  - listPositions
  - quoteBuy
  - quoteSell
  - buyShares
  - sellShares
  - redeemPositions
  - liquidate
  - approveToken
---

# Trade on Delphi prediction markets

Gensyn publishes a first-party Agent Skills package, `gensyn-delphi-skills` (built on the
Claude Agent SDK), that wraps the `@gensyn-ai/gensyn-delphi-sdk`. This skill lets an agent
participate in Delphi information markets safely. All operation names below are real
`DelphiClient` methods from the published SDK — none are invented.

## Setup

- Install the SDK: `npm install @gensyn-ai/gensyn-delphi-sdk`.
- Set `DELPHI_API_ACCESS_KEY` (issue a key at `https://api-access.delphi.fyi` for mainnet
  or `https://delphi-api-access.gensyn.ai/` for testnet).
- Provide wallet signing credentials: a private key **or** a Coinbase CDP Server Wallet.
- Choose an environment: mainnet (`https://api.delphi.fyi/`) or testnet
  (`https://delphi-api.gensyn.ai/`). Default to **testnet** for any first run.

## Happy path

1. `health()` — confirm the API is reachable (no auth required).
2. `listMarkets({ skip, limit, ... })` — discover open markets; filter by status, category,
   and liquidity. Use `skip`/`limit` for pagination.
3. `getMarket({ address })` — fetch a single market with live on-chain pricing and implied
   probability.
4. `quoteBuy(...)` / `quoteSell(...)` — simulate a trade (read-only, no gas) to see price and
   slippage BEFORE committing funds.
5. `approveToken(...)` / `ensureTokenApproval(...)` — grant the required ERC-20 allowance.
6. `buyShares(...)` / `sellShares(...)` — execute the on-chain trade with slippage protection.
7. `listPositions({ wallet })` — review portfolio positions across market statuses.
8. `redeemPositions(...)` — claim winnings from settled markets; `liquidate(...)` burns shares
   on expired markets.

## Rules

- Always `quoteBuy`/`quoteSell` first and respect the returned slippage bounds — trades settle
  on-chain and are irreversible.
- On-chain writes are idempotent only by blockchain settlement; there is no REST
  idempotency-key. Do not blindly retry a submitted transaction — check position/tx state first.
- Prefer testnet until a flow is verified; only move to mainnet with real funds deliberately.
- Pagination is offset-based via `skip` + `limit`.
