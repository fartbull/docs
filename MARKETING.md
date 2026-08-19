# FartBull — Marketing Protocol

See [PROTOCOL.md](./PROTOCOL.md) for the authoritative protocol specification. This document covers the marketing mechanism within the Fee Program.

**Chain:** Solana (`mainnet-beta`) · **Token Standard:** SPL Token · **Native Asset:** SOL (lamports)

---

## Overview

The FartBull marketing mechanism enables creators to optionally allocate a portion of net trading fees to a marketing ledger. Marketing funds accrue in the shared Fee Program's per-token PDA accounts and are spent according to community governance.

This system allows for:

- **Optional Marketing Allocation** — 20% of net fees to a marketing ledger (creator can disable)
- **Initial $299 USD auto-reserved for DEX Screener Token Enhancement**
- **Community governance** spend proposals after the initial allocation
- **Transparent Tracking** — All balances and proposals publicly verifiable on Solana

---

## Fee Integration

### Fee Architecture

Marketing is **optional**. See [FEE_PROGRAM.md](./FEE_PROGRAM.md) for full details.

```
Gross Trade Fee (lamports)
↓
Protocol Share (protocol revenue, taken first)
↓
Net Fee
↓
Marketing Enabled? → Yes: 20% to Marketing Ledger, 80% to Destination Ledger
                  No: 100% to Destination Ledger
```

The 20% marketing split is calculated only on the remaining amount after the protocol share. The protocol share is taken first and is unrelated to the 200,000,000 token supply allocation.

### Marketing Configuration

| Parameter | Range | Description |
|-----------|-------|-------------|
| `marketing_enabled` | true/false | Whether marketing split is active |
| Marketing split | Fixed 20% of net fee | 20% of the remaining amount after protocol revenue |

When `marketing_enabled = false`, 100% of the net fee goes to the destination ledger.

---

## Marketing Ledger

### Purpose

The marketing ledger in the Fee Program's per-token PDA holds accumulated marketing SOL (lamports). Funds remain on-chain until claimed or spent via governance.

### Initial Mandatory Spend

When marketing is enabled, the first $299 USD equivalent of marketing funds is automatically reserved for **DEX Screener Token Enhancement**. This:

- Requires no governance vote
- Executes automatically via protocol logic
- Is the protocol's mandatory first marketing action

After the initial $299 enhancement completes, marketing funds become subject to governance proposals.

> **Note:** DEX Screener is an off-chain analytics and visibility service. The threshold is an off-chain execution milestone triggered by on-chain accounting in the Fee Program PDA. Do not pretend DEX Screener is a Solana on-chain program.

---

## Governance

### Marketing Proposals

Top 100 holders vote on marketing spend proposals for each token. Voting is snapshot-based (balances frozen at proposal creation). See [GOVERNANCE.md](./GOVERNANCE.md) for full voting mechanics.

Marketing proposals are **per-token**, never global.

### Eligible Campaign Categories

| Category | Description | Governance Required |
|----------|-------------|---------------------|
| DEX Screener Boost | Token visibility enhancement | Yes |
| Additional DEX Screener Services | Further DEX tools | Yes |
| Social Media Campaign | Social platform promotion | Yes |
| Exchange Listing Application | Apply to exchanges | Yes |
| Influencer Campaign | Creator/affiliate payments | Yes |
| Community Promotion | User distribution | Yes |

Do not imply that every category is automatically integrated on-chain. If an action requires an external service:

```
On-chain governance approves expenditure
+
external execution mechanism performs the service
```

Keep that boundary clear.

### Current Status

The first $299 USD equivalent of marketing funds (when enabled) is automatically reserved for **DEX Screener Token Enhancement** without governance. After this initial spend, marketing proposals require governance approval.

---

## Claims

Marketing funds are claimed from the Fee Program PDA, not automatically distributed:

```
claim_marketing instruction:
    verify caller authorized via governance
    amount = fee_config_pda.marketing_balance
    fee_config_pda.marketing_balance = 0
    system_program.transfer(fee_config_pda, caller, amount)
```

Governance-executed proposals may trigger spending from the marketing ledger for approved campaigns. All spending requires prior governance approval.

---

## Key Differences from Destination Ledger

| Aspect | Destination Ledger | Marketing Ledger |
|--------|--------------------|------------------|
| Distribution | Manual claim by recipient | Governance-controlled spend |
| Auto-transfer | No | No |
| Governance | No | Yes (spend proposals) |
| Initial mandatory spend | No | Yes ($299 DEX Screener) |

---

## DEX / Marketing Integration Points

The marketing system interacts with the following external services:

| Service | Purpose | Status |
|---------|---------|--------|
| DEX Screener | Token visibility enhancement | Integration point (requires off-chain execution) |
| Social platforms | Campaign execution | Integration point |
| Exchange listing | Application and payment | Integration point |

These are **integration points**, not on-chain Solana programs. The protocol approves expenditure on-chain via governance; external execution is handled by off-chain services coordinated by the API layer (`api.fartbull.xyz`).

---

*Marketing protocol documentation last updated: August 2026*
*Next review: October 2026*
*Source of truth: [PROTOCOL.md](./PROTOCOL.md)*
