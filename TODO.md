# FartBull — Open Architectural Questions

> This document tracks every unresolved architectural question in the FartBull protocol documentation. Each question is marked as `[OPEN]` if not yet decided, or `[FUTURE]` if planned for a later phase. **Open questions are NOT silently answered.**

---

## Agent System

| ID | Question | Status | Owner |
|----|----------|--------|-------|
| [OPEN] | Exact Agent PDA seeds | `TBD — PDA seeds` | Implementation team |
| [OPEN] | Agent permission model (exact scope and granularity) | `TBD` | Engineering team |
| [OPEN] | Agent treasury reference model | `TBD` | Engineering team |
| [OPEN] | Agent automation configuration fields | `TBD` | Engineering team |
| [OPEN] | Agent status management (active/paused/revoked) transitions | `TBD` | Engineering team |
| [OPEN] | Agent PDA authority for associated launches | `TBD` | Engineering team |
| [OPEN] | Agent social identity linkage mechanism | `TBD` | Engineering team |
| [FUTURE] | Agent upgrade/migration mechanism | `TBD` | Engineering team |

---

## Asset Registry & Stock-Linked Tokens

| ID | Question | Status | Owner |
|----|----------|--------|-------|
| [OPEN] | Asset custody model | `OPEN DESIGN / LEGAL & COMPLIANCE REQUIREMENT` | Protocol + Legal |
| [OPEN] | Stock-linked token legal structure | `FUTURE MODULE / DESIGN REQUIRED` | Legal team |
| [OPEN] | Oracle / data source provider | `TBD — oracle provider` | Engineering team |
| [OPEN] | Asset ID format | `TBD — asset ID format` | Engineering team |
| [OPEN] | Cross-chain asset verification model | `TBD — cross-chain model` | Engineering team |
| [OPEN] | Price/data oracle integration | `TBD` | Engineering team |
| [OPEN] | Settlement mechanism for asset-linked tokens | `TBD — settlement model` | Engineering team |
| [OPEN] | Holder accounting for asset-linked yields | `TBD` | Engineering team |
| [OPEN] | Distribution accounting for asset-linked yields | `TBD` | Engineering team |
| [OPEN] | Stock allocation percentage (proposed fee integration) | `TBD` | Governance team |
| [OPEN] | Asset approval governance threshold | `TBD` | Governance team |
| [FUTURE] | Supported chains for cross-chain asset reference | `TBD` | Engineering team |

---

## Migration & Liquidity

| ID | Question | Status | Owner |
|----|----------|--------|-------|
| [OPEN] | Exact migration threshold values | `TBD` | Protocol team |
| [OPEN] | Migration trigger condition categories (supply-based, SOL-balance, slot-based) thresholds | `TBD` | Protocol team |
| [OPEN] | DEX selection (Raydium, Orca, Meteora, etc.) | `TBD — DEX selection` | Engineering team |
| [OPEN] | LP liquidity position representation (NFT, LP token, PDA) | `TBD — DEX liquidity-position model` | Engineering team |
| [OPEN] | LP position ownership (who owns the liquidity) | `TBD — LP ownership` | Engineering team |
| [OPEN] | LP locking mechanism | `TBD — LP locking mechanism` | Engineering team |
| [OPEN] | LP lock duration | `TBD — LP lock duration` | Engineering team |
| [OPEN] | LP unlock conditions | `TBD — LP unlock conditions` | Engineering team |
| [OPEN] | LP position authority model | `TBD — LP authority model` | Engineering team |
| [OPEN] | Migration failure handling / rollback semantics | `TBD` | Engineering team |
| [OPEN] | Burn destination for unused migration buffer tokens | `TBD — burn destination` | Protocol team |
| [OPEN] | Migration edge-case behavior (final/in-flight buys) | `TBD` | Engineering team |

---

## Governance

| ID | Question | Status | Owner |
|----|----------|--------|-------|
| [OPEN] | Final governance approval threshold | `TBD — final governance approval threshold` | Governance team |
| [OPEN] | CTO approval threshold | `TBD — final CTO approval threshold` | Governance team |
| [OPEN] | Minimum holding duration for eligibility | `DESIGN OPTION` | Governance team |
| [OPEN] | Anti-splitting analysis mechanism | `DESIGN OPTION` | Governance team |
| [OPEN] | Governance snapshot mechanism details | `TBD` | Engineering team |
| [OPEN] | Governance PDA seeds | `TBD — PDA seeds` | Engineering team |

---

## Fees

| ID | Question | Status | Owner |
|----|----------|--------|-------|
| [OPEN] | Platform fee BPS (default is 200 BPS = 2%, final TBD) | `TBD` | Protocol team |

---

## Token & Metadata

| ID | Question | Status | Owner |
|----|----------|--------|-------|
| [OPEN] | Token Metadata implementation (Metaplex or other) | `TBD — metadata implementation` | Engineering team |
| [OPEN] | Burn authority for unused tokens | `TBD — burn destination` | Protocol team |
| [FUTURE] | Token-2022 extensions | `OPTIONAL / FUTURE` | Engineering team |

---

## Solana Program Details

| ID | Question | Status | Owner |
|----|----------|--------|-------|
| [OPEN] | Program IDs (all six shared programs) | `TBD` | Deployment team |
| [OPEN] | PDA seeds (all state accounts) | `TBD — PDA seeds` | Implementation team |
| [OPEN] | Solana program framework (Anchor vs native Rust) | `TBD — Solana program framework` | Engineering team |
| [OPEN] | Production upgrade authority policy / revocation | `TBD` | Security team |
| [OPEN] | Social registry escrow mechanism | `TBD — social registry escrow mechanism` | Implementation team |
| [OPEN] | Social verification mechanism (on-chain signature validation) | `TBD` | Engineering team |

---

## API & Off-Chain Services

| ID | Question | Status | Owner |
|----|----------|--------|-------|
| [OPEN] | API rate limit details (per-endpoint) | `TBD` | Backend team |
| [OPEN] | API authentication mechanism (API key format) | `TBD` | Backend team |
| [OPEN] | Agent orchestration service design | `TBD` | Backend team |
| [OPEN] | Oracle provider for off-chain price feeds | `TBD` | Backend team |

---

## Network & Deployment

| ID | Question | Status | Owner |
|----|----------|--------|-------|
| [OPEN] | RPC endpoint for mainnet | `TBD` | Infrastructure team |
| [OPEN] | DEX selection for post-migration trading | `TBD — DEX selection` | Protocol team |

---

## Summary

| Category | OPEN Count | FUTURE Count |
|----------|-----------|-------------|
| Agent System | 7 | 1 |
| Asset Registry & Stocks | 12 | 2 |
| Migration & Liquidity | 12 | 0 |
| Governance | 6 | 0 |
| Fees | 1 | 0 |
| Token & Metadata | 3 | 1 |
| Solana Program Details | 6 | 0 |
| API & Off-Chain | 4 | 0 |
| Network & Deployment | 2 | 0 |
| **Total** | **53** | **6** |

---

*Open questions tracker last updated: August 2026*
*Next review: October 2026*
*Source of truth: [PROTOCOL.md](./PROTOCOL.md)*
