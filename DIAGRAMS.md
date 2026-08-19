# FartBull — System Diagrams

See [PROTOCOL.md](./PROTOCOL.md) for the authoritative protocol specification. This document is a collection of all Mermaid system diagrams used in the documentation suite.

**Chain:** Solana (`mainnet-beta`) · **Token Standard:** SPL Token · **Native Asset:** SOL (lamports)

All diagrams reflect the canonical architecture: Solana, bonding curve launch, Solana DEX migration, a single shared Fee Program with per-token PDA isolation, claim-based distribution, optional marketing, top-100 snapshot governance, and CTO recovery. Plus the FartBull Agent System and Asset Registry.

---

## 1. System Architecture

### High-Level Overview

```mermaid
graph TD
    subgraph "Application Layer"
        A1[Frontend (fartbull.xyz)]
        A2[API (api.fartbull.xyz)]
    end

    subgraph "On-Chain (Solana)"
        A2 --> B1[Token Factory Program]
        B1 --> B2[SPL Token Mint<br/>via SPL Token Program]
        B1 --> B3[Bonding Curve State PDA]
        B3 --> B4[Fee Program]
        B3 --> B5[Migration Program]
        B5 --> B6[Solana DEX / AMM]
        B4 --> B7[Social Registry Program]
        B1 --> B8[Agent PDA]
        B1 --> B9[Asset Registry PDA]
    end

    subgraph "Monitoring"
        D1[Solscan Explorer]
        D2[Prometheus]
    end

    style A1 fill:#e8f4fd
    style B1 fill:#fff3e0
    style B2 fill:#fff3e0
    style B3 fill:#fff3e0
    style B4 fill:#e8f5e9
    style B5 fill:#e8f5e9
    style B6 fill:#e8f5e9
    style B7 fill:#e8f5e9
    style B8 fill:#e8f5e9
    style B9 fill:#e8f5e9
    style D1 fill:#f3e5f5
    style D2 fill:#f3e5f5
```

---

## 2. FartBull Primary Architecture

The primary diagram showing the eight conceptual modules:

```mermaid
graph TD
    subgraph "FARTBULL"
        CTR[FartBull Program]
    end

    subgraph "Modules"
        direction L
        subgraph "1. Token Launch"
            LAUNCH[Token Factory Program]
        end

        subgraph "2. Bonding Curve"
            BC[Bonding Curve Program]
            BCPDA[Bonding Curve State PDA]
        end

        subgraph "3. Fee Management"
            FEE[Fee Program]
            FEE_PDA[Fee Config Per-Token PDA]
        end

        subgraph "4. Migration"
            MIG[Migration Program]
        end

        subgraph "5. Governance"
            GOV[Governance Program]
        end

        subgraph "6. Agent System"
            AGENT[Agent PDA]
        end

        subgraph "7. Asset Registry"
            ASSET[Asset Registry PDA]
        end

        subgraph "8. Social Registry"
            SOC[Social Registry Program]
        end
    end

    CTR --> LAUNCH
    LAUNCH --> BC
    LAUNCH --> AGENT
    LAUNCH --> ASSET
    BC --> BCPDA
    BCPDA --> FEE
    BCPDA --> MIG
    FEE --> FEE_PDA
    FEE --> SOC
    MIG --> LP[Liquidity Position (TBD)]
    LP --> DEX[Solana DEX / AMM]
    GOV --> GOV_PDA[Governance PDA]

    style CTR fill:#e8f4fd
    style LAUNCH fill:#fff3e0
    style BC fill:#fff3e0
    style BCPDA fill:#fff3e0
    style FEE fill:#e8f5e9
    style FEE_PDA fill:#e8f5e9
    style MIG fill:#e8f5e9
    style GOV fill:#e8f5e9
    style AGENT fill:#e8f5e9
    style ASSET fill:#e8f5e9
    style SOC fill:#e8f5e9
    style DEX fill:#e8f5e9
    style LP fill:#e8f5e9
```

