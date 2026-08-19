# FartBull — API Reference

See [PROTOCOL.md](./PROTOCOL.md) for the authoritative protocol specification. This document provides programmatic access specifications for the FartBull API.

**Chain:** Solana (`mainnet-beta`) · **Token Standard:** SPL Token · **Native Asset:** SOL (lamports)

**Frontend:** `https://fartbull.xyz` | **API:** `https://api.fartbull.xyz`

---

## Table of Contents

1. [Overview](#overview)
2. [Rate Limits](#rate-limits)
3. [Error Handling](#error-handling)
4. [Endpoints](#endpoints)
5. [WebSocket API](#websocket-api)
6. [On-Chain vs Off-Chain Authority](#on-chain-vs-off-chain-authority)

---

## Overview

The FartBull API provides access to protocol data: token information, curve pricing, trade execution, fee accounting, governance, and social identity.

### API Version

Current Version: v1
Base Path: `/v1`
Base URL: `https://api.fartbull.xyz`

### Supported Formats

- **Request/Response:** JSON
- **Numbers:** Decimal strings for precision
- **Dates:** ISO 8601 format
- **Solana addresses:** Base58-encoded Pubkeys

### Program Sources

The API surfaces data from the canonical Solana programs: `Token Factory Program`, `Bonding Curve Program`, `Fee Program`, `Migration Program`, `Social Registry Program`, and `Governance Program`.

> **Important:** The API is **not** the ultimate authority for on-chain state. The Solana programs remain authoritative for protocol state and protocol-enforced permissions. The API provides indexing, convenience, and off-chain services but cannot override on-chain rules. Users can always verify state directly via Solana RPC or Solscan.

---

## Rate Limits

Rate limits are applied per API key. The following headers are returned on every request:

```
X-RateLimit-Limit: 30000
X-RateLimit-Remaining: 24850
X-RateLimit-Reset: 1692100060
```

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum requests per window |
| `X-RateLimit-Remaining` | Requests remaining |
| `X-RateLimit-Reset` | Unix timestamp when limit resets |

### Handling Rate Limits

When rate limit exceeded:

```json
{
  "type": "rate-limit",
  "title": "Rate Limit Exceeded",
  "status": 429,
  "detail": "Too many requests."
}
```

---

## Error Handling

### Error Response Format

All errors follow RFC 7807 Problem Details format:

```json
{
  "type": "validation-error",
  "title": "Bad Request",
  "status": 400,
  "detail": "Invalid mint address format",
  "errors": [
    {
      "field": "mint",
      "message": "Must be a valid Solana Pubkey (base58, 32 bytes)"
    }
  ]
}
```

### HTTP Status Codes

| Code | Title | Description |
|------|-------|-------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created |
| 400 | Bad Request | Invalid parameters |
| 401 | Unauthorized | Invalid API key |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |
| 503 | Service Unavailable | Maintenance/outage |

### Error Types

| Type | Code | Description |
|------|------|-------------|
| `validation-error` | 400 | Input validation failed |
| `unauthorized` | 401 | Invalid credentials |
| `forbidden` | 403 | Access denied |
| `not-found` | 404 | Resource doesn't exist |
| `rate-limit` | 429 | Too many requests |
| `internal-error` | 500 | Server-side error |

---

## Endpoints

### GET /token/info

Get token metadata and current state.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mint` | string | Yes | SPL Token Mint address (base58 Pubkey) |

**Response:**

```json
{
  "mint": "TBD",
  "name": "TBD",
  "symbol": "TBD",
  "decimals": 9,
  "totalSupply": "1000000000000000000",
  "curveSupply": "800000000000000000",
  "migrationSupply": "200000000000000000",
  "status": "active",
  "chain": "Solana",
  "currentPriceSOL": "0.00015",
  "marketCapUSD": "1500000",
  "holders": 12500
}
```

> Token amounts are in **base units** (1 token = 10^decimals base units). The `totalSupply` field reflects base units, not human-readable tokens.

### GET /token/{mint}/holders

Get top token holders.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mint` | string | Yes | SPL Token Mint (base58 Pubkey, path) |
| `limit` | integer | No | Max results (default: 100, max: 1000) |

**Response:**

```json
{
  "mint": "TBD",
  "totalHolders": 12543,
  "topHolders": [
    {
      "rank": 1,
      "owner": "TBD",
      "tokenAccount": "TBD",
      "balance": "200000000000000000",
      "percentage": "20.00%"
    }
  ]
}
```

> Holder balances are resolved from SPL **token accounts**. The `owner` field is the wallet address; the `tokenAccount` is the specific ATA holding the balance. The API aggregates balances across all token accounts per owner.

### GET /curve/price

Get current bonding curve price.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mint` | string | Yes | SPL Token Mint (base58 Pubkey) |
| `amount` | integer | No | Tokens to buy in base units (for cost estimate) |

**Response:**

```json
{
  "mint": "TBD",
  "currentPriceSOL": "0.00015",
  "soldTokens": "500000000",
  "totalSupply": "1000000000",
  "availableTokens": "500000000",
  "estimatedSpotSOL": "75000"
}
```

> `currentPriceSOL` is in human-readable SOL. `soldTokens` and `availableTokens` are in human-readable tokens. `estimatedSpotSOL` is the SOL value of all tokens at the curve at the current price.

### POST /trade/buy

Calculate and execute token purchase via the Bonding Curve Program.

**Request Body:**

```json
{
  "mint": "TBD",
  "amount": "1000000000",
  "minTokensOut": "950000000",
  "maxSOLIn": "0.15"
}
```

**Response:**

```json
{
  "success": true,
  "signature": "TBD",
  "tokensReceived": "1000000000",
  "costSOL": "0.150000000",
  "feeSOL": "0.003000000",
  "computeUnits": 120000
}
```

> `signature` is the Solana transaction signature (base58). `costSOL` and `feeSOL` are in human-readable SOL. `computeUnits` reflects the compute budget consumed.

### POST /trade/sell

Calculate and execute token sale back to the curve (pre-migration).

**Request Body:**

```json
{
  "mint": "TBD",
  "amount": "1000000000",
  "minSOLPayable": "0.14"
}
```

**Response:**

```json
{
  "success": true,
  "signature": "TBD",
  "solReceived": "0.147000000",
  "feeSOL": "0.003000000",
  "computeUnits": 110000
}
```

### GET /fee/config

Get current fee configuration for a token.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mint` | string | Yes | SPL Token Mint (base58 Pubkey) |

**Response:**

```json
{
  "mint": "TBD",
  "feeBps": 200,
  "platformFeeBps": 200,
  "destination": "TBD",
  "marketingEnabled": true,
  "feeProgram": "TBD"
}
```

### GET /fee/ledger

Get Fee Program PDA balances for a token.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mint` | string | Yes | SPL Token Mint (base58 Pubkey) |

**Response:**

```json
{
  "mint": "TBD",
  "protocolBalanceSOL": "100.0",
  "marketingBalanceSOL": "20.0",
  "destinationBalanceSOL": "80.0",
  "totalClaimedSOL": "500.0"
}
```

> All balances are in human-readable SOL. The underlying PDA stores lamports.

### POST /fee/claim

Claim accumulated destination or protocol fees from the Fee Program.

**Request Body:**

```json
{
  "mint": "TBD",
  "claimType": "destination"
}
```

### GET /migration/status

Get migration status for a token.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mint` | string | Yes | SPL Token Mint (base58 Pubkey) |

**Response:**

```json
{
  "mint": "TBD",
  "graduated": false,
  "migrationCondition": "TBD",
  "solBalance": "75000.0",
  "tokenBalance": "800000000",
  "dex": "Solana DEX (TBD)"
}
```

### GET /governance/proposals

List governance proposals for a token.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mint` | string | Yes | SPL Token Mint (base58 Pubkey) |

**Response:**

```json
{
  "mint": "TBD",
  "proposals": [
    {
      "id": "1",
      "type": "marketing",
      "title": "DEX Screener Boost",
      "status": "active",
      "votesFor": "120000000",
      "votesAgainst": "30000000"
    }
  ]
}
```

### POST /governance/vote

Cast a vote on a governance proposal.

**Request Body:**

```json
{
  "mint": "TBD",
  "proposalId": "1",
  "support": true
}
```

### GET /social/identities

List verified social identities registered as fee destinations.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mint` | string | Yes | SPL Token Mint (base58 Pubkey) |

### GET /agents

List agents managed by the FartBull Program.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `agentAddress` | string | No | Filter by agent PDA address |

**Response:**

```json
{
  "agents": [
    {
      "address": "TBD",
      "status": "active",
      "permissions": ["interact_curve"],
      "associatedLaunches": []
    }
  ]
}
```

### GET /assets

List assets in the Asset Registry.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `assetId` | string | No | Filter by asset ID |

**Response:**

```json
{
  "assets": [
    {
      "assetId": "TBD",
      "type": "crypto",
      "underlying": "SOL",
      "network": "Solana",
      "status": "active"
    }
  ]
}
```

---

## WebSocket API

### Connection

Connect to the WebSocket endpoint to receive real-time event updates.

### Event Types

| Event | Description | Payload |
|-------|-------------|---------|
| `trade.executed` | Trade completed | `{mint, buyer, amount, price, slot, signature}` |
| `fee.collected` | Fee payment | `{mint, amount, destination}` |
| `proposal.created` | New governance proposal | `{id, title, proposer}` |
| `vote.cast` | Vote recorded | `{proposalId, voter, support}` |
| `claim.executed` | Fee claimed | `{mint, claimant, amount}` |
| `migration.completed` | Migration to Solana DEX | `{mint, liquidityPosition, graduated, signature}` |
| `agent.action` | Agent performed action | `{agentAddress, action, targetMint, signature}` |
| `asset.updated` | Asset registry updated | `{assetId, status}` |

### Subscription Example

```javascript
const ws = new WebSocket('wss://api.fartbull.xyz/v1/ws');

ws.onopen = () => {
  ws.send(JSON.stringify({
    type: 'subscribe',
    topic: 'trades',
    filter: { mint: 'TBD' }
  }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Trade:', data);
};
```

---

## Solana Integration

The API bridges to Solana RPC for on-chain data. Key Solana concepts used in API responses:

| API Field | Solana Equivalent |
|-----------|-------------------|
| `signature` | Transaction signature (base58) |
| `slot` | Slot number for ordering |
| `mint` | SPL Token Mint address (base58 Pubkey) |
| `tokenAccount` | SPL Token Account / ATA |
| `owner` | Wallet address (base58 Pubkey) |
| `computeUnits` | Compute units consumed |
| `feeSOL` | Transaction fee + priority fee (lamports) |

---

## On-Chain vs Off-Chain Authority

The API provides the following services:

- **Authentication** — user/session management via the frontend
- **Agent orchestration** — off-chain agent logic and scheduling
- **Social integrations** — OAuth verification and proof generation
- **Indexing** — fast read access to on-chain state
- **Notifications** — real-time event delivery
- **Off-chain metadata** — token metadata, descriptions, images
- **Automation** — automated agent actions within permission bounds
- **Market/data services** — price feeds, charts, analytics

But the API must **not** be treated as the ultimate authority for on-chain state. The Solana programs remain authoritative for protocol state and protocol-enforced permissions.

If an API response disagrees with on-chain state, the on-chain state is correct. The API is a convenience layer; the programs are the source of truth.

---

## Best Practices

### Query Optimization

1. **Batch Requests:** Use batch endpoints for multiple queries
2. **Caching:** Cache frequently accessed data
3. **Pagination:** Use cursor-based pagination for large datasets
4. **Filtering:** Apply filters server-side to reduce transfer

### Error Handling

Always handle errors gracefully:

```javascript
try {
  const result = await fetch('https://api.fartbull.xyz/v1/trade/buy', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(params)
  });

  if (result.status === 429) {
    // Rate limited — wait and retry
    console.log('Rate limited, retrying...');
  } else if (result.status >= 400) {
    // Handle error
    const error = await result.json();
    console.log('Error:', error.detail);
  }
} catch (err) {
  console.error('Network error:', err);
}
```

### State Verification

Always verify critical state directly against Solana RPC when necessary:

```javascript
// Verify API response against on-chain state
const onChainBalance = await connection.getBalance(new PublicKey(wallet));
if (onChainBalance !== apiResponse.balance) {
  console.warn('API response disagrees with on-chain state');
}
```

---

*API reference documentation last updated: August 2026*
*Next review: October 2026*
*Source of truth: [PROTOCOL.md](./PROTOCOL.md)*
