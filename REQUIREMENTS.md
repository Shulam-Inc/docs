# Shulam Cross-Repo Requirements

This document defines user journeys that span multiple repositories and maps each requirement to the repo that owns its implementation. Each repo's own `PLAN.md` contains the implementation-level milestones and Gherkin scenarios.

---

## How This Document Works

- **User journeys** describe end-to-end flows from the user's perspective
- **Requirement slices** break each journey into per-repo responsibilities
- Each slice references the owning repo and links to its PLAN.md
- When a requirement changes, update this document first, then propagate to per-repo PLANs

---

## UJ-1: Buyer Makes a Payment

> As a buyer, I want to pay for a resource with USDC so that I receive the resource instantly.

| Step | Description | Owner | Status |
|------|-------------|-------|--------|
| UJ-1.1 | Merchant server returns HTTP 402 with payment details | merchant (external) | N/A |
| UJ-1.2 | buyer-sdk constructs EIP-3009 authorization from 402 response | **buyer-sdk** | Planned |
| UJ-1.3 | buyer-sdk signs authorization off-chain (gasless) | **buyer-sdk** | Planned |
| UJ-1.4 | buyer-sdk encodes X-PAYMENT header and resubmits request | **buyer-sdk** | Planned |
| UJ-1.5 | Facilitator verifies signature, time bounds, nonce | **facilitator** | Done |
| UJ-1.6 | Facilitator screens addresses (from, to, merchant) via OFAC | **compliance** | Done |
| UJ-1.7 | Facilitator executes Txn 1: buyer → facilitator (transferWithAuthorization) | **facilitator** | Done |
| UJ-1.8 | Facilitator executes Txn 2: facilitator → merchant (USDC.transfer for netAmount) | **facilitator** | Done |
| UJ-1.9 | Facilitator returns { settled, fee, netAmount, merchantTxHash } | **facilitator** | Done |
| UJ-1.10 | Facilitator dispatches `payment.completed` webhook with fee fields | **facilitator** | Done |
| UJ-1.11 | Facilitator credits cashback to buyer's CashbackVault balance | **facilitator** + **contracts** | Done |
| UJ-1.12 | merchant-dashboard displays transaction with net amount | **merchant-dashboard** | Planned |

---

## UJ-2: Buyer Makes an Escrow Payment

> As a buyer, I want to pay into escrow so that funds are held until the merchant delivers.

| Step | Description | Owner | Status |
|------|-------------|-------|--------|
| UJ-2.1 | buyer-sdk signs authorization with to=escrow contract | **buyer-sdk** | Planned |
| UJ-2.2 | Facilitator calls POST /settle with mode="deferred" | **facilitator** | Done |
| UJ-2.3 | Facilitator deposits USDC into ShulamEscrow contract | **facilitator** + **contracts** | Done |
| UJ-2.4 | Merchant releases escrow after delivery | **facilitator** | Done |
| UJ-2.5 | Buyer requests refund if merchant fails to deliver | **facilitator** | Done |
| UJ-2.6 | DisputeResolver handles contested escrows | **contracts** | Planned |
| UJ-2.7 | merchant-dashboard shows escrow status and release/refund controls | **merchant-dashboard** | Planned |

---

## UJ-3: Merchant Integrates Shulam

> As a merchant, I want to integrate Shulam payments so that I can accept USDC on my site.

| Step | Description | Owner | Status |
|------|-------------|-------|--------|
| UJ-3.1 | Merchant reads quickstart guide on docs.shulam.io | **docs** | Planned |
| UJ-3.2 | Merchant signs up on merchant-dashboard and gets API keys | **merchant-dashboard** | Planned |
| UJ-3.3 | Merchant installs server-side SDK or calls facilitator API directly | **docs** | Planned |
| UJ-3.4 | Merchant configures webhook endpoint via dashboard | **merchant-dashboard** | Planned |
| UJ-3.5 | Merchant returns HTTP 402 responses with payment requirements | merchant (external) | N/A |
| UJ-3.6 | Merchant calls POST /verify then POST /settle on facilitator | **facilitator** | Done |
| UJ-3.7 | Merchant receives webhook notifications for payment events | **facilitator** | Done |
| UJ-3.8 | Merchant views transaction history and analytics on dashboard | **merchant-dashboard** | Planned |