---

## 3. Token Launch

```mermaid
flowchart TB
    A[Creator invokes Token Factory Program] --> B[SPL Token Mint Created]
    B --> C[Bonding Curve State PDA Initialized]
    C --> D[Bonding Curve wired to Fee Program]
    D --> E[Bonding Curve wired to Migration Program]
    E --> F[Trading active on curve]

    subgraph "Shared Programs"
        FP[Fee Program]
        MP[Migration Program]
        SR[Social Registry Program]
    end

    B -.-> FP
    C --> FP
    C --> MP
    FP --> SR

    style A fill:#e8f4fd
    style F fill:#e8f5e9
```

---

## 4. Bonding Curve

```mermaid
flowchart TD
    A[Buy Request] --> B{Amount > 0?}
    B -->|No| C[Reject: amount=0]
    B -->|Yes| D[Calculate Cost via price_to_buy]
    D --> E{Sufficient lamports?}
    E -->|No| F[Reject: insufficient lamports]
    E -->|Yes| G[Calculate Fee]
    G --> H[CPI to Fee Program: forward fee SOL]
    H --> I[Update sold counter in PDA]
    I --> J[Mint/transfer tokens to buyer ATA]
    J --> K{Amount >= min_tokens_out?}
    K -->|No| L[Reject: slippage]
    K -->|Yes| M[Refund excess lamports]
    M --> N[Trade complete]

    style A fill:#e8f4fd
    style N fill:#e8f5e9
```

---

## 5. Fee Flow

```mermaid
flowchart TD
    A[Trade Executed] --> B[Fee Collected]
    B --> C[Fee Program Receives 100%]

    C --> D[Protocol Share Taken First]
    D --> E[Remaining Net Fee]

    E --> F{Marketing Enabled?}
    F -->|Yes| G[20% Marketing Ledger]
    F -->|Yes| H[80% Destination Ledger]
    F -->|No| I[100% Destination Ledger]

    G --> J[Initial $299 DEX Screener]
    J --> K[Then Governance / Claim]
    H --> L[Claim via claim_destination]
    I --> L

    style A fill:#e8f4fd
    style B fill:#e8f4fd
    style C fill:#e8f4fd
    style D fill:#fff3e0
    style E fill:#fff3e0
    style F fill:#e8f5e9
    style G fill:#e8f5e9
    style H fill:#fff3e0
    style I fill:#e8f5e9
    style K fill:#e8f5e9
    style L fill:#e8f5e9
```

---

## 6. Fee Program Accounting

```mermaid
graph TD
    subgraph "Fee Program (shared, single deployment)"
        FP[Fee Program]
    end

    subgraph "Per-Token Accounting (PDA, isolated)"
        TA_A[Token A Fee PDA]
        TA_B[Token B Fee PDA]
        TA_C[Token C Fee PDA]
    end

    P_A[Protocol Balance] --> TA_A
    M_A[Marketing Balance] --> TA_A
    D_A[Destination Balance] --> TA_A
    C_A[Claimed Tracking] --> TA_A

    P_B[Protocol Balance] --> TA_B
    M_B[Marketing Balance] --> TA_B
    D_B[Destination Balance] --> TA_B
    C_B[Claimed Tracking] --> TA_B

    P_C[Protocol Balance] --> TA_C
    M_C[Marketing Balance] --> TA_C
    D_C[Destination Balance] --> TA_C
    C_C[Claimed Tracking] --> TA_C

    TA_A -.-> FP
    TA_B -.-> FP
    TA_C -.-> FP

    classDef ledger fill:#e8f5e9,stroke:#2e7d32;
    class FP ledger;
```

---

## 7. Destination Claim

```mermaid
flowchart LR
    A[Destination Balance PDA] --> B[claim_destination instruction]
    B --> C[Authorization Check]
    C --> D[Signer validated against PDA destination]
    D --> E[Funds Released via System Program CPI]

    style A fill:#e8f4fd
    style B fill:#fff3e0
    style C fill:#e8f5e9
    style D fill:#e8f5e9
    style E fill:#e8f5e9
```

