# ClawHub Skill Repo Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a standalone repo (`shippo-clawhub-skill`) containing a unified Shippo shipping skill formatted for ClawHub publication.

**Architecture:** Single SKILL.md merging all 6 workflow skills from the Claude Code plugin, plus 4 bundled reference files. ClawHub-specific frontmatter declares env requirements and metadata. Separate git repo at `goshippo/shippo-clawhub-skill`.

**Tech Stack:** Markdown, git, GitHub CLI (`gh`)

**Source repo:** `/Users/wyatt.reid/Desktop/official/shippo-claude-plugin`
**Target repo:** `/Users/wyatt.reid/Desktop/official/shippo-clawhub-skill`
**Spec:** `docs/superpowers/specs/2026-03-18-clawhub-skill-design.md`

---

### Task 1: Initialize the repo

**Files:**
- Create: `/Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/`
- Create: `/Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/.gitignore`
- Create: `/Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/.clawhubignore`
- Create: `/Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/LICENSE`

- [ ] **Step 1: Create directory and init git**

```bash
mkdir -p /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill
cd /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill
git init
```

- [ ] **Step 2: Create .gitignore**

```
.DS_Store
.env
.env.local
.env*.local
node_modules/
*.log
.vscode/
.idea/
```

- [ ] **Step 3: Create .clawhubignore**

Excludes non-skill files from ClawHub publish:

```
README.md
LICENSE
.gitignore
.git/
```

- [ ] **Step 4: Create LICENSE**

Copy MIT license from the Claude Code plugin repo:

```bash
cp /Users/wyatt.reid/Desktop/official/shippo-claude-plugin/LICENSE /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/LICENSE
```

- [ ] **Step 5: Commit**

```bash
git add .gitignore .clawhubignore LICENSE
git commit -m "Initialize repo with .gitignore, .clawhubignore, and LICENSE"
```

---

### Task 2: Copy and verify reference files

**Files:**
- Create: `references/tool-reference.md`
- Create: `references/customs-guide.md`
- Create: `references/carrier-guide.md`
- Create: `references/csv-format.md`

- [ ] **Step 1: Copy the 4 reference files**

```bash
mkdir -p /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/references
cp /Users/wyatt.reid/Desktop/official/shippo-claude-plugin/skills/shippo/references/tool-reference.md /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/references/
cp /Users/wyatt.reid/Desktop/official/shippo-claude-plugin/skills/shippo/references/customs-guide.md /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/references/
cp /Users/wyatt.reid/Desktop/official/shippo-claude-plugin/skills/shippo/references/carrier-guide.md /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/references/
cp /Users/wyatt.reid/Desktop/official/shippo-claude-plugin/skills/shippo/references/csv-format.md /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/references/
```

- [ ] **Step 2: Verify files copied correctly**

```bash
wc -l /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/references/*.md
```

Expected: tool-reference ~404, customs-guide ~227, carrier-guide ~136, csv-format ~106.

- [ ] **Step 3: Commit**

```bash
git add references/
git commit -m "Add reference files: tool catalog, customs, carrier guide, CSV format"
```

---

### Task 3: Write SKILL.md

**Files:**
- Create: `SKILL.md`

This is the main task. Write the unified SKILL.md with:

- [ ] **Step 1: Write SKILL.md frontmatter and Setup section**

Frontmatter with ClawHub-specific `metadata.openclaw` fields. Setup section covering MCP server configuration, API key, and prerequisites.

- [ ] **Step 2: Write Address Validation section**

Merge content from `skills/address-validation/SKILL.md`. Inline key bits from `address-formats.md` (the v1 field names table is already in the skill). ~50 lines.

- [ ] **Step 3: Write Rate Shopping section**

Merge content from `skills/rate-shopping/SKILL.md`. Inline rate-shopping-guide tips about dimensional weight and flat-rate where relevant. ~45 lines.

- [ ] **Step 4: Write Label Purchase section**

Merge content from `skills/label-purchase/SKILL.md`. Inline label-formats table (the supported formats list). Inline test-mode key identification. ~90 lines.

- [ ] **Step 5: Write Tracking section**

Merge content from `skills/tracking/SKILL.md`. Inline test-mode mock tracking numbers (SHIPPO_TRANSIT, SHIPPO_DELIVERED, etc.). ~40 lines.

- [ ] **Step 6: Write Batch Shipping section**

Merge content from `skills/batch-shipping/SKILL.md`. References `references/csv-format.md` and `references/customs-guide.md`. ~60 lines.

- [ ] **Step 7: Write Shipping Analysis section**

Merge content from `skills/shipping-analysis/SKILL.md`. ~40 lines.

- [ ] **Step 8: Write Error Handling section**

Global rules from the former router skill plus key patterns from error-reference.md (condensed). ~15 lines.

- [ ] **Step 9: Write Security & Data Transparency section**

Disclose: data is sent to Shippo's API via the MCP server hosted at `app.getgram.ai/mcp/shippo-mcp-beta`. Declare env var usage. ~10 lines.

- [ ] **Step 10: Review total line count**

Target: ~365 lines. If significantly over, trim verbose sections. If under, that's fine.

- [ ] **Step 11: Commit**

```bash
git add SKILL.md
git commit -m "Add unified Shippo skill with all 6 workflows"
```

---

### Task 4: Write README.md

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write README**

GitHub-facing README covering:
- What the skill does
- Installation via `npx clawhub@latest install shippo`
- MCP server setup (env var, server URL)
- Quick examples of what you can ask
- Link to the Claude Code plugin for the full plugin experience

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "Add README with installation and setup instructions"
```

---

### Task 5: Create GitHub repo and push

- [ ] **Step 1: Create the remote repo**

```bash
gh repo create goshippo/shippo-clawhub-skill --public --description "Shippo shipping skill for ClawHub — rate shopping, label generation, tracking, address validation, customs, and batch processing" --source . --push
```

- [ ] **Step 2: Verify on GitHub**

```bash
gh repo view goshippo/shippo-clawhub-skill
```

---

### Task 6: Publish to ClawHub

- [ ] **Step 1: Import via ClawHub web UI**

Navigate to https://clawhub.ai/import, paste `https://github.com/goshippo/shippo-clawhub-skill`, select the SKILL.md, set slug to `shippo`, version `1.0.1`, tag `latest`.

- [ ] **Step 2: Verify listing**

Check https://clawhub.ai/ and search for "shippo" to confirm the skill appears.
