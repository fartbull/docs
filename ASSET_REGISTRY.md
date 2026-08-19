# FartBull — Asset Registry

See [PROTOCOL.md](./PROTOCOL.md) for the authoritative protocol specification.

> **Status:** Conceptual architecture / future module. Not yet implemented.

**Chain:** Solana (`mainnet-beta`) · **Token Standard:** SPL Token · **Native Asset:** SOL (lamports)

---

## Overview

The **Asset Registry** is a planned abstraction for supported assets that may eventually be used by FartBull launch mechanisms.

**DO NOT claim that FartBull owns or issues real-world securities.**
**DO NOT claim that FartBull provides legal ownership of stocks.**
**DO NOT invent custody arrangements.**

The registry can describe supported assets that may eventually be used by FartBull launch mechanisms. The exact mechanism is **NOT yet finalized**.

---

## Architecture

Conceptually:

```
Asset Registry
    |
    v
Asset ID
    |
    v
Asset Type
    |
    v
 Underlying Reference
    |
    v
 Supported Network
    |
    v
 Oracle / Data Source
    |
    v
 Status
    |
    v
 Distribution Configuration
```

### Components

| Component | Description | Status |
|-----------|-------------|--------|
| **Asset selection** | Choosing supported assets for future launch configuration | Design |
| **Asset registry** | On-chain registry of supported assets | Design |
| **Price/data oracle abstraction** | Oracle interface, provider TBD | `TBD — orcale provider` |
| **Holder accounting** | Tracking token holder positions | `TBD` |
| **Distribution accounting** | Tracking distribution obligations | `TBD` |
| **Supported chains** | Cross-chain asset reference model | `TBD — cross-chain model` |
| **Settlement mechanism** | TBD | `TBD — settlement model` |

---

## Asset Identity

### Asset ID

A unique identifier for each registered asset. The exact format (string, hash, on-chain PDA address) is `TBD — asset ID format`.

### Asset Type

The category of the underlying asset:

| Type | Description | Status |
|------|-------------|--------|
| **Crypto Asset** | Native or tokenized cryptocurrency (e.g. SOL, BTC) | Design |
| **Tokenized Asset** | An SPL token representing a share in an underlying | Design |
| **Stock-Linked** | A token with economic behavior tied to a stock price | `FUTURE / DESIGN REQUIRED` |
| **Commodity** | A token tied to a commodity price | `FUTURE` |
| **Index** | A token tied to an index of assets | `FUTURE` |

### Underlying Reference

A reference to the underlying asset (ticker, contract address, ISIN, etc.). The exact format is `TBD`.

### Supported Network

The blockchain/network where the underlying asset lives. The core launchpad is Solana-first, but the Asset Registry is designed to reference assets from other networks as well:

| Network | Purpose | Status |
|---------|---------|--------|
| **Solana** | Native SPL token assets | Design |
| **External network** | Potential tokenized asset reference | `FUTURE — cross-chain` |
| **Base** | Potential tokenized asset reference | `FUTURE — cross-chain` |
| **BNB Chain** | Potential tokenized asset reference | `FUTURE — cross-chain` |

### Oracle / Data Source

The oracle abstraction defines how price data is sourced:

```
Asset Registry PDA
    |
    v
Oracle/Data Source (TBD)
    |
    v
Price Feed
    |
    v
On-Chain Price (in PDA state)
```

> **Open:** Oracle provider is TBD. See [TODO.md](./TODO.md). Do not invent a specific oracle provider.

### Status

Each asset has an on-chain status:

| Status | Description |
|--------|-------------|
| `Active` | Asset is available for new launches |
| `Paused` | Asset is temporarily unavailable |
| `Delisted` | Asset is no longer available |

### Distribution Configuration

Configuration for how asset-linked yields or distributions flow to token holders:

> **Open:** Distribution accounting model is TBD. See [TODO.md](./TODO.md).

---

## Required Distinctions

The documentation must distinguish between three concepts that are **not** automatically the same:

