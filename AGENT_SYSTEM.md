# FartBull — Agent System

> **Status:** Conceptual architecture. Implementation is a future phase.

**Chain:** Solana (`mainnet-beta`) · **Token Standard:** SPL Token · **Native Asset:** SOL (lamports)

See [PROTOCOL.md](./PROTOCOL.md) for the authoritative protocol specification.

---

## Overview

FartBull can create and manage autonomous agents whose protocol identity is represented by a **Solana PDA**. The FartBull Program defines, manages, and enforces the agent's capabilities.

**The website/API must NOT simply create an unrestricted private key and call that an agent.** The program should define the agent's permitted capabilities. Agent authority is program-enforced, narrow, and least-privilege.

The AI/automation logic (off-chain) is explicitly separated from on-chain authority (program-enforced). The agent does not automatically gain unrestricted protocol authority.

---

## Conceptual Architecture

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

---

## Intended Concepts

| Concept | Description | Status |
|---------|-------------|--------|
| **Agent PDA** | A program-owned PDA representing the agent's on-chain identity | `TBD — PDA seeds` |
| **Agent identity** | The binding between the agent and its allowed creator/on-chain actions | Design |
| **Agent configuration** | Parameters defining the agent's behavior envelope | `TBD` |
| **Agent permissions** | Narrow, program-enforced capabilities (least privilege) | `TBD — permission model` |
| **Agent status** | Active / Paused / Revoked | `TBD` |
| **Agent associated launches** | Token launches this agent is permitted to interact with | `TBD` |
| **Agent treasury references** | Treasury PDAs controlled by the agent's authority | `TBD` |
| **Agent social identity** | Verified social identity linkage (via Social Registry) | Design |
| **Agent automation configuration** | Off-chain execution parameters for automated actions | `TBD` |

> **Do not invent the exact PDA seeds yet unless they already exist in documentation.** Mark implementation-specific PDA seeds as TODO if not established.

---

## Agent PDA

The Agent PDA is managed and enforced by the FartBull Program. It represents the agent's on-chain identity and serves as the authority for agent-scoped actions.

```
Agent PDA
    |
    +--> Agent Config (behavior envelope)
    +--> Permissions (allowed actions)
    +--> Status (active / paused / revoked)
    +--> Associated Launches (permitted tokens)
    +--> Treasury References (controlled PDAs)
    +--> Social Identity (verified identity linkage)
    +--> Automation Config (off-chain trigger rules)
```

### PDA Seeds

```
TBD — Agent PDA seeds
```

The seeds will likely include a namespace (e.g. `b"agent"`) and an agent identifier. Do not invent exact seeds until the implementation defines them.

### Agent Identity

The agent's identity on-chain is the PDA derived from the FartBull Program. The program validates that the agent PDA is the correct authority for any action the agent attempts.

---

## Agent Permissions

### Permission Framework

The permission framework grants each agent a narrowly-scoped set of capabilities. Permissions are evaluated on-chain by the FartBull Program before any action is authorized.

Potential permissions may include:

| Permission | Description | Status |
|------------|-------------|--------|
| `launch_token` | Ability to create new token launches | `TBD` |
| `interact_curve` | Ability to buy/sell on approved bonding curves | `TBD` |
| `execute_action` | Ability to execute predefined actions | `TBD` |
| `interact_asset` | Ability to interact with approved assets | `TBD` |
| `publish_social` | Ability to publish through approved social integrations | `TBD` |

Potentially restricted (not granted by default):

| Capability (Restricted) | Description | Status |
|------------------------|-------------|--------|
| `arbitrary_treasury_withdrawal` | Withdraw funds from any treasury | Restricted — not granted by default |
| `protocol_upgrade` | Upgrade protocol programs | Restricted — never granted to agents |
| `governance_bypass` | Vote/bypass governance requirements | Restricted — never granted |
| `arbitrary_config` | Modify program configuration | Restricted — never granted |
| `arbitrary_asset_creation` | Create any asset in the registry | Restricted — requires approval |

