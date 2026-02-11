# Docs Project Plan

## Overview

Developer documentation for integrating Shulam payments. Includes API reference, integration guides, and x402 protocol explainers.

**Live site:** docs.shulam.io

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         DOCS                                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────┐        │
│  │              Mintlify / Nextra                   │        │
│  │                                                  │        │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐        │        │
│  │  │  Guides  │ │   API    │ │   SDK    │        │        │
│  │  │          │ │Reference │ │  Docs    │        │        │
│  │  └──────────┘ └──────────┘ └──────────┘        │        │
│  │                                                  │        │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐        │        │
│  │  │ x402     │ │ Examples │ │   FAQ    │        │        │
│  │  │ Protocol │ │          │ │          │        │        │
│  │  └──────────┘ └──────────┘ └──────────┘        │        │
│  └─────────────────────────────────────────────────┘        │
│                          │                                   │
│                          ▼                                   │
│  ┌─────────────────────────────────────────────────┐        │
│  │              Vercel Deployment                   │        │
│  │              docs.shulam.io                      │        │
│  └─────────────────────────────────────────────────┘        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Content Structure

```
docs/
├── index.mdx                    # Homepage
├── getting-started/
│   ├── introduction.mdx
│   ├── quickstart.mdx
│   ├── concepts.mdx
│   └── account-setup.mdx
├── guides/
│   ├── merchant-integration.mdx
│   ├── buyer-sdk.mdx
│   ├── webhooks.mdx
│   ├── testing.mdx
│   ├── subscriptions.mdx
│   └── migration-from-stripe.mdx
├── api-reference/
│   ├── authentication.mdx
│   ├── verify.mdx
│   ├── settle.mdx
│   ├── status.mdx
│   ├── webhooks.mdx
│   └── errors.mdx
├── sdk/
│   ├── installation.mdx
│   ├── shulam-payment.mdx
│   ├── react-components.mdx
│   ├── typescript-types.mdx
│   └── examples.mdx
├── x402-protocol/
│   ├── overview.mdx
│   ├── eip-3009.mdx
│   ├── payment-flow.mdx
│   └── security.mdx
├── resources/
│   ├── faq.mdx
│   ├── glossary.mdx
│   ├── changelog.mdx
│   └── support.mdx
└── mint.json                    # Mintlify config
```

---

## Milestones

### M1: Foundation
- [ ] Mintlify/Nextra setup
- [ ] Basic navigation structure
- [ ] Shulam branding
- [ ] Vercel deployment

### M2: Getting Started
- [ ] Introduction page
- [ ] Quickstart guide
- [ ] Concepts overview
- [ ] Account setup

### M3: Integration Guides
- [ ] Merchant integration guide
- [ ] Buyer SDK integration
- [ ] Webhook setup
- [ ] Testing guide

### M4: API Reference
- [ ] Facilitator API docs
- [ ] Authentication docs
- [ ] Error codes
- [ ] Rate limits

### M5: SDK Documentation
- [ ] Core SDK reference
- [ ] React components
- [ ] TypeScript types
- [ ] Examples

### M6: Advanced Topics
- [ ] x402 protocol deep-dive
- [ ] EIP-3009 explainer
- [ ] Multi-chain support
- [ ] Security best practices

---

## User Stories (Gherkin)

### Epic 1: Getting Started

```gherkin
Feature: Developer Onboarding
  As a developer new to Shulam
  I want clear getting started documentation
  So that I can integrate quickly

  Scenario: Find quickstart guide
    Given I am on docs.shulam.io
    When I look at the homepage
    Then I see a prominent "Get Started" button
    And clicking it takes me to the quickstart guide

  Scenario: Complete quickstart in 15 minutes
    Given I am on the quickstart page
    When I follow the steps:
      | step | description              |
      | 1    | Create merchant account  |
      | 2    | Get API keys             |
      | 3    | Install SDK              |
      | 4    | Add payment button       |
      | 5    | Test payment             |
    Then I have a working test integration
    And total time is under 15 minutes

  Scenario: Understand key concepts
    Given I am new to Shulam
    When I read the Concepts page
    Then I understand:
      | concept      | explained |
      | x402 protocol| yes       |
      | facilitator  | yes       |
      | USDC         | yes       |
      | cashback     | yes       |
      | EIP-3009     | yes       |
```