| Concept | Meaning |
|---------|---------|
| **Price Exposure** | Economic behavior tied to an asset's price (what the token does) |
| **Tokenized Real-World Asset** | A token that represents a claim on an underlying asset (what it might represent) |
| **Legal Ownership** | Actual legal title to the underlying asset (who legally owns it) |

These three may exist independently:

- A token can have **price exposure** to a stock without being a **tokenized real-world asset**
- A **tokenized real-world asset** can exist without conferring **legal ownership** to token holders
- **Legal ownership** requires explicit custody and compliance arrangements that are **not** implied by the protocol

If the existing documentation does not define the legal/custody model, it is marked as:

```
OPEN DESIGN / LEGAL & COMPLIANCE REQUIREMENT
```

---

## Stock-Linked Token Concept

### Document Status

Document the user's intended future concept:

Users may eventually be able to select approved assets/stocks when launching a token. The token may have economic behavior tied to an approved asset configuration. However, the exact mechanism is **NOT yet finalized**.

Therefore document the following architectural components:

- **Asset selection** — choosing from the Asset Registry
- **Asset registry** — on-chain registry of supported assets
- **Price/data oracle abstraction** — oracle interface (provider TBD)
- **Holder accounting** — tracking token holder positions
- **Distribution accounting** — tracking distribution obligations
- **Supported chains** — cross-chain asset reference
- **Settlement mechanism** — TBD

### What is NOT Finalized

- The economic model linking token price to asset price
- The legal structure conferring rights to holders
- The custody arrangement for underlying assets
- The oracle provider and data source
- The distribution mechanism for yields/dividends
- The settlement timeline and mechanism

### Status Markers

Mark the stock-linked token concept as:

```
FUTURE MODULE / DESIGN REQUIRED
```

---

## Proposed Fee Integration (Planned)

When stock pairing is enabled for a token, a configurable percentage of the net fee (after protocol revenue and before the destination split) would be diverted to a stock purchase fund.

This is an **additional** allocation, not a replacement for the standard fee routing. The destination ledger and marketing ledger continue to operate as described in [FEE_PROGRAM.md](./FEE_PROGRAM.md).

### Proposed Allocation

| Allocation | Destination |
|------------|-------------|
| Protocol Share | Fee Program protocol ledger |
| Stock Allocation | Stock purchase vault PDA (configurable %) |
| Marketing | Marketing ledger (optional) |
| Creator Destination | Destination ledger |

> **Open:** This is a proposed design, not finalized. The exact percentage, vault mechanism, and custody model are TBD.

---

## Proposed Yield Distribution (Planned)

Yield from holdings would be distributed to token holders through the Fee Program:

1. Yield received by the vault PDA
2. Value converted to SOL or a stablecoin (SPL token)
3. Recorded as a yield ledger entry in the Fee Program PDA
4. Holders claim via the standard `claim_destination` mechanism

This maintains consistency with the claim-based distribution model used across the protocol.

> **Open:** The yield distribution mechanism is TBD. Requires oracle, custody, and legal compliance design.

---

## Risk Management (Planned)

| Risk | Mitigation |
|------|-----------|
| Market Risk | Diversification across assets |
| Yield Risk | Escrow reserves for missed yield |
| Liquidity Risk | Minimum liquidity requirements on asset tokens |
| Regulatory Risk | Compliance monitoring and screening |

> **Open:** Legal and compliance framework for real-world asset exposure is `OPEN DESIGN / LEGAL & COMPLIANCE REQUIREMENT`.

---

## Conclusion

The Asset Registry is a future module that would add real-world asset exposure to the FartBull ecosystem on Solana. The core protocol — token creation, bonding curve trading, fee distribution, and liquidity migration — does not depend on this module.

Implementation of the Asset Registry and stock-linked token concept will be documented in future protocol updates once available.

---

*Asset registry documentation last updated: August 2026*
*Next review: October 2026*
*Source of truth: [PROTOCOL.md](./PROTOCOL.md)*
