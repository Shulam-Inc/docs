# Shulam Documentation

Developer documentation for integrating Shulam payments.

🌐 **Live site:** [docs.shulam.io](https://docs.shulam.io)

## Overview

This repository contains the source for Shulam's developer documentation, including API references, integration guides, and x402 protocol explainers.

## Documentation Structure

```
docs/
├── getting-started/
│   ├── introduction.md
│   ├── quickstart.md
│   └── concepts.md
├── integration/
│   ├── merchant-setup.md
│   ├── buyer-sdk.md
│   ├── webhooks.md
│   └── testing.md
├── api-reference/
│   ├── authentication.md
│   ├── transactions.md
│   ├── payouts.md
│   └── webhooks.md
├── x402-protocol/
│   ├── overview.md
│   ├── eip-3009.md
│   └── multi-chain.md
└── guides/
    ├── migration-from-stripe.md
    ├── cost-comparison.md
    └── compliance.md
```

## Tech Stack

- [Mintlify](https://mintlify.com) or [Nextra](https://nextra.site)
- MDX for interactive examples
- Deployed on Vercel

## Local Development

```bash
# Install dependencies
npm install

# Start dev server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

## Contributing

1. Create a branch for your changes
2. Edit markdown files in `docs/`
3. Preview locally with `npm run dev`
4. Submit PR for review

## Style Guide

- Use active voice
- Include code examples for every endpoint
- Add "Try it" buttons where possible
- Keep pages focused — one concept per page

## Deployment

Pushes to `main` auto-deploy to docs.shulam.io via Vercel.

## License

MIT
