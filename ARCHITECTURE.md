# FartBull — System Architecture

See [PROTOCOL.md](./PROTOCOL.md) for the authoritative protocol specification. This document provides the architectural overview, module breakdown, and data flow.

**Chain:** Solana (`mainnet-beta`) · **Token Standard:** SPL Token · **Native Asset:** SOL (lamports)

---

## Architecture Overview

FartBull uses **shared protocol programs** for cross-token services and **per-token state accounts** for launch-specific data. All mutable state is stored in program-owned PDA accounts. Solana programs themselves are stateless executables.

```
Frontend (fartbull.xyz)
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

Full protocol rationale, invariants, and module connections are defined in [PROTOCOL.md](./PROTOCOL.md).

---

## Conceptual Modules

The FartBull architecture is organized into eight modules for separation of concerns. These modules may eventually be implemented as one or multiple programs depending on the implementation decision.

### 1. Token Launch

**Purpose:** Entry point for creating SPL token mints and initializing curve state.
**On-Chain Representation:** Token Factory Program
**Per-Token State:** SPL Token Mint, Bonding Curve State PDA
**PDA Type:** Launch state PDA (TBD seeds)

The Token Factory Program creates the SPL Token Mint via the SPL Token Program and initializes the Bonding Curve State PDA with curve parameters, fee configuration, and destination. It may also initialize Agent PDAs and Asset Registry PDAs as needed.

### 2. Bonding Curve

**Purpose:** Linear price-discovery curve state.
**On-Chain Representation:** Bonding Curve Program + Bonding Curve State PDA
**Fee Integration:** Forwards fees to the Fee Program via CPI
**State:** Stored in a Bonding Curve State PDA (program-owned, frozen after migration)

The bonding curve is the core launch mechanism. Price follows a deterministic linear function; all calculations use lamports and token base units (no floating point).

### 3. Fee Management

**Purpose:** Single shared program for all fee collection, accounting, and routing across every token.
**On-Chain Representation:** Fee Program + per-token Fee Config PDAs
**Accounting:** Isolated per-token balances in dedicated PDA accounts (no cross-token mixing)
**Distribution:** Claim-based — platform revenue first, then marketing/destination split

The Fee Program maintains fully isolated balances per token. There is **one** Fee Program for the entire protocol; no per-token fee programs are deployed.

### 4. Migration

**Purpose:** Transition from bonding curve to a Solana DEX.
**On-Chain Representation:** Migration Program
**Trigger:** Migration condition reached (TBD)
**Dex destination:** Solana DEX / AMM (TBD)

The Migration Program retrieves all SOL and tokens from the Bonding Curve State PDA vault, creates a liquidity position on a Solana DEX, and marks the curve as graduated (frozen).

### 5. Governance

**Purpose:** Snapshot-based top-100 voting for marketing proposals and CTO recovery.
**On-Chain Representation:** Governance Program + Governance PDA state
**Eligibility:** Top 100 token holders by balance at snapshot
**Voting period:** 7 days (slots)
**Quorum:** 5% of total supply

Governance records community consensus. The authorized administrator executes the final privileged action. See [GOVERNANCE.md](./GOVERNANCE.md).

### 6. Agent System

**Purpose:** Autonomous agents whose on-chain identity is a program-controlled PDA.
**On-Chain Representation:** Agent PDA managed by the FartBull Program
**Permissions:** Narrow, program-enforced capabilities (least privilege)

The FartBull Program can create and manage agents. An agent's protocol identity is represented by a Solana PDA. The program defines the agent's permitted capabilities — the website/API must not simply create an unrestricted private key and call that an agent.

> **Status:** Conceptual architecture. Implementation is a future phase. See [AGENT_SYSTEM.md](./AGENT_SYSTEM.md).

### 7. Asset Registry

**Purpose:** Registry of supported assets that may eventually be used by FartBull launch mechanisms.
**On-Chain Representation:** Asset Registry PDA
**Scope:** Optional / future module

The Asset Registry can describe supported assets (crypto, tokenized, stock-linked, etc.) with reference data, oracle/data source, status, and distribution configuration.

> **Status:** Conceptual architecture. See [ASSET_REGISTRY.md](./ASSET_REGISTRY.md).

### 8. Social Registry

**Purpose:** Identity verification and social-account association for claim destinations.
**On-Chain Representation:** Social Registry Program + Identity PDAs
**Integration:** Fee Program uses verified identities as registered destinations

The frontend/API handles authentication and verification; the Social Registry Program maintains an on-chain representation of the verified relationship. OAuth secrets and private credentials are never stored on-chain.

See [SOCIAL_REGISTRY.md](./SOCIAL_REGISTRY.md).

---

## Shared vs Per-Token Accounts

| Account Type | Scope | Count |
|--------------|-------|-------|
| **Token Factory Program** | Shared | 1 |
| **Fee Program** | Shared | 1 |
| **Migration Program** | Shared | 1 |
| **Social Registry Program** | Shared | 1 |
| **Governance Program** | Shared | 1 |
| **Agent Program** | Shared | 1 (or part of Token Factory) |
| **Asset Registry Program** | Shared | 1 (or part of Token Factory) |
| **SPL Token Mint** | Per launch | Per token |
| **Bonding Curve State PDA** | Per launch | Per token |
| **Fee Config PDA** | Per launch | Per token |
| **Agent PDA** | Per agent | Per agent |
| **Asset PDA** | Per asset | Per asset |

There is **one** Fee Program, one Migration Program, one Social Registry Program, one Governance Program for the entire protocol. Each token's accounting is fully isolated in its own PDA state account.

Per-token accounts (Token Mint, Bonding Curve State PDA, Fee Config PDA) are initialized fresh for each launch via the Token Factory Program.

---

## Account Ownership Model

Every Solana account passed to an instruction **must** be validated:

- **Owner check** — the account owner must be the expected program (or the SPL Token Program for token accounts)
- **PDA derivation check** — PDA accounts must be verified via `find_program_address`
- **Signer check** — required signers must be present and marked as signers in the instruction
- **Writable check** — only accounts that need mutation should be marked writable
- **Mint check** — token accounts must match the expected mint
- **Token Program check** — the SPL Token Program ID must be validated

Program IDs for all shared programs: `TBD` (not yet deployed). Do not fabricate addresses.

---

## PDA Architecture

FartBull uses Program-Derived Addresses for all program-controlled accounts. A PDA is derived from:

```
Program ID + seeds
```

PDAs are used for the following purposes. **Exact seeds are TBD** unless already established:

| PDA Purpose | Status |
|-------------|--------|
| Global configuration | `TBD — PDA seeds` |
| Token launch state | `TBD — PDA seeds` |
| Bonding curve state | `TBD — PDA seeds` |
| Fee config / per-token accounting | `TBD — PDA seeds` |
| Fee vault | `TBD — PDA seeds` |
| Creator fee vault | `TBD — PDA seeds` |
| Marketing state | `TBD — PDA seeds` |
| Migration state | `TBD — PDA seeds` |
| Protocol treasury | `TBD — PDA seeds` |
| Liquidity / migration state | `TBD — PDA seeds` |
| Agent PDA | `TBD — PDA seeds` |
| Agent treasury | `TBD — PDA seeds` |
| Asset registry | `TBD — PDA seeds` |
| Asset entry | `TBD — PDA seeds` |
| Identity PDA | `TBD — PDA seeds` |
| Governance PDA | `TBD — PDA seeds` |

Do not invent exact PDA seeds until the implementation defines them. Use `TBD — PDA seeds` where the implementation has not yet specified the seed scheme.

---

## SPL Token Model

FartBull uses the **standard SPL Token Program** (not Token-2022). Each launch creates:

```
SPL Mint
    |
    +--> Token Accounts (holders)
    |
    +--> Associated Token Accounts (ATAs)
    |
    +--> Metadata Account (TBD — metadata implementation)
