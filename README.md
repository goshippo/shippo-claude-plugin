# Shippo Plugin for Claude Code

A [Claude Code plugin](https://code.claude.com/docs/en/plugins) that adds shipping capabilities to Claude. It bundles a **skill** that teaches Claude shipping workflows and an **MCP server** that connects Claude to the [Shippo API](https://docs.goshippo.com) — so you can validate addresses, compare carrier rates, generate labels, track packages, handle customs, and process bulk shipments through natural conversation.

## How It Works

This plugin has two components:

- **Skill** (`skills/shippo/SKILL.md`) — Procedural knowledge that teaches Claude *how* to perform shipping tasks: the correct API call sequences, field requirements, error handling, and best practices for each workflow.
- **MCP server** (`.mcp.json`) — Connects Claude to Shippo's API tools so it can *execute* those workflows — creating shipments, purchasing labels, tracking packages, etc.

The skill and MCP server work together: the MCP server gives Claude access to Shippo's tools, and the skill teaches Claude how to use them effectively.

## What You Can Do

Just ask Claude things like:

- "Compare shipping rates for a 2lb package from San Francisco to New York"
- "Validate this address: 350 Fifth Ave, New York, NY 10118"
- "Ship this internationally to London — it's 2 books worth $30"
- "Track package SHIPPO_TRANSIT"
- "Process this CSV of shipments and generate labels for all of them"
- "What's the cheapest way to ship a 5lb box to Chicago?"

### Features

| Capability | Description |
|---|---|
| **Address Validation** | Validate, parse, and standardize US and international addresses |
| **Rate Shopping** | Compare rates across USPS, UPS, FedEx, DHL, and 30+ carriers |
| **Label Generation** | Purchase domestic and international shipping labels |
| **Package Tracking** | Track packages across all carriers with full status history |
| **Customs & International** | Automated customs declarations, HS code guidance, EEL/PFC handling |
| **Batch Processing** | Process CSV files of shipments and generate labels in bulk |

## Requirements

- [Claude Code](https://code.claude.com) (CLI)
- A [Shippo account](https://apps.goshippo.com/join) (free to sign up)
- A Shippo API key from your [Shippo dashboard](https://apps.goshippo.com/settings/api)

## Installation

### Before the plugin is published

While this plugin is pending approval in Anthropic's plugin directory, you can install it locally:

**1. Clone the repo**

```bash
git clone https://github.com/goshippo/shippo-claude-plugin.git
```

**2. Set your Shippo API key**

```bash
export SHIPPO_API_KEY="ShippoToken shippo_test_xxxxx"
```

Add this to your `~/.zshrc` or `~/.bashrc` to persist across sessions.

**3. Launch Claude Code with the plugin**

```bash
claude --plugin-dir ./shippo-claude-plugin
```

Or, to install the skill and MCP server separately without using the plugin wrapper:

- **Skill only**: Copy `skills/shippo/` into your project's `.claude/skills/` directory
- **MCP server only**: Run `claude mcp add --transport http shippo-mcp https://app.getgram.ai/mcp/shippo-mcp-beta -H "Mcp-Shippo-Merged-Api-Key-Header: ${SHIPPO_API_KEY}"`

### After the plugin is published

Once accepted into Anthropic's plugin directory, install with:

```bash
/plugin install goshippo/shippo-claude-plugin
```

**4. Verify it's working**

Ask Claude:

> "List my Shippo carrier accounts"

If configured correctly, Claude will return your connected carriers.

## Pricing

- **This plugin**: Free and open source (MIT)
- **Shippo account**: Free to sign up
- **Test mode**: Free — create shipments, get rates, validate addresses at no cost
- **Live labels**: Charged at [Shippo's discounted rates](https://goshippo.com/pricing) only when you purchase a label

## Repo Structure

```
shippo-claude-plugin/
├── .claude-plugin/
│   └── plugin.json        # Plugin manifest (name, version, metadata)
├── .mcp.json               # MCP server configuration (Shippo API connection)
├── skills/
│   └── shippo/
│       ├── SKILL.md        # Main skill — shipping workflow instructions
│       └── references/
│           ├── csv-format.md      # CSV column spec for batch processing
│           ├── customs-guide.md   # International customs declaration guide
│           └── tool-reference.md  # Complete MCP tool catalog
├── LICENSE
└── README.md
```

## Terminology

| Term | What it means |
|---|---|
| **Plugin** | The distribution package — this entire repo. Bundles skills, MCP configs, and metadata for installation via Claude Code. |
| **Skill** | A set of markdown instructions that teach Claude *how* to do something. Loaded on demand when relevant. Invokable with `/shippo`. |
| **MCP Server** | A running service that gives Claude *access* to external tools. In this plugin, it connects Claude to the Shippo API. |

For more on these concepts, see Anthropic's docs on [plugins](https://code.claude.com/docs/en/plugins), [skills](https://code.claude.com/docs/en/skills), and [MCP](https://code.claude.com/docs/en/mcp).

## Links

- [Shippo Documentation](https://docs.goshippo.com)
- [Shippo API Reference](https://docs.goshippo.com/docs/api_concepts/apiversioning/)
- [MCP Server (npm)](https://www.npmjs.com/package/@shippo/shippo-mcp)
- [Claude Code Docs — Plugins](https://code.claude.com/docs/en/plugins)
- [Agent Skills Standard](https://agentskills.io)

## License

MIT — see [LICENSE](LICENSE).
