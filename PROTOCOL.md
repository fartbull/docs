# FartBull — Protocol Specification

Source of truth for the FartBull protocol architecture. All other documentation references this document.

**Project:** FartBull
**Chain:** Solana (`mainnet-beta`) · **Token Standard:** SPL Token · **Native Asset:** SOL (lamports)
**Primary Token (CA):** `4uuxPfEdy2ZHhgux9zrRHtsbVx9yrnDrtLLAPWhmdKSE`
**Website:** `https://fartbull.xyz` · **X:** `https://x.com/fartbullssol` · **API:** `https://api.fartbull.xyz`

---

## Network Reference

```text
Network:        Solana (mainnet-beta)
Native Asset:   SOL (1 SOL = 1,000,000,000 lamports)
RPC:            TBD
Explorer:       https://solscan.io
Token CA:       4uuxPfEdy2ZHhgux9zrRHtsbVx9yrnDrtLLAPWhmdKSE
```

FartBull is built natively on Solana. Programs are deployed as executable BPF accounts, state is stored in separate accounts (including PDAs), and transactions carry instructions that invoke programs via CPIs.

---

## Table of Contents

1. [Protocol Philosophy](#protocol-philosophy)
2. [Launch Lifecycle](#launch-lifecycle)
3. [Program Architecture](#program-architecture)
4. [Conceptual Modules](#conceptual-modules)
5. [Fee Architecture](#fee-architecture)
6. [Migration Lifecycle](#migration-lifecycle)
7. [Agent System](#agent-system)
8. [Asset Registry](#asset-registry)
9. [Governance Scope](#governance-scope)
10. [Social Registry](#social-registry)
11. [Protocol Invariants](#protocol-invariants)
12. [Module Connections](#module-connections)

---

## Protocol Philosophy

FartBull is a **permissionless bonding-curve launchpad protocol on Solana**. It enables token creators to launch SPL tokens with:

1. **Deterministic price discovery** via a linear bonding curve
2. **Single destination fee routing** — creators choose one primary fee destination, tracked in program state and claimed on demand
3. **Optional marketing ledger** — a protocol-level mechanism for community-governed marketing spend
4. **Automated liquidity migration** — when the curve completes, liquidity migrates to a Solana DEX
5. **Social identity verification** — verified social accounts enable directed reward claims
6. **Optional asset pairing** — an optional module that uses a portion of fees to purchase real-world asset exposures
7. **Agent System** — autonomous on-chain agents whose protocol identity is represented by a Solana PDA

Core design principles:

- **Non-custodial:** Users control their own wallets and private keys at all times.
- **Fair launch:** No pre-allocated supply, no team tokens, no presale.
- **Transparency:** All protocol state is on-chain and publicly verifiable via Solana accounts.
- **Composability:** Shared protocol programs reduce deployment overhead and compute costs through CPI.
- **Least privilege:** Agents and external integrations are granted narrow, program-enforced capabilities.

---

## Launch Lifecycle

### Phase 1: Token Creation

A creator invokes the **Token Factory Program**. The factory creates:

1. **SPL Token Mint** — a mint account created via the SPL Token Program, with the full supply minted to the creator's associated token account
2. **Bonding Curve State PDA** — a program-owned state account initialized with the token mint, curve parameters, and fee configuration

The bonding curve is active for trading immediately after creation.

### Phase 2: Trading

Users buy tokens from the bonding curve using SOL. Price follows the linear formula:

```
Price(s) = basePrice + slope × sold
```

Prices and costs are expressed in **lamports** (the smallest unit of SOL). Each trade incurs a fee. Fees are credited to the Fee Program's per-token accounting PDA.

### Phase 3: Migration

When the bonding curve reaches its configured migration condition (TBD), the **Migration Program** migrates liquidity to a Solana DEX and trading moves to the DEX. The Fee Program continues processing fees from the new fee source.

---

## Program Architecture

FartBull uses **shared protocol programs** for cross-token services and **per-token state** for launch-specific data. Mutable state is stored in program-owned PDA accounts.

```
Frontend
    |
    v
Solana RPC
    |
    v
FartBull Programs
    |
    +---- Token Mint (via SPL Token Program)
    |
    +---- Bonding Curve State PDA
    |
    +---- Fee Config PDA
    |
    +---- Agent PDAs
    |
    +---- Asset Registry PDAs
    |
    +---- Shared Fee Program
    |
    +---- Migration Program
    |
    +---- Social Registry Program
    |
    +---- Governance Program
    |
    v
Solana
```

The protocol is conceptually organized into eight modules (detailed in [ARCHITECTURE.md](./ARCHITECTURE.md)):

1. **Token Launch** — Token Factory Program
2. **Bonding Curve** — Bonding Curve Program + State PDA
3. **Fee Management** — Fee Program + per-token PDAs
4. **Migration** — Migration Program + LP position handling
5. **Governance** — Governance Program + snapshot voting
6. **Agent System** — Agent PDA + permissions + automation
7. **Asset Registry** — Asset registry PDA + supported assets
8. **Social Registry** — Social Registry Program + identity PDAs

### Shared Protocol Programs

| Program | Count | Purpose |
|---------|-------|---------|
| **Token Factory Program** | 1 | Entry point for creating token mints and initializing curve state |
| **Fee Program** | 1 | Central fee collection and routing for all tokens |
| **Migration Program** | 1 | Handles migration from bonding curve to Solana DEX |
| **Social Registry Program** | 1 | Verifies social identities and tracks eligible claim destinations |
| **Governance Program** | 1 | Snapshot-based top-100 voting for marketing and CTO recovery |

### Per-Token State

| Account | Count | Purpose |
|---------|-------|---------|
| **SPL Token Mint** | Per launch | Token definition via the SPL Token Program |
| **Bonding Curve State PDA** | Per launch | Linear price-discovery state; frozen after migration |
| **Fee Config PDA** | Per launch | Per-token fee accounting and routing |
| **Agent PDA** | Per agent | Agent identity and permissions |
| **Asset PDA** | Per registered asset | Asset metadata and oracle reference |

### Key Architectural Decision

There is **one** Fee Program for the entire protocol. It does **not** deploy per-token fee programs. It maintains isolated per-token accounting state in separate PDA accounts. Similarly, there is one Migration Program, one Social Registry Program, one Governance Program, and one Token Factory Program.

Per-token accounts (Token Mint, Bonding Curve State PDA, Fee Config PDA) are initialized fresh for each launch via the Token Factory Program. Agent and Asset PDAs are initialized as needed.

### Program IDs

Program IDs are TBD and will be published at deployment:

```text
Token Factory Program:     TBD
Fee Program:               TBD
Migration Program:         TBD
Social Registry Program:   TBD
Governance Program:        TBD
Agent Program:             TBD
Asset Registry Program:    TBD
```

Do not fabricate program addresses. All program deployments are TBD.

---

## Fee Architecture

### Fee Collection

Every trade — whether on the Bonding Curve (pre-migration) or on the Solana DEX (post-migration) — generates a fee. The fee is credited to the **Shared Fee Program**'s per-token accounting PDA.

### Fee Program Accounting

The Fee Program maintains fully isolated balances per token. Each token's accounting lives in a dedicated PDA state account — no cross-token accounting occurs.

### Per-Token State Account (PDA)

For each token, the Fee Program tracks in a dedicated state PDA:

| Field | Type | Description |
|-------|------|-------------|
| `protocol_balance` | u64 (lamports) | Protocol revenue share |
| `marketing_balance` | u64 (lamports) | Marketing ledger funds |
| `destination_balance` | u64 (lamports) | Creator-selected destination funds |
| `claimed_balance` | u64 (lamports) | Tracking of claimed amounts |
| `marketing_enabled` | bool | Whether marketing split is active |
| `destination` | Pubkey | Claim destination (wallet or verified social identity) |

No mappings, no shared mutable storage — each token has its own PDA account. The accounting invariant:

```
fees received = protocol allocation + marketing allocation + destination allocation + legitimately claimed amounts
```

The accounting for Token A must never affect Token B. This is a core protocol security invariant.

### Fee Flow

```
Trade
  |
  v
Fee Calculated
  |
  v
Fee Program receives 100%
  |
  v
Protocol Share (taken first)
  |
  v
Remaining Creator Fee
  |
  +--------------------------+
  |                          |
Marketing Enabled?            No
  |                          |
 YES                         |
  |                          |
  v                          v
20% Marketing              100% Destination
80% Destination
  |
  v
Recipient calls claim()
```

The platform/protocol share is taken **before** the optional marketing split. This distinction must be consistent everywhere.

### Platform Revenue

```
Gross Trading Fee
        |
        v
Protocol Share
        |
        v
Remaining Creator Allocation
        |
        +---- 80% Destination
        |
        +---- 20% Marketing
```

The platform fee is a protocol-level percentage deducted from every gross trade fee. It is deducted **before** the marketing split. The documented default is `200` basis points (2%). This value is independent of the 200,000,000 token supply allocation.

The 20/80 marketing split is calculated only on the remaining amount after the platform share. Do **not** calculate marketing as 20% of the gross trade fee.

### Marketing

Marketing is **optional**. Creators may enable or disable it per token at launch.

If **disabled**: remaining creator fee → 100% Destination Balance.

If **enabled**: remaining creator fee → 80% Destination Balance, 20% Marketing Balance.

Marketing funds do **not** automatically transfer to any wallet. Recipients claim them manually. The 20% marketing split is unrelated to the 200,000,000 token supply allocation.

### Claims

The destination does **not** receive funds automatically. The Fee Program maintains the destination balance in a PDA account. The recipient triggers a claim instruction to withdraw accumulated SOL.

```
Destination Balance (PDA)
       |
       v
claim instruction
       |
       v
Authorization Check (signer validated)
       |
       v
Funds Released (SOL transferred to claimant)
```

The same principle applies to social destinations where appropriate. The protocol balance is claimable by the authorized protocol admin. Marketing funds are claimed or spent via governance.

Fees accumulate on-chain in PDA-controlled vault accounts; they are never pushed automatically to any recipient.

### Initial Mandatory Marketing Spend

When marketing is enabled, the first $299 USD equivalent of marketing funds is automatically reserved for **DEX Screener Token Enhancement**. This is automatic — there is no governance vote for the initial action. Once the initial action completes, future marketing spending is governed.

---

## Migration Lifecycle

Migration changes **where fees originate**, not how they are processed.

| Phase | Fee Source | Fee Destination |
|-------|-----------|-----------------|
| Pre-migration | Bonding Curve trades | Fee Program |
| Post-migration | Solana DEX trading fees | Fee Program |

The Fee Program, its accounting, and its routing logic remain identical throughout. The Bonding Curve simply stops forwarding fees after migration.

### Migration Flow

```
Bonding Curve
      |
      v
Migration Condition Reached
      |
      v
Migration Program
      |
      v
Prepare Migration Assets
      |
      v
Solana DEX / AMM
      |
      v
Liquidity Created
      |
      v
Bonding Curve Frozen
      |
      v
DEX Trading Begins
```

The exact migration threshold (supply percentage, SOL target, slot deadline) is TBD. The migration flow above is the canonical architecture regardless of the specific trigger values.

---

## Agent System

> **Status:** Conceptual architecture. Implementation is a future phase.

FartBull can create and manage autonomous agents whose protocol identity is represented by a **Solana PDA**. The Agent PDA is managed and enforced by the FartBull program — the website/API must **not** simply create an unrestricted private key and call that an agent.

```
FartBull Program
      |
      v
Agent PDA
      |
      v
Agent Configuration
      |
      v
Permissions
      |
      v
Supported Actions
      |
      v
Associated Launches / Assets / Social Identity
```

### Intended Concepts

| Concept | Description |
|---------|-------------|
| **Agent PDA** | A program-owned PDA representing the agent's on-chain identity |
| **Agent identity** | The binding between the agent and its allowed creator/on-chain actions |
| **Agent configuration** | Parameters defining the agent's behavior envelope |
| **Agent permissions** | Narrow, program-enforced capabilities (least privilege) |
| **Agent status** | Active / paused / revoked |
| **Agent associated launches** | Tokens this agent is permitted to interact with |
| **Agent treasury references** | Treasury PDAs controlled by the agent's authority |
| **Agent social identity** | Verified social identity linkage (via Social Registry) |
| **Agent automation configuration** | Off-chain execution parameters for automated actions |

### Agent Security Model

The documentation explicitly separates **AI/automation logic** (off-chain API) from **on-chain authority** (program-enforced). The agent does **not** automatically gain unrestricted protocol authority.

Potential permissions may include:
- Launch token
- Interact with approved bonding curves
- Execute predefined actions
- Interact with approved assets
- Publish through approved social integrations

Potentially restricted:
- Arbitrary treasury withdrawal
- Protocol upgrade
- Governance bypass
- Arbitrary program configuration
- Arbitrary asset creation

> **Open:** Exact Agent PDA seeds and permission framework are TBD. See [AGENT_SYSTEM.md](./AGENT_SYSTEM.md) and [TODO.md](./TODO.md).

---

## Asset Registry

> **Status:** Conceptual architecture. Future module.

The Asset Registry is a planned abstraction for supported assets that may eventually be used by FartBull launch mechanisms. It does **not** claim that FartBull owns or issues real-world securities, provides legal ownership of stocks, or operates any custody arrangement.

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

### Architecture Components

- **Asset selection** — choosing supported assets for future launch configuration
- **Asset registry** — on-chain registry of supported assets
- **Price/data oracle abstraction** — oracle interface, provider TBD
- **Holder accounting** — tracking token holder positions
- **Distribution accounting** — tracking distribution obligations
- **Supported chains** — cross-chain asset reference model
- **Settlement mechanism** — TBD

The architecture allows future support for:
- Crypto assets
- Tokenized assets
- Stock-linked products
- Other supported assets

### Required Distinctions

The documentation distinguishes between:

| Concept | Meaning |
|---------|---------|
| **Price exposure** | Economic behavior tied to an asset's price |
| **Tokenized real-world asset** | A token that represents a claim on an underlying asset |
| **Legal ownership** | Actual legal title to the underlying asset |

These are **not** automatically the same thing.

> **Open:** Asset custody model, legal/compliance framework, oracle provider. See [ASSET_REGISTRY.md](./ASSET_REGISTRY.md) and [TODO.md](./TODO.md).

---

## Governance Scope

Governance exists **only** for two purposes:

1. **Marketing proposals** — per-token marketing spend voting (top 100 holders, snapshot-based)
2. **CTO recovery** — community approval to redirect fee destinations after a legitimate community takeover

Governance **does not** control:

- Protocol upgrades or program deployments
- Fee rates, fee structures, or fee parameters
- Protocol-level configurations
- Program authorities or deployments
- Token Factory, Fee Program, Migration Program, Social Registry Program, or Agent Program internals

Solana token holder balances are resolved from SPL **token accounts** / **Associated Token Accounts** associated with the token mint. The governance system must resolve token accounts to their owners correctly — do not assume single-account-per-owner semantics; an owner may hold tokens across multiple token accounts.

### Governance Approval ≠ Administrative Execution

Governance approval does **not** automatically authorize an arbitrary wallet to execute a privileged protocol change. The distinction is:

- **ON-CHAIN GOVERNANCE APPROVAL** — recorded in program state (snapshot + votes)
- **ADMINISTRATIVE EXECUTION** — the authorized admin performs the actual privileged action

The typical example is fee-route changes: governance establishes "the eligible governance participants approved this change," while the authorized administrator executes the actual destination configuration change via the Fee Program.

The governance record is the on-chain proof of community consensus; the admin role is the execution authority that references that record.

---

## Social Registry

> **Status:** Conceptual architecture, partially implemented per existing docs.

A social identity can be associated with a FartBull account/wallet through an authentication and verification process. The frontend/API handles authentication; the protocol maintains an on-chain representation of the verified relationship.

```
User
 ↓
Frontend (fartbull.xyz)
 ↓
Authentication provider
 ↓
API (api.fartbull.xyz)
 ↓
Social Registry Program (on-chain PDA)
 ↓
Wallet / escrow relationship
```

OAuth secrets and private credentials are **never** stored on-chain. Social identity is an identity/verification layer, not a wallet equivalent.

See [SOCIAL_REGISTRY.md](./SOCIAL_REGISTRY.md) for full details.

---

## Protocol Invariants

1. **Total supply is fixed** — each token launch creates 1,000,000,000 tokens. 800,000,000 allocated to the Bonding Curve (trading), 200,000,000 reserved for liquidity migration. This is supply allocation, not fee distribution.

2. **Platform revenue is deducted first** — the platform share is taken before any marketing/destination split.

3. **One Fee Program for all tokens** — no per-token fee programs are deployed. Each token's accounting is fully isolated in its own PDA state account.

4. **Fees accumulate, never auto-transfer** — recipients must call `claim` to receive funds. No automatic pushes to destination wallets.

5. **Marketing is optional** — enabled or disabled per token. When enabled, 20% of the net fee goes to marketing, 80% goes to destination.

6. **Initial $299 DEX Screener spend** — when marketing is enabled, the first $299 USD equivalent is automatically reserved for DEX Screener enhancement.

7. **Bonding curve is frozen after migration** — no trades can occur on the curve once migration completes.

8. **Social payouts are claim-based** — verified social identities register a claim destination; fees accrue to the Fee Program and are claimed manually.

9. **Fee accounting invariant per token:** `fees received = protocol allocation + marketing allocation + destination allocation + legitimately claimed amounts`. The accounting for one token never affects another.

10. **Agent authority is least-privilege** — agents operate with program-enforced, narrowly-scoped permissions. No agent has blanket protocol authority.

---

## Module Connections

```
┌─────────────────────────────────────────────────────────┐
│                    Frontend (fartbull.xyz)               │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│              Solana RPC                                  │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│            Token Factory Program                          │
│  Creates: Token Mint + Bonding Curve State PDA           │
│  Initializes: Agent PDA, Asset Registry PDA              │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│            Bonding Curve State PDA                        │
│  - Price discovery (linear)                              │
│  - Forwards fees to Fee Program                           │
│  - Triggers migration via Migration Program               │
└──────────┬────────────────────┬─────────────────────────┘
           │                    │
           ▼                    ▼
┌──────────────────┐  ┌─────────────────────────────────────┐
│    Token Mint    │  │            Fee Program              │
│   (SPL Token)    │  │ - Shared across all tokens          │
│                  │  │ - Per-token PDA accounting          │
│                  │  │ - Platform revenue first            │
│                  │  │ - Marketing split (optional)        │
│                  │  │ - claim() for all distributions     │
└──────────────────┘  └────────────┬────────────────────────┘
                                  │
                                  ▼
                └─────────────────────────────────────┐
                │    Migration Program                 │
                │  - Retrieves tokens + SOL             │
                │  - Adds Solana DEX liquidity           │
                └────────────┬────────────────────────┘
                             │
                             ▼
                └─────────────────────────────────────┐
                │    Solana DEX / AMM                  │
                │  - Post-migration trading              │
                │  - Fees continue flowing to            │
                │    Fee Program                          │
                └────────────┬────────────────────────┘
                             │
                             ▼
                └─────────────────────────────────────┐
                │       Social Registry Program        │
                │  - Verifies social identities        │
                │  - Registers claim destinations      │
                └────────────┬────────────────────────┘
                             │
                             ▼
                └─────────────────────────────────────┐
                │       Governance Program             │
                │  - Top-100 snapshot voting           │
                │  - Marketing proposals               │
                │  - CTO recovery                      │
                └─────────────────────────────────────┘
```

---

*This protocol specification is the authoritative source. All other documents in this suite must be consistent with it. If any conflict arises, this document supersedes all others.*

*Protocol specification last updated: August 2026*
*Next review: October 2026*