```

- **Token Mint** — defines the token (name, symbol, decimals, supply, authorities)
- **Token Account** — holds a wallet's token balance for a given mint
- **Associated Token Account (ATA)** — deterministic token account for a wallet/mint pair
- **Metadata Account** — `TBD — metadata implementation` (Metaplex Token Metadata if adopted)

### Token Standard Decision

Unless the existing project specification requires Token-2022 functionality:

**Use the standard SPL Token program.**

Do NOT introduce Token-2022 extensions simply because they exist. Token-2022 is documented as an OPTIONAL / FUTURE enhancement only:

```
Token-2022: OPTIONAL / FUTURE — not part of current design
```

### Token Metadata

Standard SPL Token functionality does not itself provide token name/symbol/image metadata. Solana documentation describes Metaplex Token Metadata as a metadata account associated with the token mint.

```
SPL Mint
    |
    +--> Token Accounts
    |
    +--> Metadata Account (TBD — metadata implementation)
```

If FartBull adopts Metaplex metadata, it will be documented explicitly. If the implementation has not yet selected a metadata system:

```
TBD — metadata implementation
```

---

## Upgrade Authority

Solana programs use **program upgrade authorities** managed via the BPF loader upgrade mechanism, which is separate from the state PDAs that hold protocol configuration.

| Authority Type | Scope |
|----------------|-------|
| **Program Upgrade Authority** | Controls whether the program binary can be upgraded |
| **Configuration Authority** | Controls protocol-level configuration parameters |

If the final production protocol intends to make programs immutable, document:

```
Upgrade Authority Revocation
```

as a production security milestone. Do not claim immutability unless the authority is actually revoked.

---

## On-Chain vs Off-Chain

```
┌─────────────────────────────┐
│         ON-CHAIN             │
├─────────────────────────────┤
│ Token Factory Program        │
│ Fee Program                  │
│ Migration Program            │
│ Social Registry Program      │
│ Governance Program           │
│ Agent Program                │
│ Asset Registry Program       │
│ SPL Token Program            │
│ System Program               │
│                          │
│ Bonding Curve State PDA      │
│ Fee Config PDA               │
│ Agent PDA                    │
│ Asset Registry PDA           │
│ Identity PDA                 │
│ Governance PDA               │
│ PDA Vault Accounts           │
│ SPL Token Mints              │
│ Token Accounts / ATAs        │
└─────────────────────────────┘
            │
            │ RPC
            ▼
┌─────────────────────────────┐
│         OFF-CHAIN            │
├─────────────────────────────┤
│ Frontend (fartbull.xyz)      │
│ API (api.fartbull.xyz)       │
│ Indexing services            │
│ Notification services        │
│ Oracle providers             │
│ Social authentication        │
│ Agent automation services    │
│ Market/data services         │
└─────────────────────────────┘
```

**On-chain programs are authoritative for protocol state and protocol-enforced permissions.** Off-chain services (frontend, API, indexing) provide interfaces and convenience but cannot override on-chain rules.

---

*Architecture specification last updated: August 2026*
*Next review: October 2026*
*Source of truth: [PROTOCOL.md](./PROTOCOL.md)*
