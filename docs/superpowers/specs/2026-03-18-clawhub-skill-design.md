# ClawHub Skill Design Spec

**Date:** 2026-03-18
**Status:** Approved
**Author:** Wyatt Reid + Claude

## Goal

Create a standalone, ClawHub-compatible skill repo for publishing the Shippo shipping skill to the ClawHub marketplace. This is a separate repo from the Claude Code plugin (`shippo-claude-plugin`).

## Repo Location

`/Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/`
Remote: `goshippo/shippo-clawhub-skill` on GitHub

## Structure

```
shippo-clawhub-skill/
├── SKILL.md                     # Unified skill with ClawHub frontmatter
├── references/
│   ├── tool-reference.md        # Full MCP tool catalog
│   ├── customs-guide.md         # International customs workflow
│   ├── carrier-guide.md         # Carrier-specific rules and limits
│   └── csv-format.md            # Batch CSV column spec
├── .clawhubignore               # Exclude non-skill files from publish
├── README.md                    # GitHub-facing docs (not published to ClawHub)
└── LICENSE                      # MIT (repo license; ClawHub forces MIT-0 on publish)
```

## SKILL.md Frontmatter

```yaml
---
name: shippo
description: >
  Ship packages with Shippo. Multi-carrier rate shopping, label generation,
  package tracking, address validation, customs declarations, and batch
  processing from CSV files.
version: 1.0.1
metadata:
  openclaw:
    requires:
      env:
        - SHIPPO_API_KEY
    primaryEnv: SHIPPO_API_KEY
    emoji: "📦"
    homepage: https://github.com/goshippo/shippo-clawhub-skill
---
```

## SKILL.md Body Sections

1. **Setup** (~15 lines) — MCP server config, prerequisites, API key setup
2. **Address Validation** (~50 lines) — validate, parse, bulk-validate workflows
3. **Rate Shopping** (~45 lines) — create shipment, compare rates, international rates
4. **Label Purchase** (~90 lines) — domestic, international + customs, return labels, void/refund
5. **Tracking** (~40 lines) — track by number, webhook setup, test mode
6. **Batch Shipping** (~60 lines) — CSV parsing, batch creation, purchase, manifests
7. **Shipping Analysis** (~40 lines) — cost comparison, dimension optimization, spend review
8. **Error Handling** (~15 lines) — global rules from former router skill
9. **Security & Data Transparency** (~10 lines) — data transmission disclosure

Estimated total: ~365 lines.

## Reference Files

4 files copied from `shippo-claude-plugin/skills/shippo/references/`:
- `tool-reference.md` (404 lines) — complete MCP tool catalog
- `customs-guide.md` (227 lines) — customs declaration workflow and fields
- `carrier-guide.md` (136 lines) — carrier-specific rules, limits, surcharges
- `csv-format.md` (106 lines) — batch CSV column specification

These are copied as-is. The remaining 6 reference files (address-formats, error-reference, international-shipping, label-formats, rate-shopping-guide, test-mode) have their key content inlined into the relevant SKILL.md sections.

## Key Design Decisions

1. **MCP-focused** — References specific Shippo MCP tool names. Includes setup instructions for configuring the MCP server.
2. **Single unified skill** — All 6 workflows in one SKILL.md rather than 6 separate ClawHub packages. Simpler to install, maintain, and keeps shared context.
3. **Separate repo** — Not a monorepo with the Claude Code plugin. Avoids coupling between distribution channels. Manual sync when needed.
4. **Inline dropped references** — Key content from the 6 excluded reference files is condensed into the SKILL.md body where relevant, rather than dropped entirely.
5. **Security transparency** — Includes a section disclosing that data is sent to Shippo's API via the MCP server hosted at `app.getgram.ai`.

## Publishing

- Initialize git repo, push to `goshippo/shippo-clawhub-skill`
- Publish to ClawHub via Import (paste GitHub URL)
- ClawHub slug: `shippo`
