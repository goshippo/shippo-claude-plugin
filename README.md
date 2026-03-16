# Shippo Plugin for Claude Code

Ship packages with natural language. Compare carrier rates, generate labels, track packages, validate addresses, handle customs declarations, and process bulk shipments — all by chatting with Claude.

## What You Can Do

Just ask Claude things like:

- "Compare shipping rates for a 2lb package from San Francisco to New York"
- "Validate this address: 350 Fifth Ave, New York, NY 10118"
- "Ship this internationally to London — it's 2 books worth $30"
- "Track package SHIPPO_TRANSIT"
- "Process this CSV of shipments and generate labels for all of them"
- "What's the cheapest way to ship a 5lb box to Chicago?"

## Features

- **Address Validation** — Validate, parse, and standardize US and international addresses
- **Rate Shopping** — Compare rates across USPS, UPS, FedEx, DHL, and more in seconds
- **Label Generation** — Purchase domestic and international shipping labels
- **Package Tracking** — Track packages across all carriers with status history
- **Customs & International** — Automated customs declarations for international shipments
- **Batch Processing** — Process CSV files of shipments and generate labels in bulk

## Requirements

- A [Shippo account](https://apps.goshippo.com/join) (free to sign up)
- A Shippo API key (found in your [Shippo dashboard](https://apps.goshippo.com/settings/api))
- Claude Code or another MCP-compatible client

## Setup

### 1. Set your API key

```bash
export SHIPPO_API_KEY="ShippoToken shippo_test_xxxxx"
```

Add this to your `~/.zshrc` or `~/.bashrc` to persist across sessions.

### 2. Verify it's working

Ask Claude:

> "List my Shippo carrier accounts"

If configured correctly, Claude will return your connected carriers.

## Pricing

- **This plugin**: Free
- **Shippo account**: Free to sign up
- **Test mode**: Free — create shipments, get rates, validate addresses at no cost
- **Live labels**: You only pay when you purchase a shipping label (at discounted Shippo rates)

## Links

- [Shippo Documentation](https://docs.goshippo.com)
- [Shippo API Reference](https://docs.goshippo.com/docs/api_concepts/apiversioning/)
- [MCP Server (npm)](https://www.npmjs.com/package/@shippo/shippo-mcp)