---

## 8. Marketing

```mermaid
flowchart TD
    A[Net Fee] --> B{Marketing Enabled?}
    B -->|Yes| C[20% → Marketing Ledger PDA]
    B -->|Yes| D[80% → Destination Ledger PDA]
    B -->|No| E[100% → Destination Ledger PDA]

    C --> F[$299 reserved: DEX Screener Token Enhancement]
    F --> G[Auto-executes, no vote]
    G --> H[Remaining funds: governed]
    H --> I[Marketing Proposal]
    I --> J[Governance Approval]
    J --> K[Spend / Claim]

    D --> L[Claim via claim_destination]
    E --> L

    style A fill:#e8f4fd
    style B fill:#e8f5e9
    style C fill:#e8f5e9
    style D fill:#fff3e0
    style E fill:#e8f5e9
    style F fill:#fff3e0
    style G fill:#e8f5e9
    style K fill:#e8f5e9
    style L fill:#e8f5e9
```

---

## 9. Marketing Governance

```mermaid
sequenceDiagram
    participant Creator
    participant Trader
    participant Curve
    participant FP as Fee Program
    participant GOV as Governance Program
    participant V as Voters

    Note over Curve,FP: Bonding curve trading
    Trader->>Curve: buy with SOL
    Curve->>FP: CPI — forward fee
    FP->>FP: protocol share first; marketing 20%, destination 80%
    Note over FP: Marketing balance accrues

    Creator->>GOV: submit marketing proposal
    GOV->>V: snapshot top 100; voting 7 days
    V->>V: cast votes
    V->>GOV: tally
    GOV->>FP: CPI — execute approved spend
    FP->>FP: release marketing funds
```

---

## 10. CTO Governance

```mermaid
graph TD
    A[Proposal Created] --> B[Snapshot Taken]
    B --> C[Top 100 Holders Determined]
    C --> D[Voting Period 7 Days]
    D --> E{Quorum Met?}
    E -->|Yes| F{Approval Threshold Met?}
    E -->|No| G[Rejected]
    F -->|Yes| H[Approved On-Chain]
    F -->|No| G
    H --> I[Authorized Admin Review]
    I --> J[Fee Destination Updated]

    style A fill:#e8f4fd
    style B fill:#e8f4fd
    style C fill:#e8f5e9
    style D fill:#fff3e0
    style E fill:#e8f5e9
    style H fill:#e8f5e9
    style I fill:#fff3e0
    style J fill:#e8f5e9
```

---

## 11. Social Registry

```mermaid
graph TD
    A[Identity Request] --> B{Platform Select}
    B --> C[X/Twitter]
    B --> D[Discord]
    B --> E[GitHub]
    B --> F[Twitch]
    B --> G[Instagram]
    B --> H[YouTube]
    B --> I[Telegram]

    C --> J[Signature Verification]
    D --> K[Bot Confirmation]
    E --> L[GPG Signature]
    F --> M[OAuth Validation]
    G --> N[API Verification]
    H --> O[OAuth Consent]
    I --> P[Bot Confirmation]

    J --> Q[Identity Stored in PDA]
    K --> Q
    L --> Q
    M --> Q
    N --> Q
    O --> Q
    P --> Q

    Q --> R[Verified Identity]
    R --> S[Registered as Fee Destination]
    S --> T[Fee Program Accrues]
    T --> U[Manual Claim]

    style A fill:#e8f4fd
    style B fill:#e8f4fd
    style Q fill:#e8f5e9
    style R fill:#fff3e0
    style T fill:#fff3e0
    style U fill:#e8f5e9
```

---

## 12. Migration Flow

