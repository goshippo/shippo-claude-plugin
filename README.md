# Shippo Plugin for Claude Code

A [Claude Code plugin](https://code.claude.com/docs/en/plugins) that gives Claude the ability to ship packages. Once installed, Claude can validate addresses, compare carrier rates, generate shipping labels, track packages, handle customs declarations, and process bulk shipments — all through natural conversation.

## Quick Start

**1. Install Claude Code** (if you don't have it)

Download from [code.claude.com](https://code.claude.com).

**2. Clone this repo**

```bash
git clone https://github.com/goshippo/shippo-claude-plugin.git
```

**3. Set your Shippo API key**

Get your key from the [Shippo dashboard](https://apps.goshippo.com/settings/api) (Settings > API), then run:

```bash
export SHIPPO_API_KEY="ShippoToken shippo_test_xxxxx"
```

Replace `shippo_test_xxxxx` with your actual key. Add this line to your `~/.zshrc` or `~/.bashrc` to persist it.

Don't have a Shippo account? [Sign up free](https://apps.goshippo.com/join) — test mode costs nothing.

**4. Launch Claude Code with the plugin**

```bash
claude --plugin-dir /path/to/shippo-claude-plugin
```

Replace `/path/to/` with the actual path where you cloned the repo (e.g., `~/Desktop/shippo-claude-plugin`).

> **Note:** Shippo provides managed carrier accounts for major carriers (USPS, UPS, FedEx, DHL) by default. You can get rates and buy labels immediately — no carrier setup needed.

**5. Try it**

Just ask Claude what you need:

> Compare rates for a 2lb package from San Francisco to New York

Claude will automatically use the right Shippo skill, connect to the API, and present your options. You can also invoke skills directly: `/shippo:rate-shopping`, `/shippo:label-purchase`, `/shippo:tracking`, etc.

## What You Can Do

- "Compare shipping rates for a 2lb package from San Francisco to New York"
- "Validate this address: 350 Fifth Ave, New York, NY 10118"
- "Ship this internationally to London — it's 2 books worth $30"
- "Track package SHIPPO_TRANSIT"
- "Process this CSV of shipments and generate labels for all of them"
- "What's the cheapest way to ship a 5lb box to Chicago?"

| Capability | Description |
|---|---|
| **Address Validation** | Validate, parse, and standardize US and international addresses |
| **Rate Shopping** | Compare rates across USPS, UPS, FedEx, DHL, and 30+ carriers |
| **Label Generation** | Purchase domestic and international shipping labels |
| **Package Tracking** | Track packages across all carriers with full status history |
| **Customs & International** | Automated customs declarations, HS code guidance, EEL/PFC handling |
| **Batch Processing** | Process CSV files of shipments and generate labels in bulk |

## How It Works

This plugin bundles two things:

- **Skills** — Six task-specific skills that teach Claude the workflows, API sequences, and best practices for shipping: address validation, rate shopping, label purchase, tracking, batch processing, and shipping analysis. Claude automatically picks the right skill based on what you ask.
- **An MCP server connection** — Connects Claude to Shippo's hosted API so it can actually execute those workflows (create shipments, buy labels, track packages, etc.). There's no local server to run — the connection is configured automatically when you load the plugin.

The skills teach Claude *how* to ship. The MCP server gives Claude the *tools* to ship. Together, they make Claude a shipping expert.

> The MCP server is currently hosted by Speakeasy (Gram). The connection URL is configured in `.mcp.json`.

## Pricing

- **This plugin**: Free and open source (MIT)
- **Shippo account**: Free to sign up
- **Test mode**: Free — create shipments, get rates, validate addresses at no cost
- **Live labels**: Charged at [Shippo's discounted rates](https://goshippo.com/pricing) only when you purchase a label

## Troubleshooting

**Common issues:**

- **"Invalid API key"** — Check that `SHIPPO_API_KEY` is set and includes the `ShippoToken ` prefix.
- **"No rates returned"** — Most common cause is an unvalidated address. Validate both addresses first. Also check parcel dimensions are reasonable.
- **"Carrier account not found"** — Shippo provides managed carrier accounts by default. You don't need to set up your own carrier credentials to get started.
- **"Test vs live confusion"** — Test keys (`shippo_test_*`) are free and safe to experiment with. Live keys (`shippo_live_*`) create real labels that cost money.

## Repo Structure

```
shippo-claude-plugin/
├── .claude-plugin/
│   └── plugin.json        # Plugin manifest (name, version, metadata)
├── .mcp.json               # MCP server connection (Shippo API — hosted, nothing to run)
├── skills/
│   ├── address-validation/  # /shippo:address-validation
│   ├── rate-shopping/       # /shippo:rate-shopping
│   ├── label-purchase/      # /shippo:label-purchase
│   ├── tracking/            # /shippo:tracking
│   ├── batch-shipping/      # /shippo:batch-shipping
│   ├── shipping-analysis/   # /shippo:shipping-analysis
│   └── shippo/
│       └── references/      # Shared knowledge docs (carriers, customs, errors, etc.)
├── LICENSE
└── README.md
```

## Terminology

| Term | What it means in this repo |
|---|---|
| **Plugin** | This entire repo — the installable package. Bundles skills and MCP server config together so one `--plugin-dir` flag sets up everything. |
| **Skills** | Six task-specific skills (`/shippo:rate-shopping`, `/shippo:label-purchase`, etc.). Markdown instructions that teach Claude how to perform shipping tasks. Claude picks the right one automatically. |
| **MCP Server** | The hosted service that gives Claude access to Shippo's API tools. Configured in `.mcp.json`, runs remotely — nothing to install locally. |

For more on these concepts, see Anthropic's docs on [plugins](https://code.claude.com/docs/en/plugins), [skills](https://code.claude.com/docs/en/skills), and [MCP servers](https://code.claude.com/docs/en/mcp).

## Links

- [Shippo Documentation](https://docs.goshippo.com)
- [Shippo API Reference](https://docs.goshippo.com/docs/api_concepts/apiversioning/)
- [MCP Server (npm)](https://www.npmjs.com/package/@shippo/shippo-mcp)
- [Claude Code Docs — Plugins](https://code.claude.com/docs/en/plugins)
- [Official MCP Registry](https://registry.modelcontextprotocol.io)
- [Agent Skills Standard](https://agentskills.io)

## Privacy

> This plugin transmits shipping data (addresses, parcel details, tracking numbers) to the Shippo API for processing. If you are handling customer data, ensure your use complies with your privacy obligations.

## License

MIT — see [LICENSE](LICENSE).