> **Open:** The exact permission framework and granularity are TBD. See [TODO.md](./TODO.md).

### Principle of Least Privilege

The agent should not automatically have unrestricted protocol authority. The program enforces:

1. Only the explicitly granted permissions are available
2. Permissions are checked against the Agent PDA state on every instruction
3. The agent cannot escalate its own permissions
4. The protocol admin can revoke or pause an agent

---

## Agent Status

| Status | Description |
|--------|-------------|
| `Active` | Agent is enabled and can perform permitted actions |
| `Paused` | Agent is temporarily disabled; no actions permitted |
| `Revoked` | Agent is permanently disabled; PDA frozen |

Status changes are performed by the authorized protocol admin.

---

## Agent Associated Launches

Each agent may be associated with zero or more token launches. The agent can only interact with launches it is explicitly permitted to access.

```
Agent PDA
    |
    v
Associated Launches (list of mint PDAs)
    |
    v
Per-Launch Permissions
    |
    v
- Buy/Sell (yes/no)
- Fee config (yes/no)
- Migration trigger (yes/no)
```

---

## Agent Treasury References

The agent may control or reference treasury PDAs for its operations. Treasury references are program-enforced — the agent cannot access treasuries outside its permitted scope.

```
Agent PDA
    |
    v
Treasury References (list of PDAs)
    |
    v
Per-Treasury Limits (amount, currency)
```

> **Open:** Treasury reference model and spending limits are TBD.

---

## Agent Social Identity

An agent may be linked to verified social identities from the Social Registry. This enables the agent to publish and claim through verified channels.

```
Agent PDA
    |
    +--> Social Identity PDA (via Social Registry)
    +--> Verified Platform Handles
    +--> Claim Destinations
```

---

## Agent Automation Configuration

Automation rules live off-chain (in the API/agent orchestration layer) but are constrained by on-chain permissions:

| Field | Description | Status |
|-------|-------------|--------|
| `trigger_rules` | Conditions under which the agent acts | `TBD` |
| `schedule` | Time-based or event-based scheduling | `TBD` |
| `max_tx_per_block` | Rate limiting for automated actions | `TBD` |
| `spend_limit` | Maximum spend per period | `TBD` |

The off-chain automation layer may only act within the agent's on-chain permission envelope. On-chain permission checks are the ultimate authority.

---

## API / On-Chain Separation

```
┌─────────────────────────────────────┐
│           ON-CHAIN (Authority)      │
├─────────────────────────────────────┤
│ FartBull Program                      │
│ Agent PDA (identity + permissions)   │
│ Instruction validation (permission   │
│ checks enforced per action)           │
└─────────────────────────────────────┘
            │
            │ RPC
            ▼
┌─────────────────────────────────────┐
│         OFF-CHAIN (Logic)            │
├─────────────────────────────────────┤
│ API (api.fartbull.xyz)               │
│ Agent orchestration layer            │
│ AI/automation logic                  │
│ Event monitoring                     │
└─────────────────────────────────────┘
```

The on-chain program is authoritative for permissions. The off-chain API provides automation logic but cannot override on-chain permission checks.

---

## Security

### AI/Automation vs Authority

The documentation explicitly separates:

- **AI / automation logic** (off-chain API / orchestration) — this is the "brain"
- **On-chain authority** (program-enforced, PDA-scoped) — this is the "hands"

The agent should not automatically have unrestricted protocol authority.

### Potential Agent Permissions

May include:
- Launch token
- Interact with approved bonding curves
- Execute predefined actions
- Interact with approved assets
- Publish through approved social integrations

Restricted:
- Arbitrary treasury withdrawal
- Protocol upgrade
- Governance bypass
- Arbitrary program configuration
- Arbitrary asset creation

> **Open:** Exact permissions to be defined during implementation. See [TODO.md](./TODO.md).

---

*Agent system documentation last updated: August 2026*
*Next review: October 2026*
*Source of truth: [PROTOCOL.md](./PROTOCOL.md)*