```mermaid
sequenceDiagram
    participant User
    participant BC as Bonding Curve Program
    participant BCPDA as Curve State PDA
    participant MP as Migration Program
    participant DEX as Solana DEX

    User->>BC: migrate instruction
    BC->>MP: CPI — verify_migration_condition
    MP->>BCPDA: Read state (sold, sol_balance, mint)
    MP->>BCPDA: Transfer all SOL via CPI
    MP->>DEX: CPI — add_liquidity (tokens + SOL)
    Note over DEX: Liquidity Position Created
    BC->>BCPDA: mark_graduated = true
```

---

## 13. Token Lifecycle

```mermaid
flowchart LR
    A[Creator invokes Token Factory] --> B[SPL Token Mint Created]
    B --> C[Bonding Curve State PDA Initialized]
    C --> D[Trading Active on Curve]

    D --> E[Migration Condition Met]
    E --> F[Migration Program creates DEX liquidity]
    F --> G[Bonding Curve State PDA Frozen]
    G --> H[DEX Trading Active]

    D --> I[Trade Fees to Fee Program PDA]
    I --> J[Destination Ledger PDA]
    I --> K[Marketing Ledger PDA]
    J --> L[Claim]
    K --> M[Claim / Governance]

    style A fill:#e8f4fd
    style H fill:#e8f5e9
    style L fill:#e8f5e9
    style M fill:#e8f5e9
```

---

## 14. Agent System

```mermaid
graph TD
    subgraph "FartBull Agent System"
        FP[FartBull Program]
        AP[Agent PDA]
        AC[Agent Configuration]
        APER[Agent Permissions]
        AST[Agent Status]
        ALA[Agent Associated Launches]
        ATR[Agent Treasury References]
        ASI[Agent Social Identity]
        AAUTO[Agent Automation Config]
    end

    FP --> AP
    AP --> AC
    AP --> APER
    AP --> AST
    AP --> ALA
    AP --> ATR
    AP --> ASI
    AP --> AAUTO

    ASI --> SR[Social Registry Program]

    subgraph "OFF-CHAIN"
        API[API / Orchestration]
    end

    API -->|"automation logic"| AP
    AP -->|"enforces permissions"| FP

    style FP fill:#e8f4fd
    style AP fill:#fff3e0
    style AC fill:#fff3e0
    style APER fill:#fff3e0
    style AST fill:#fff3e0
    style ALA fill:#fff3e0
    style ATR fill:#fff3e0
    style ASI fill:#fff3e0
    style AAUTO fill:#fff3e0
    style API fill:#e8f5e9
```

---

## 15. On-Chain vs Off-Chain

```mermaid
graph TD
    subgraph "ON-CHAIN (Authority)"
        FP[FartBull Program]
        AP[Agent PDA]
        BP[Bonding Curve PDA]
        FPDA[Fee Config PDA]
        ARP[Asset Registry PDA]
        SPP[Social Registry Program]
        GPP[Governance Program]
    end

    subgraph "OFF-CHAIN (Logic)"
        FE[Frontend]
        API[API Layer]
        ORCH[Orchestration]
        ORACLE[Oracle Provider]
        SOC_AUTH[Social Auth]
    end

    FE <-->|"RPC / CPI"| FP
    API <-->|"RPC"| FP
    ORCH <-->|"RPC"| AP
    ORACLE <-->|"Oracle abstraction"| ARP
    SOC_AUTH <-->|"Proof generation"| SPP

    style FP fill:#e8f5e9
    style AP fill:#e8f5e9
    style BP fill:#e8f5e9
    style FPDA fill:#e8f5e9
    style ARP fill:#e8f5e9
    style SPP fill:#e8f5e9
    style GPP fill:#e8f5e9
    style FE fill:#e8f4fd
    style API fill:#e8f4fd
    style ORCH fill:#e8f4fd
    style ORACLE fill:#f3e5f5
    style SOC_AUTH fill:#f3e5f5
```

---

*System diagrams last updated: August 2026*
*Next review: October 2026*
*Source of truth: [PROTOCOL.md](./PROTOCOL.md)*