### Epic 2: API Reference

```gherkin
Feature: API Documentation
  As a developer
  I want comprehensive API docs
  So that I can use all endpoints correctly

  Scenario: Find endpoint documentation
    Given I need to use the /verify endpoint
    When I navigate to API Reference > Verify
    Then I see:
      | section          | present |
      | endpoint_url     | yes     |
      | http_method      | yes     |
      | authentication   | yes     |
      | request_body     | yes     |
      | response_format  | yes     |
      | error_codes      | yes     |
      | example_request  | yes     |
      | example_response | yes     |

  Scenario: Try API in documentation
    Given I am viewing an endpoint doc
    When I click "Try it"
    Then an interactive API console opens
    And I can enter parameters
    And I can send a test request
    And I see the response

  Scenario: Copy code examples
    Given I am viewing an endpoint doc
    When I see the code example
    Then I can switch between languages:
      | language   | available |
      | curl       | yes       |
      | JavaScript | yes       |
      | Python     | yes       |
      | Go         | yes       |
    And I can copy with one click
```

### Epic 3: SDK Documentation

```gherkin
Feature: SDK Documentation
  As a developer using the SDK
  I want detailed SDK docs
  So that I can use all features

  Scenario: Find SDK installation
    Given I want to install the SDK
    When I navigate to SDK > Installation
    Then I see installation commands for:
      | package_manager | command                        |
      | npm             | npm install @shulam/buyer-sdk  |
      | yarn            | yarn add @shulam/buyer-sdk     |
      | pnpm            | pnpm add @shulam/buyer-sdk     |

  Scenario: Understand SDK methods
    Given I am viewing the SDK reference
    When I look at ShulamPayment class
    Then I see documentation for:
      | method          | documented |
      | constructor     | yes        |
      | connect         | yes        |
      | pay             | yes        |
      | getCashback     | yes        |
      | disconnect      | yes        |
    And each method shows parameters and return types

  Scenario: Find React component props
    Given I am using React components
    When I view ShulamButton documentation
    Then I see all props with types and descriptions
```

### Epic 4: Search & Navigation

```gherkin
Feature: Documentation Search
  As a developer
  I want to search documentation
  So that I can find answers quickly

  Scenario: Search for topic
    Given I am on any docs page
    When I press Cmd+K
    Then the search modal opens
    And I can type my query
    And I see relevant results instantly

  Scenario: Navigate via sidebar
    Given I am reading documentation
    When I look at the sidebar
    Then I see organized sections
    And I can expand/collapse sections
    And current page is highlighted

  Scenario: Use breadcrumbs
    Given I am on a nested page
    When I look at the breadcrumbs
    Then I see my location in the hierarchy
    And I can click to navigate up
```

---

## Environment Variables

```bash
# Mintlify
MINTLIFY_API_KEY=

# Analytics
NEXT_PUBLIC_POSTHOG_KEY=

# Search
ALGOLIA_APP_ID=
ALGOLIA_API_KEY=
```

---

## Agent Instructions

### For Matthew (Content)
1. Own all documentation content
2. Write in clear, concise language
3. Include code examples for everything
4. Keep docs in sync with code changes

### For Luke (Scribe)
1. Maintain changelog
2. Update docs when APIs change
3. Review for accuracy
4. Manage versioning

### For Thaddaeus (Designer)
1. Custom Mintlify theme
2. Diagrams and illustrations
3. Code syntax highlighting
4. Mobile responsiveness
