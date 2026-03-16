# Shippo Plugin for Claude Code

A [Claude Code plugin](https://code.claude.com/docs/en/plugins) that gives Claude the ability to ship packages. Once installed, type **`/shippo`** and Claude can validate addresses, compare carrier rates, generate shipping labels, track packages, handle customs declarations, and process bulk shipments — all through natural conversation.

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

**5. Try it**

Type `/shippo` followed by what you want to do:

> `/shippo` compare rates for a 2lb package from San Francisco to New York

Claude will connect to Shippo, fetch live rates across carriers, and present your options. You can also just ask Claude shipping questions directly — the `/shippo` command ensures Claude loads its full shipping expertise.

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

- **A skill** (`/shippo`) — Teaches Claude the workflows, API sequences, and best practices for every shipping task. This is the procedural knowledge that makes Claude good at shipping.
- **An MCP server connection** — Connects Claude to Shippo's hosted API so it can actually execute those workflows (create shipments, buy labels, track packages, etc.). There's no local server to run — the connection is configured automatically when you load the plugin.

The skill teaches Claude *how* to ship. The MCP server gives Claude the *tools* to ship. Together, they make Claude a shipping expert.

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
├── .mcp.json               # MCP server connection (Shippo API — hosted, nothing to run)
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

| Term | What it means in this repo |
|---|---|
| **Plugin** | This entire repo — the installable package. Bundles the skill and MCP server config together so one `--plugin-dir` flag sets up everything. |
| **Skill** | The `/shippo` slash command. Markdown instructions that teach Claude how to perform shipping tasks. |
| **MCP Server** | The hosted service that gives Claude access to Shippo's API tools. Configured in `.mcp.json`, runs remotely — nothing to install locally. |

For more on these concepts, see Anthropic's docs on [plugins](https://code.claude.com/docs/en/plugins), [skills](https://code.claude.com/docs/en/skills), and [MCP servers](https://code.claude.com/docs/en/mcp).

## Publishing to Anthropic

This plugin is being submitted to [Anthropic's plugin directory](https://code.claude.com/docs/en/discover-plugins). Once approved, anyone will be able to install it with:

```
/plugin install goshippo/shippo-claude-plugin
```

Until then, use the `--plugin-dir` method in the Quick Start above.

## Links

- [Shippo Documentation](https://docs.goshippo.com)
- [Shippo API Reference](https://docs.goshippo.com/docs/api_concepts/apiversioning/)
- [MCP Server (npm)](https://www.npmjs.com/package/@shippo/shippo-mcp)
- [Claude Code Docs — Plugins](https://code.claude.com/docs/en/plugins)

## License

MIT — see [LICENSE](LICENSE).
