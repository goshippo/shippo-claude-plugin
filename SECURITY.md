# Security Policy

This repository is the public Claude plugin mirror of
[goshippo/ai](https://github.com/goshippo/ai). It ships agent skill content
(Markdown), a plugin manifest, and an `.mcp.json` that points at Shippo's
hosted MCP server. It contains no secrets and no runtime code that handles
credentials.

## Reporting a vulnerability

Please report security issues **privately**. Do **not** open a public issue or
pull request for a security report.

- Email: security@goshippo.com
- Shippo API or platform issues: https://goshippo.com/security

We will acknowledge your report and follow up with next steps.

## In scope

- Prompt-injection or unsafe instructions in skill content (`skills/**`)
- A plugin manifest (`.claude-plugin/plugin.json`) or `.mcp.json` that leaks
  credentials or points at an unintended endpoint
- Anything that could cause the plugin to send a user's data or token somewhere
  other than Shippo's hosted MCP (`https://mcp.shippo.com`, per-user OAuth)

## Out of scope

- Vulnerabilities in Claude, the Claude plugin runtime, or third-party clients
- Issues in the canonical source repo (report those against goshippo/ai)
- The Shippo API itself (use https://goshippo.com/security)
