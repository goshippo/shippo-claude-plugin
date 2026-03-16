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

> **Note:** Shippo provides managed carrier accounts for major carriers (USPS, UPS, FedEx, DHL) by default. You can get rates and buy labels immediately — no carrier setup needed.

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

## Distribution Roadmap

This plugin is being published across the major AI assistant ecosystems. The Shippo MCP server is already hosted and the Agent Skills files follow the open [agentskills.io](https://agentskills.io) standard, so the same core works across platforms with minimal per-channel packaging.

### Tier 1 — In Progress

| Platform | Status | What Ships | How Users Install |
|---|---|---|---|
| **Anthropic (Claude Code)** | Submitting | Plugin (skill + MCP) | `/plugin install shippo` from the Discover tab |
| **Official MCP Registry** | Next | MCP server metadata | Auto-surfaces in GitHub MCP Registry, VS Code, and registry-aware clients |

**Claude Code submission process:**
1. Validate plugin structure with `claude plugin validate .`
2. Submit via [claude.ai/settings/plugins/submit](https://claude.ai/settings/plugins/submit) or [platform.claude.com/plugins/submit](https://platform.claude.com/plugins/submit)
3. Anthropic reviews for quality and security
4. On approval, plugin appears in the [official marketplace](https://github.com/anthropics/claude-plugins-official) (`external_plugins/`)
5. Users install with `/plugin install shippo@claude-plugins-official`

**MCP Registry publishing process:**
1. Install `mcp-publisher` (`brew install mcp-publisher`)
2. Create `server.json` with namespace `io.github.goshippo/shippo-mcp`
3. Authenticate via `mcp-publisher login github`
4. Publish via `mcp-publisher publish`
5. Server appears in the [MCP Registry API](https://registry.modelcontextprotocol.io) — no approval step, instant

### Tier 2 — After Anthropic

| Platform | Effort | Audience | Submission |
|---|---|---|---|
| **ChatGPT App Directory** | Medium — needs OAuth 2.1, tool annotations, test credentials | 200M+ users | [platform.openai.com/apps-manage](https://platform.openai.com/apps-manage) |
| **Cursor Marketplace** | Low — same plugin format (skill + MCP) | Large (developers) | [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish) |
| **Cline MCP Marketplace** | Low — GitHub issue with logo + repo URL | Large (developers) | [github.com/cline/mcp-marketplace](https://github.com/cline/mcp-marketplace) |

### Tier 3 — Good to Have

| Platform | Notes |
|---|---|
| **Windsurf** | Curated MCP marketplace — no self-service, need to contact their team |
| **JetBrains / Junie** | Standard plugin submission at [plugins.jetbrains.com](https://plugins.jetbrains.com) |
| **Zed** | Requires Rust/WASM extension wrapper — more engineering effort |
| **mcp.so / Glama.ai / Smithery.ai** | Third-party MCP directories for discovery — low effort to list |

### Free Reach via Open Standards

The skill files in this repo follow the [Agent Skills](https://agentskills.io) standard, which is supported by 30+ platforms including Claude Code, Codex CLI, Gemini CLI, Cursor, VS Code/Copilot, and JetBrains Junie. No additional packaging needed — these platforms can consume `SKILL.md` files directly.

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
