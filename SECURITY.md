# FartBull — Security Framework

See [PROTOCOL.md](./PROTOCOL.md) for the authoritative protocol specification. This document describes the designed security architecture and the active threat model — rewritten for Solana's account and program model.

**Chain:** Solana (`mainnet-beta`) · **Token Standard:** SPL Token · **Native Asset:** SOL (lamports)

---

## Table of Contents

1. [Security Architecture](#security-architecture)
2. [Security Patterns](#security-patterns)
3. [Threat Model](#threat-model)
4. [Authorization Model](#authorization-model)
5. [Agent Security](#agent-security)
6. [Asset Registry Security](#asset-registry-security)
7. [Security Checklist](#security-checklist)

---

## Security Architecture

### Defense-in-Depth

Security is layered across development, pre-deployment, and runtime phases:

```mermaid
flowchart LR
    subgraph "Development Phase"
        A[Code Review] --> B[Unit Tests]
        B --> C[Static Analysis<br/>(cargo clippy, cargo-audit)]
    end

    subgraph "Pre-Deployment"
        C --> D[Internal Security Review]
        D --> E[Solana Program Audit]
    end

    subgraph "Deployment Phase"
        E --> F[Mainnet Launch]
        F --> G[Real-time Monitoring]
        G --> H[Incident Response]
    end

    style A fill:#e8f5e9
    style B fill:#e8f5e9
    style C fill:#e8f5e9
    style D fill:#fff3e0
    style E fill:#fff3e0
    style F fill:#e8f4fd
    style G fill:#e8f4fd
    style H fill:#e8f4fd
```

This document does **not** claim completed audits, audit firm approval, formal verification, a completed bug bounty program, security ratings, or penetration tests. The repository contains specifications and documentation; audit status is TBD.

### Security Model

- **Rust with checked arithmetic** — all math operations use `checked_add`, `checked_sub`, `checked_mul` to prevent overflow
- **Account validation** — every account passed to an instruction is validated for owner, derivation, and writability
- **PDA verification** — all PDA accounts are verified via `find_program_address` on every instruction
- **Signer verification** — required signers must be present and marked as signers
- **CPI safety** — cross-program invocations are minimal and validated
- **Role-based access control** — authorized admin role for protocol-level operations
- **Least privilege** — agent permissions are narrowly scoped and program-enforced

### Solana-Specific Considerations

Solana programs are stateless executables. Mutable state lives in separate accounts that programs validate explicitly. Because account data is supplied to programs (not stored internally to the program), every instruction must validate:

- **Account owner** — the account owner must be the expected program
- **PDA derivation** — PDA accounts must be verified via `find_program_address`
- **Signer** — required signers must be present
- **Writable flag** — only accounts that need mutation should be writable
- **Mint** — token accounts must match the expected mint
- **Token Program** — the SPL Token Program ID must be validated

Solana's account model is fundamentally different from architectures where state is internal to a single contract.

---

## Security Patterns

### Account Validation Pattern

Every Solana instruction must validate all accounts it reads or writes before any state mutation occurs:

```rust
// Validate PDA derivation
let expected_curve_pda = find_program_address(
    &[b"curve", mint.key.as_ref(), creator.key.as_ref()],
    program_id
);
if expected_curve_pda != *curve_pda.key {
    return Err(ErrorCode::InvalidPda.into());
}

// Validate account owner
if curve_pda.owner != program_id {
    return Err(ErrorCode::InvalidAccountOwner.into());
}

// Validate signer
if !signer.is_signer {
    return Err(ErrorCode::MissingRequiredSignature.into());
}
```

### Reentrancy Protection (Solana-native)

Solana does **not** have recursive call reentrancy. CPIs do not re-enter in the same execution context. However, CPIs can fail and leave state in an inconsistent state. The protection is:

1. **Update state before CPI** — all state changes happen before cross-program invocations
2. **Idempotent claims** — claim operations can be retried safely because they zero the balance before transferring
3. **No push payments** — funds are always pulled via claim instructions, never pushed

### Checked Arithmetic

All mathematical operations use checked arithmetic to prevent overflow:

```rust
let new_sold = sold.checked_add(amount)
    .ok_or(ErrorCode::MathOverflow)?;
```

Rust panics on integer overflow only in debug mode, and release builds wrap silently. Solana programs **must** use explicit checked operations for all arithmetic.

### Immutable Parameters

Curve parameters are stored as immutable fields in the state PDA:

- `base_price`
- `slope`
- `fee_bps`
- `total_supply`

These are set during initialization and cannot be changed afterward. There is no `set_parameters` instruction.

---

## Threat Model

### Core Protocol Threats

| Threat | Mitigation |
|--------|------------|
| **Account substitution** | PDA derivation validation on every instruction; mint validation on all token accounts |
| **Reentrancy** | Solana has no recursive call reentrancy; state updated before CPI; pull-pattern for all payouts |
| **Integer overflow/underflow** | Rust checked arithmetic (`checked_add`, `checked_sub`, `checked_mul`) on all math |
| **Front-running** | Slippage protection parameters (`min_tokens_out`); compute-unit priority fees |
| **Approval race conditions** | SPL tokens use direct transfer/mint CPIs; no allowance race conditions |
| **Fee accounting imbalance** | Per-token PDA isolation invariant; accounting checks in Fee Program |
| **Cross-token balance leakage** | Each token has its own Fee Config PDA; no shared mutable state accessible by other tokens |
| **Unauthorized claims** | Signer validation on every claim path; destination verified against PDA state |
| **Governance manipulation** | Snapshot-based top-100 voting with frozen voter set at proposal creation |
| **Snapshot integrity** | Balances frozen at proposal creation slot; later changes ignored |
| **PDA derivation bypass** | `find_program_address` validation on every PDA account; bump seed verified |
| **Unauthorized migration** | Only the Migration Program PDA can call `mark_graduated`; verified via CPI signer |
| **Unauthorized fee forwarding** | Only the Bonding Curve Program can invoke `on_trade`; verified via CPI |
| **Duplicate initialization** | `is_initialized` flag in state PDAs; reject if already initialized |
| **Duplicate migration** | `graduated` flag in Curve State PDA; reject if already migrated |
| **Mint mismatch** | Token accounts validated against expected mint in PDA state |
| **Signer spoofing** | Solana runtime enforces signer verification; programs check `is_signer` |

### Agent System Threats

| Threat | Mitigation |
|--------|------------|
| **Agent authority abuse** | Agent permissions are program-enforced, narrow, and least-privilege |
| **Agent PDA impersonation** | PDA derivation validation on every agent instruction |
| **Arbitrary treasury withdrawal** | Restricted by agent permission scopes; not granted by default |
| **Protocol upgrade via agent** | Agent permissions never include upgrade authority |
| **Governance bypass** | Agent actions are logged on-chain; governance proposals required for privileged changes |
| **Arbitrary program configuration** | Agent cannot modify protocol configuration; only pre-approved actions |
| **Arbitrary asset creation** | Agent must be explicitly granted asset creation permission per-launch |

### Asset Registry Threats

| Threat | Mitigation |
|--------|------------|
| **Oracle manipulation** | Oracle is abstracted; provider is TBD; price feeds must be from verified sources |
| **Stale pricing** | Freshness checks in PDA state |
| **Wrong asset reference** | Asset IDs validated against registry PDA |
| **Legal/compliance bypass** | Asset custody and legal ownership are explicitly distinguished as OPEN; no automated issuance |

### Social Registry Threats

| Threat | Mitigation |
|--------|------------|
| **OAuth secret storage** | Never stored on-chain; only verification status recorded |
| **Social account compromise** | Off-chain authentication layer; on-chain state is verification result only |
| **Identity spoofing** | Solana signed message verification for signature-based platforms |
| **Sybil attacks** | Unique wallet requirement; snapshot-based eligibility |
| **Replay attacks** | Nonce-based proofs; non-reusable nonces |

### Migration Threats

| Threat | Mitigation |
|--------|------------|
| **Migration race condition** | `graduated` flag prevents double-migration; atomic CPI chain |
| **LP ownership theft** | LP position ownership/custody model is TBD; documented as OPEN |
| **Insufficient liquidity** | `min_sol_amount` parameter bounds acceptable liquidity |
| **Unrugpull safety** | Liquidity position ownership and locking model must be explicitly defined before mainnet |
| **Migration failure** | Curve remains frozen=false; funds recoverable; retry possible |

### API / Off-Chain Threats

| Threat | Mitigation |
|--------|------------|
| **API compromise** | API is not authoritative; all on-chain state is independently verifiable via Solana RPC |
| **Indexing lag** | Users can verify directly via Solana RPC / Solscan |
| **False data presentation** | API responses must reference on-chain state; discrepancy reporting path required |

### Cross-Chain Assumptions

| Threat | Mitigation |
|--------|------------|
| **Bridge failure** | No bridges are specified; cross-chain asset support is future extensibility only |
| **Oracle on foreign chain** | Asset Registry abstracts oracle; foreign-chain integration is TBD |

Do not invent cross-chain bridge mechanisms. Cross-chain functionality is documented as future extensibility only.

---

## Authorization Model

### Roles

| Role | Capabilities |
|------|--------------|
| **Trader** | Buy tokens on the Bonding Curve; claim their own destination balance |
| **Creator** | Configure initial destination and marketing flag for their token at launch |
| **Authorized Admin** | Claim protocol revenue; execute CTO destination changes after governance approval; manage upgrade authority |
| **Governor (top 100)** | Submit and vote on marketing and CTO proposals |
| **Agent** | Pre-approved subset of actions (launch token, interact with approved curves, execute predefined actions, interact with approved assets, publish through approved social integrations) |

### Claim Authorization

Each claim path performs an authorization check before releasing funds:

```
Destination Balance (PDA)
       |
       v
claim_destination instruction
       |
       v
Authorization Check (signer validated against PDA destination)
       |
       v
Funds Released (SOL transferred via System Program CPI)
```

The protocol balance is claimable only by the authorized protocol admin. Marketing funds are claimable/spendable only via passing governance proposals. Destination funds are claimable only by the registered destination (wallet or verified social identity).

### Administrative Controls

Administrative functions are gated by an authorized admin role. Emergency pause and recovery specifics are TBD; they are not claimed as deployed here. The authorized admin cannot unilaterally modify fee accounting, bypass governance for destination changes, or alter per-token accounting isolation.

### Upgrade Authority

Solana programs use **program upgrade authorities** managed via the BPF loader upgrade mechanism, which is separate from the state PDAs that hold protocol configuration.

| Authority Type | Scope |
|----------------|-------|
| **Program Upgrade Authority** | Controls whether the program binary can be upgraded (separate from state PDAs) |
| **Configuration Authority** | Controls protocol-level configuration parameters in PDA state |

If the final production protocol intends to make programs immutable, document:

```
Upgrade Authority Revocation
```

as a production security milestone. Do not claim immutability unless the authority is actually revoked. Solana's documentation explicitly notes that upgradeable programs can have an upgrade authority and that revoking it makes the program immutable.

---

## Agent Security

The documentation explicitly separates:

**AI / automation logic** (off-chain API / orchestration)

from

**on-chain authority** (program-enforced, PDA-scoped).

The agent should not automatically have unrestricted protocol authority. Document the principle of least privilege:

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

> **Open:** Exact agent permission model and PDA seeds. See [AGENT_SYSTEM.md](./AGENT_SYSTEM.md) and [TODO.md](./TODO.md).

---

## Asset Registry Security

The Asset Registry must not be treated as the ultimate authority for on-chain token economics. See [ASSET_REGISTRY.md](./ASSET_REGISTRY.md).

Key risks to address:
- **Price exposure** ≠ **tokenized asset** ≠ **legal ownership**
- Oracle provider is TBD
- Custody/legal model is TBD (OPEN DESIGN / LEGAL & COMPLIANCE REQUIREMENT)

---

## Security Checklist

### Pre-Deployment Checklist

```markdown
- [ ] Rust code reviewed by team
- [ ] Unit tests written (program instruction tests, account validation, PDA derivation)
- [ ] Static analysis run (cargo clippy, cargo-audit)
- [ ] Security review status tracked (audit: TBD)
- [ ] Compute unit cost analysis reviewed
- [ ] Incident procedures documented
- [ ] Monitoring systems configured
```

### Post-Deployment Checklist

```markdown
- [ ] Programs verified on Solana explorer (solscan.io)
- [ ] Monitoring configured
- [ ] Community notified
- [ ] Incident response procedures tested
```

### Common Vulnerabilities Addressed

| Vulnerability | Mitigation |
|---------------|------------|
| Account substitution | PDA derivation validation on every instruction |
| Reentrancy | No recursive call reentrancy; state before CPI; pull over push |
| Integer Overflow | Rust checked arithmetic |
| Access Control | Signer validation + role-based PDA state |
| State Visibility | Explicit PDA derivation and ownership checks |
| Front Running | Slippage protection parameters |
| Denial of Service | Compute unit limits; no unbounded loops |
| Fee Accounting Imbalance | Per-token PDA isolation invariant |
| Token Isolation | Separate PDA per token; no cross-token access |
| Duplicate initialization | `is_initialized` flag in all state PDAs |
| Duplicate migration | `graduated` flag in Curve State PDA |
| CPI safety | Minimal CPIs; validate all CPI account inputs |
| Agent authority abuse | Least-privilege permissions; program-enforced |
| API compromise | On-chain state is authoritative; API is convenience |
| Social account compromise | Off-chain auth layer; on-chain is verification result only |

---

*Security framework last updated: August 2026*
*Next review: October 2026*
*Source of truth: [PROTOCOL.md](./PROTOCOL.md)*