---

## UJ-4: Buyer Claims Cashback

> As a buyer, I want to withdraw accumulated cashback so that I receive USDC in my wallet.

| Step | Description | Owner | Status |
|------|-------------|-------|--------|
| UJ-4.1 | buyer-sdk reads cashback balance from CashbackVault | **buyer-sdk** + **contracts** | Planned |
| UJ-4.2 | Buyer calls POST /cashback/withdraw on facilitator | **facilitator** | Done |
| UJ-4.3 | Facilitator calls vault.withdraw() to send USDC to buyer | **facilitator** + **contracts** | Done |
| UJ-4.4 | Facilitator dispatches `cashback.withdrawn` webhook | **facilitator** | Done |

---

## UJ-5: Compliance Screens a Payment

> As a compliance officer, I want all payments screened so that sanctioned entities cannot transact.

| Step | Description | Owner | Status |
|------|-------------|-------|--------|
| UJ-5.1 | Facilitator calls compliance service before settlement | **facilitator** | Done |
| UJ-5.2 | OFAC SDN list screening of from, to, merchantAddress | **compliance** | Done |
| UJ-5.3 | KYT risk scoring via Chainalysis | **compliance** | Planned |
| UJ-5.4 | Velocity monitoring detects structuring patterns | **compliance** | Done |
| UJ-5.5 | AlertManager raises alerts for flagged transactions | **compliance** | Done |
| UJ-5.6 | SAR generation for suspicious activity | **compliance** | Done |
| UJ-5.7 | Blocked payment returns 403 and dispatches `payment.blocked` webhook | **facilitator** | Done |

---

## UJ-6: Ops Monitors the System

> As an operations engineer, I want to monitor payment health so that I can respond to failures.

| Step | Description | Owner | Status |
|------|-------------|-------|--------|
| UJ-6.1 | GET /metrics returns transaction counts, success rate, fees collected | **facilitator** | Done |
| UJ-6.2 | GET /metrics/alerts returns unresolved alerts | **facilitator** | Done |
| UJ-6.3 | POST /metrics/alerts/:id/resolve marks alert resolved | **facilitator** | Done |
| UJ-6.4 | Critical alerts raised for settlement failures and partial failures | **facilitator** | Done |
| UJ-6.5 | merchant-dashboard shows ops view with system health | **merchant-dashboard** | Planned |

---

## Per-Repo Requirement Summary

### facilitator (`Shulam-Inc/facilitator`)

| Requirement | Source Journey | Status |
|-------------|---------------|--------|
| EIP-3009 signature verification | UJ-1.5 | Done (M2) |
| Direct settlement with fee deduction | UJ-1.7, UJ-1.8 | Done (M8) |
| Escrow deposit/release/refund | UJ-2.2-2.5 | Done (M4) |
| OFAC screening integration | UJ-5.1, UJ-5.7 | Done (M4) |
| Webhook dispatch for all events | UJ-1.10, UJ-3.7 | Done (M5) |
| Transaction status tracking | UJ-6.1-6.4 | Done (M6) |
| Cashback credit/withdraw | UJ-4.2-4.4 | Done (M4) |
| Fee fields in response/webhook/status | UJ-1.9, UJ-1.10 | Done (M8) |
| totalFeesCollected in metrics | UJ-6.1 | Done (M8) |

### contracts (`Shulam-Inc/contracts`)

| Requirement | Source Journey | Status |
|-------------|---------------|--------|
| ShulamEscrow (deposit, release, refund) | UJ-2.3-2.5 | Done |
| CashbackVault (credit, withdraw, balanceOf) | UJ-4.1, UJ-4.3 | Done |
| DisputeResolver (open, resolve, escalate) | UJ-2.6 | Planned |
| Deploy scripts for Base Sepolia + Mainnet | All | Done (Sepolia) |

