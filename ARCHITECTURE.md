# Shulam System Architecture

## Ecosystem Overview

Shulam is an x402 payment facilitator that enables internet-native USDC payments on Base L2. The buyer signs a gasless EIP-3009 authorization, the facilitator verifies and settles on-chain, and the merchant receives funds minus a configurable fee.

```
                        Shulam Ecosystem
 ┌───────────────────────────────────────────────────────────────┐
 │                                                               │
 │   CLIENTS                 SERVICES                CHAIN       │
 │                                                               │
 │  ┌──────────┐         ┌──────────────┐        ┌──────────┐   │
 │  │ buyer-sdk│────────▶│  facilitator │───────▶│contracts │   │
 │  └──────────┘         │              │        │(Escrow,  │   │
 │                       │  /verify     │        │ Vault)   │   │
 │  ┌──────────┐         │  /settle     │        └──────────┘   │
 │  │merchant- │────────▶│  /status     │             │         │
 │  │dashboard │         │  /webhooks   │        ┌──────────┐   │
 │  └──────────┘         │  /escrow     │        │   USDC   │   │
 │                       │  /cashback   │        │ (Base L2)│   │
 │  ┌──────────┐         └──────┬───────┘        └──────────┘   │
 │  │ website  │                │                               │
 │  └──────────┘         ┌──────▼───────┐                       │
 │                       │  compliance  │                       │
 │  ┌──────────┐         │  OFAC / KYT  │                       │
 │  │  docs    │         │  Velocity    │                       │
 │  └──────────┘         └──────────────┘                       │
 │                                                               │
 │  ┌──────────┐                                                │
 │  │  souls   │  (identity / reputation layer)                 │
 │  └──────────┘                                                │
 │                                                               │
 └───────────────────────────────────────────────────────────────┘
```

---

## Repository Map

| Repo | Type | Purpose | Depends On |
|------|------|---------|------------|
| **facilitator** | Service | Core x402 payment processing: verify, settle, webhooks, monitoring | contracts, compliance |
| **contracts** | On-chain | Solidity: ShulamEscrow, CashbackVault, DisputeResolver | USDC (Base L2) |
| **compliance** | Service | OFAC SDN screening, Chainalysis KYT, velocity monitoring, SAR | facilitator (called by) |
| **buyer-sdk** | SDK | TypeScript SDK for wallets/dApps + headless agent mode + MCP tools | facilitator (API client), souls (reputation) |
| **merchant-dashboard** | Frontend | Web UI: webhook management, transaction history, analytics, settings | facilitator (API client) |
| **souls** | On-chain + Service | Agent orchestration (18 Apostles), SBT identity, reputation oracle | contracts, facilitator |
| **docs** | Content | Mintlify docs at docs.shulam.io: API reference, guides, protocol specs | All repos (documents them) |
| **website** | Frontend | Marketing site at shulam.io | None |
| **admin-dashboard** | Frontend | Internal ops: system health, fee tracking, escrow recovery, compliance review | facilitator, compliance |
| **merchant-sdk** | SDK | Express/Next.js/Fastify middleware, project scaffolder, x402 manifest | facilitator (API client) |
| **testkit** | Dev tooling | Mock facilitator, test wallets, payment simulation, CLI tools | facilitator, buyer-sdk, merchant-sdk |

---

## Funds Flow (Direct Settlement with Fee)

```
Buyer Wallet          Facilitator Wallet          Merchant Wallet
     │                       │                          │
     │── EIP-3009 sign ──▶   │                          │
     │   (to=facilitator,    │                          │
     │    value=100 USDC)    │                          │
     │                       │                          │
     │   Txn 1: transferWithAuthorization               │
     │   100 USDC ─────────▶ │                          │
     │                       │                          │
     │                       │  Txn 2: USDC.transfer    │
     │                       │  99.25 USDC ───────────▶ │
     │                       │                          │
     │                       │  0.75 USDC retained      │
     │                       │  (facilitator fee)       │
     │                       │                          │
     │   Cashback: 0.25 USDC │                          │
     │ ◀──────────────────── │                          │
     │   (CashbackVault)     │                          │
```

**Key constraint:** The buyer signs `(from, to=facilitator, value)` immutably via EIP-3009. The on-chain `transferWithAuthorization` must match exactly. The facilitator then forwards `value - fee` to the merchant in a second transaction.

---

## Data Flow: End-to-End Payment

```
Step  Actor               Action                              Repo
────  ──────────────────  ──────────────────────────────────  ──────────────
 1    Buyer (browser)     Clicks "Pay" on merchant site       buyer-sdk
 2    Merchant server     Returns HTTP 402 + payment details  merchant-sdk*
 3    buyer-sdk           Signs EIP-3009 authorization         buyer-sdk
 4    Buyer (browser)     Resubmits with X-PAYMENT header     buyer-sdk
 5    Merchant server     POST /verify → facilitator          facilitator
 6    Facilitator         Validates signature + time bounds   facilitator
 7    Merchant server     POST /settle → facilitator          facilitator
 8    Facilitator         OFAC screen (from, to, merchant)    compliance
 9    Facilitator         Txn 1: buyer → facilitator (USDC)   contracts
10    Facilitator         Txn 2: facilitator → merchant       contracts
11    Facilitator         Returns { settled, fee, netAmount } facilitator
12    Merchant server     Delivers resource to buyer          (merchant)
13    Facilitator         Dispatches webhook                  facilitator
14    Facilitator         Credits cashback to vault           contracts
15    merchant-dashboard  Shows transaction in dashboard      merchant-dashboard
```

*merchant-sdk provides the 402 response builder and facilitator client

---

## Data Flow: Agent-to-Agent Payment

```
Step  Actor               Action                              Repo
────  ──────────────────  ──────────────────────────────────  ──────────────
 1    Buyer agent         GET /.well-known/x402-manifest.json buyer-sdk/agent
 2    Merchant API        Returns manifest (endpoints, prices) merchant-sdk
 3    Buyer agent         Compares providers (price + reputation) buyer-sdk/agent
 4    Buyer agent         Checks reputation via ReputationOracle souls
 5    Buyer agent         Checks budget (per-request, session)  buyer-sdk/agent
 6    Buyer agent         GET /api/data (no payment header)    buyer-sdk/agent
 7    Merchant API        Returns HTTP 402 + payment details   merchant-sdk
 8    Buyer agent         Signs EIP-3009 (headless, private key) buyer-sdk/agent
 9    Buyer agent         Retries GET /api/data + X-PAYMENT    buyer-sdk/agent
10    Merchant API        POST /verify → facilitator           merchant-sdk
11    Facilitator         Validates signature + OFAC screen    facilitator
12    Merchant API        POST /settle → facilitator           merchant-sdk
13    Facilitator         Two-txn settlement (buyer→fac→merch) facilitator
14    Facilitator         Updates ReputationOracle             souls
15    Merchant API        Returns resource to buyer agent      merchant-sdk
16    Buyer agent         Logs payment to spending history     buyer-sdk/agent
```

**AI model flow (MCP):** Steps 1-16 are orchestrated by the MCP server (`@shulam/buyer-sdk/mcp`). The AI calls `shulam_pay` or `shulam_discover` tools, and the MCP server delegates to `ShulamAgent` internally.

---

## Network Topology

```
                    Internet
                       │
          ┌────────────┼────────────┐
          │            │            │
     ┌────▼────┐  ┌────▼────┐  ┌───▼────┐
     │ buyer-  │  │merchant-│  │website │
     │  sdk    │  │dashboard│  │        │
     │(browser)│  │ (Next)  │  │(Next)  │
     └────┬────┘  └────┬────┘  └────────┘
          │            │
          └──────┬─────┘
                 │ HTTPS
          ┌──────▼──────┐
          │ facilitator │ ◀── Express, port 3000
          │             │
          └──┬──────┬───┘
             │      │
    ┌────────▼┐  ┌──▼────────┐
    │complianc│  │ Coinbase  │
    │  (OFAC) │  │  CDP SDK  │
    └─────────┘  └─────┬─────┘
                       │ JSON-RPC
                 ┌─────▼─────┐
                 │  Base L2  │
                 │  (USDC,   │
                 │  Escrow,  │
                 │  Vault)   │
                 └───────────┘
```

---

## Shared Conventions

All TypeScript repos follow:
- ES modules with `.js` import extensions
- TypeScript: ES2022, NodeNext, strict
- Module structure: `types.ts → store.ts → service.ts → routes.ts → index.ts`
- Config via Zod schema + dotenv
- In-memory stores implementing interfaces (swappable for DB)
- Vitest with `restoreMocks: true`
- Router factory functions with dependency injection

Solidity repos follow:
- Foundry for compile, test, deploy
- OpenZeppelin for access control, reentrancy guards
- Deploy scripts in `script/`