### compliance (`Shulam-Inc/compliance`)

| Requirement | Source Journey | Status |
|-------------|---------------|--------|
| OFAC SDN list screening | UJ-5.2 | Done |
| Chainalysis KYT integration | UJ-5.3 | Planned |
| Velocity monitoring | UJ-5.4 | Done |
| Alert management | UJ-5.5 | Done |
| SAR generation | UJ-5.6 | Done |

### buyer-sdk (`Shulam-Inc/buyer-sdk`)

| Requirement | Source Journey | Status |
|-------------|---------------|--------|
| Parse HTTP 402 response into payment details | UJ-1.2 | Planned |
| Construct EIP-3009 authorization | UJ-1.3 | Planned |
| Sign authorization via wallet (ethers/viem) | UJ-1.3 | Planned |
| Encode X-PAYMENT header (base64url) | UJ-1.4 | Planned |
| Read cashback balance from vault | UJ-4.1 | Planned |
| React components (PayButton, CashbackBadge) | UJ-1.2 | Planned |

### merchant-dashboard (`Shulam-Inc/merchant-dashboard`)

| Requirement | Source Journey | Status |
|-------------|---------------|--------|
| Merchant signup and API key management | UJ-3.2 | Planned |
| Webhook endpoint configuration UI | UJ-3.4 | Planned |
| Transaction history with fee/netAmount display | UJ-1.12, UJ-3.8 | Planned |
| Escrow status view with release/refund controls | UJ-2.7 | Planned |
| Analytics dashboard (volume, fees, success rate) | UJ-3.8 | Planned |
| System health / ops view | UJ-6.5 | Planned |

### souls (`Shulam-Inc/souls`)

| Requirement | Source Journey | Status |
|-------------|---------------|--------|
| Soul-bound token issuance | — | Planned |
| Identity attestation (KYC link) | — | Planned |
| Reputation scoring based on payment history | — | Planned |

### docs (`Shulam-Inc/docs`)

| Requirement | Source Journey | Status |
|-------------|---------------|--------|
| Quickstart guide (< 15 min to first payment) | UJ-3.1 | Planned |
| API reference for all facilitator endpoints | UJ-3.3 | Planned |
| buyer-sdk installation and usage guide | UJ-1.2 | Planned |
| x402 protocol explainer | UJ-1 | Planned |
| Webhook integration guide | UJ-3.7 | Planned |
| Update payment-flow.mdx for fee model | UJ-1.8 | TODO |
| Update concepts.mdx settlement diagram | UJ-1.8 | TODO |

### website (`Shulam-Inc/website`)

| Requirement | Source Journey | Status |
|-------------|---------------|--------|
| Landing page with value proposition | — | Planned |
| Pricing page showing fee structure | — | Planned |
| Developer CTA linking to docs.shulam.io | — | Planned |

### admin-dashboard (`Shulam-Inc/admin-dashboard`)

| Requirement | Source Journey | Status |
|-------------|---------------|--------|
| System health metrics view (success rate, volume, fees) | UJ-6.1 | Planned |
| Alert manager with resolve action | UJ-6.2, UJ-6.3 | Planned |
| Fee & revenue tracking dashboard | UJ-6.1 | Planned |
| Transaction explorer (search, filter, detail) | UJ-6 | Planned |
| Escrow management (manual release/refund/recovery) | UJ-2.7 | Planned |
| Compliance review (screening results, blocked tx, SARs) | UJ-5 | Planned |

---

## Future Repos (Candidates)

| Repo | Rationale | Priority |
|------|-----------|----------|
| **merchant-sdk** | Server-side SDK (Express/Fastify middleware, webhook verification, payment request helpers). Currently merchants call the facilitator API directly — an SDK would reduce integration friction. | Medium |

---

## Changelog

| Date | Change |
|------|--------|
| 2026-02-13 | Added admin-dashboard repo and requirements. |
| 2026-02-12 | Initial cross-repo requirements. Fee deduction (M8) complete in facilitator. |
