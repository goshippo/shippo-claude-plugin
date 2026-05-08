# Consolidate `shippo-clawhub-skill` into `shippo-claude-plugin` Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Eliminate cross-repo content drift by collapsing `shippo-clawhub-skill` into `shippo-claude-plugin` as a sibling distribution target, with a ClawHub redirect preserving the existing `shippo-official` install slug.

**Architecture:** One repo, two channels. `skills/<name>/SKILL.md` keeps powering the Claude Code plugin (six split skills + shared references). A new `clawhub/shippo-official/` subdirectory holds the consolidated single-skill ClawHub bundle, hand-curated to match. A future ClawHub publish points at the new repo path; the redirect remaps the existing `shippo-official` slug to the new source. No build scripts — every cross-channel change is a single hand-edited PR touching both views.

**Tech Stack:** Markdown (SKILL.md format per agentskills.io standard), ClawHub publish CLI, GitHub.

**Out of scope:** OAuth transport rewrite (lands ~2026-05-15 alongside Gram OAuth cutover, separate work). Plugin submission to `anthropics/claude-plugins-official` marketplace.

---

## Repo state at plan-write time

- `shippo-claude-plugin` is on branch `wyatt/clawhub-alignment` (clean working tree, no commits beyond `main`). All work in this plan lands on this branch.
- `shippo-clawhub-skill` is on branch `wyatt/sync-with-plugin` with three commits ahead of `main` and clean working tree:
  - `c5b079f` Fix typos in references
  - `307e3c1` Exclude .env files from ClawHub publish
  - `43b0874` Migrate from hosted MCP to local @shippo/shippo-mcp npm package
  These commits are the canonical source for the consolidated ClawHub bundle. Plan reads from this branch (NOT from `main`) when copying content.
- The `shippo-official` skill is currently published live on ClawHub. **No commits in this plan should be merged to `main` of either repo until Phase 4** — that's where the live cutover happens.

---

## File structure (target end-state in `shippo-claude-plugin`)

```
shippo-claude-plugin/
├── .claude-plugin/
│   └── plugin.json                          # existing — Claude Code manifest
├── .clawhub/                                # NEW — ClawHub publish manifest
│   └── plugin.json                          # NEW — ClawHub bundle config (single skill)
├── .clawhubignore                           # NEW — ClawHub publish exclusions
├── .mcp.json                                # existing — Claude Code MCP wiring (untouched)
├── clawhub/
│   └── shippo-official/                     # NEW — the consolidated ClawHub skill
│       ├── SKILL.md                         # NEW — copied from clawhub repo (post-MCP-migration version)
│       └── references/
│           ├── carrier-guide.md             # NEW — copied
│           ├── csv-format.md                # NEW — copied
│           ├── customs-guide.md             # NEW — copied (typo-fixed version)
│           └── tool-reference.md            # NEW — copied (typo-fixed version)
├── skills/                                  # existing — Claude Code skills (untouched layout)
│   ├── address-validation/SKILL.md          # existing — may be edited in Phase 2
│   ├── batch-shipping/SKILL.md              # existing — may be edited in Phase 2
│   ├── label-purchase/SKILL.md              # existing — may be edited in Phase 2
│   ├── rate-shopping/SKILL.md               # existing — may be edited in Phase 2
│   ├── shipping-analysis/SKILL.md           # existing — may be edited in Phase 2
│   ├── tracking/SKILL.md                    # existing — may be edited in Phase 2
│   └── shippo/references/                   # existing 10 reference files
│       ├── address-formats.md               # existing
│       ├── carrier-guide.md                 # existing
│       ├── csv-format.md                    # existing
│       ├── customs-guide.md                 # existing (already has typo-fixed content per audit)
│       ├── error-reference.md               # existing — extended in Phase 2 with non-envelope errors
│       ├── international-shipping.md        # existing
│       ├── label-formats.md                 # existing
│       ├── rate-shopping-guide.md           # existing
│       ├── response-envelope.md             # NEW — Speakeasy wrapper structure (Phase 2)
│       ├── test-mode.md                     # existing
│       └── tool-reference.md                # existing (already has typo-fixed content per audit)
├── docs/superpowers/plans/
│   └── 2026-05-08-consolidate-clawhub-into-plugin.md   # this file
├── README.md                                # existing — minor updates in Phase 3 only
└── (no other layout changes)
```

**Why two parallel ClawHub copies of `customs-guide.md` and `tool-reference.md` exist** (one under `clawhub/shippo-official/references/`, one under `skills/shippo/references/`): the ClawHub bundle is published as a self-contained skill folder per the ClawHub spec — it can't reference files outside its own directory. The shared `skills/shippo/references/` is a Claude Code plugin convention, not a ClawHub-portable layout. Hand-curated parity is enforced by Phase 2.

---

## Phase 0: Pre-flight verification

These tasks resolve unknowns. **Do not begin Phase 1 until all Phase 0 tasks return clear answers.** No code changes in this phase — research and decision only.

### Phase 0 findings (recorded 2026-05-08, ClawHub CLI v0.12.3)

**0.1 — Publish mechanics.** `clawhub skill publish <path>` (or legacy alias `clawhub publish <path>`) takes a skill folder path and publishes it. Slug, name, version come from the SKILL.md frontmatter; `--slug`, `--name`, `--version`, `--changelog`, `--tags`, `--owner`, `--fork-of` are available as overrides. **No `--dry-run` on `publish`** — but `clawhub sync --dry-run --root <dir>` scans a tree and reports what would publish, which serves the dry-run need in Phase 3. The `.clawhubignore` file lives **at the skill folder root** (per `openclaw/clawhub/docs/skill-format.md`), not at the repo root — so for our setup it goes in `clawhub/shippo-official/.clawhubignore`. The repo-root `.clawhubignore` and `.clawhub/plugin.json` originally proposed in this plan are **not required** for single-skill publishing; **dropping them from Phase 1**.

**0.2 — Redirect mechanism.** Built into the CLI as `clawhub skill rename <old> <new>` (keeps old slug as a redirect) and `clawhub skill merge <source> <target>` (redirects source to target). For our case the slug stays `shippo-official` and only the source repo changes, so **no rename/merge is needed** — publishing under the same slug from the new path is just a new version. ClawHub treats it as an update, not a new skill.

**0.3 — Bundle shape.** Confirmed single-skill (the plan default). The current live skill is `shippo-official`; publishing from `clawhub/shippo-official/` keeps the same slug, no breaking change. Multi-skill expansion remains a future option (use `clawhub sync` to scan multiple folders) but is out of scope.

**Plan adjustments based on findings:**
- File-structure section: remove `.clawhub/` dir and `.clawhub/plugin.json` from the target layout. Move `.clawhubignore` from repo root into `clawhub/shippo-official/.clawhubignore`.
- Phase 1.4: replace "Copy `.clawhubignore` and seed `.clawhub/plugin.json`" with "Copy `.clawhubignore` into the skill folder."
- Phase 3.1 (finalize manifest) becomes a no-op (no separate manifest file). The `metadata.openclaw` block in `SKILL.md` is already complete from the original clawhub repo.
- Phase 3.2 dry-run uses `clawhub sync --dry-run --root clawhub` instead of a `publish --dry-run` flag.

### Task 0.1: Confirm ClawHub publish mechanics from a sub-directory

**Files:** none (research only)

- [ ] **Step 1: Read the ClawHub publish CLI reference.**

Run:
```bash
npx clawhub@latest skill publish --help
npx clawhub@latest plugin publish --help 2>/dev/null || true
```
Expected output: command help text. Read both `skill publish` and `plugin publish` (if it exists) sections.

- [ ] **Step 2: Confirm publishing from a path argument is supported.**

Look in the help output for a `<path>` positional or `--path` flag. The Salesforce-plugin precedent at `github.com/dsouzaAnush/salesforce-plugin` runs `clawhub skill publish <skill-folder>` per skill. Verify this still matches the current CLI surface.

- [ ] **Step 3: Determine whether ClawHub treats `clawhub/shippo-official/` as a publishable skill.**

Run from a clean checkout of the target plan state (after Phase 1):
```bash
cd /Users/wyatt.reid/Desktop/official/shippo-claude-plugin
npx clawhub@latest skill validate clawhub/shippo-official
```
Expected: validation passes (exit 0) — the SKILL.md frontmatter and references resolve. If `skill validate` doesn't exist, use whatever lint/dry-run command the CLI offers.

- [ ] **Step 4: Document findings inline below.**

Add a short subsection to this plan file (`### Task 0.1 findings`) with:
- The exact `publish` command shape that targets a sub-directory
- Whether the CLI has a `--dry-run` or sandbox/staging publish mode
- Any constraints (e.g., must `.clawhubignore` and `.clawhub/` live at repo root, or can they live alongside the skill folder?)

**Decision gate:** if ClawHub does not support sub-directory publishing, escalate to user — the plan needs restructuring (e.g., `clawhub/shippo-official` becomes a separate workspace inside a monorepo, or we keep two repos and accept partial alignment).

### Task 0.2: Confirm ClawHub redirect mechanism

**Files:** none (research only)

- [ ] **Step 1: Search ClawHub docs and CLI for a redirect / move command.**

Run:
```bash
npx clawhub@latest skill --help | grep -iE "redirect|move|rename|transfer"
```
Then check the openclaw/clawhub repo: `https://github.com/openclaw/clawhub/tree/main/docs`. Look for `redirect`, `move-skill`, or slug-transfer documentation.

- [ ] **Step 2: If no first-class redirect command exists, identify the practical fallbacks.**

Likely options to evaluate:
- (a) Re-publishing the same `name: shippo-official` slug from a new repo source — does ClawHub block this on origin-mismatch, or accept it?
- (b) Filing an issue/PR against `openclaw/clawhub` requesting a slug transfer
- (c) Publishing under a new slug (e.g., `shippo`) and letting `shippo-official` decay; users must migrate

- [ ] **Step 3: Document findings inline below.**

Add `### Task 0.2 findings` subsection with:
- The redirect path (CLI command, manual ops, or PR-against-registry)
- Time/coordination required (instant vs. requires ClawHub maintainer review)
- Whether the existing `app.getgram.ai/...` MCP-server URL referenced in the live skill changes (it shouldn't — same skill content, just different source repo)

**Decision gate:** if redirect requires multi-day coordination, push that conversation upstream now (Phase 0 of the plan — not blocking, runs in parallel with Phase 1).

### Task 0.3: Decide single-skill vs multi-skill ClawHub bundle shape

**Files:** none (decision only)

- [ ] **Step 1: Re-confirm the goal.**

The current ClawHub install installs ONE skill (`shippo-official`). Plan baseline assumes we keep that — single-skill bundle, just sourced from a new repo. Multi-skill (publish 6 ClawHub skills under one plugin) is the alternative.

- [ ] **Step 2: Evaluate the multi-skill alternative.**

Pros: 1:1 with the Claude Code plugin's six skills, no monolithic SKILL.md to hand-curate.
Cons: Breaking change to the existing `shippo-official` slug (users get fragmented installs); requires the redirect to handle a 1→6 expansion; `npx clawhub install shippo-official` becomes ambiguous.

- [ ] **Step 3: Document the decision inline.**

Add `### Task 0.3 decision` subsection. Default recommendation: **stay single-skill**. Multi-skill expansion can be a future, separate plan once the consolidation is stable.

**Decision gate:** if multi-skill is chosen, restart Phase 1 with a different file layout. Plan as written assumes single-skill.

---

## Phase 1: Lay out target structure in plugin repo

No publishing impact. All work on `wyatt/clawhub-alignment` branch. No `main` merges in this phase.

### Task 1.1: Create the ClawHub bundle directory and `.clawhub/` manifest dir

**Files:**
- Create: `clawhub/shippo-official/` (directory only — placeholder)
- Create: `clawhub/shippo-official/references/` (directory only — placeholder)
- Create: `.clawhub/` (directory only — placeholder)

- [ ] **Step 1: Create the directories.**

Run:
```bash
cd /Users/wyatt.reid/Desktop/official/shippo-claude-plugin
mkdir -p clawhub/shippo-official/references
mkdir -p .clawhub
```
Expected: directories exist. `ls clawhub/shippo-official/references` → empty.

- [ ] **Step 2: Verify branch and clean tree.**

Run:
```bash
git status
git branch --show-current
```
Expected: branch is `wyatt/clawhub-alignment`. Working tree shows the new (empty) directories as untracked.

- [ ] **Step 3: Skip commit — empty directories don't track.**

Move on to 1.2; the commit happens after the files land.

### Task 1.2: Copy SKILL.md from `shippo-clawhub-skill` (post-MCP-migration version)

**Files:**
- Create: `clawhub/shippo-official/SKILL.md` (copy of `shippo-clawhub-skill/SKILL.md` from branch `wyatt/sync-with-plugin`)

- [ ] **Step 1: Copy the SKILL.md from the clawhub repo's branch.**

Run:
```bash
cp /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/SKILL.md \
   /Users/wyatt.reid/Desktop/official/shippo-claude-plugin/clawhub/shippo-official/SKILL.md
```
Source must be from the `wyatt/sync-with-plugin` branch checkout (which is the current working tree of `shippo-clawhub-skill`). If unsure, run `cd /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill && git branch --show-current` and confirm `wyatt/sync-with-plugin`.

- [ ] **Step 2: Verify the copy succeeded and contains the post-migration content.**

Run:
```bash
cd /Users/wyatt.reid/Desktop/official/shippo-claude-plugin
head -3 clawhub/shippo-official/SKILL.md
grep -c "@shippo/shippo-mcp" clawhub/shippo-official/SKILL.md
grep -c "Response envelope" clawhub/shippo-official/SKILL.md
```
Expected:
- First three lines start with `---\nname: shippo-official\n`
- `@shippo/shippo-mcp` count ≥ 1 (confirms post-migration version)
- `Response envelope` count ≥ 1 (confirms response-envelope section is present)

If any check fails, re-verify the source branch and re-copy.

- [ ] **Step 3: Skip commit — files batched in Task 1.5.**

### Task 1.3: Copy reference files from `shippo-clawhub-skill`

**Files:**
- Create: `clawhub/shippo-official/references/carrier-guide.md`
- Create: `clawhub/shippo-official/references/csv-format.md`
- Create: `clawhub/shippo-official/references/customs-guide.md`
- Create: `clawhub/shippo-official/references/tool-reference.md`

- [ ] **Step 1: Copy all four references.**

Run:
```bash
cp /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/references/carrier-guide.md \
   /Users/wyatt.reid/Desktop/official/shippo-claude-plugin/clawhub/shippo-official/references/

cp /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/references/csv-format.md \
   /Users/wyatt.reid/Desktop/official/shippo-claude-plugin/clawhub/shippo-official/references/

cp /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/references/customs-guide.md \
   /Users/wyatt.reid/Desktop/official/shippo-claude-plugin/clawhub/shippo-official/references/

cp /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/references/tool-reference.md \
   /Users/wyatt.reid/Desktop/official/shippo-claude-plugin/clawhub/shippo-official/references/
```

- [ ] **Step 2: Verify all four files landed.**

Run:
```bash
cd /Users/wyatt.reid/Desktop/official/shippo-claude-plugin
ls clawhub/shippo-official/references/
wc -l clawhub/shippo-official/references/*.md
```
Expected: 4 files. `customs-guide.md` ~228 lines, `tool-reference.md` ~405 lines, `carrier-guide.md` and `csv-format.md` non-zero.

- [ ] **Step 3: Verify the typo-fix content is present (i.e., we copied from the right branch).**

Run:
```bash
grep -c "rates-checkout-update-parcel-template" clawhub/shippo-official/references/tool-reference.md
grep -c "\` eccn_ear99\`" clawhub/shippo-official/references/customs-guide.md
```
Expected:
- First grep returns ≥ 1 (the corrected name)
- Second grep returns 0 (the leading-space variant should be GONE in the fixed version)

If either check fails, the source was the unpatched `main` instead of `wyatt/sync-with-plugin`. Re-copy.

- [ ] **Step 4: Skip commit — batched in 1.5.**

### Task 1.4: Copy `.clawhubignore` and seed the `.clawhub/plugin.json` manifest

**Files:**
- Create: `.clawhubignore` (copy from clawhub repo branch)
- Create: `.clawhub/plugin.json` (placeholder until Phase 0 findings finalize the format)

- [ ] **Step 1: Copy `.clawhubignore` from the clawhub repo branch.**

Run:
```bash
cp /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill/.clawhubignore \
   /Users/wyatt.reid/Desktop/official/shippo-claude-plugin/.clawhubignore
```

- [ ] **Step 2: Verify `.clawhubignore` contents.**

Run:
```bash
cd /Users/wyatt.reid/Desktop/official/shippo-claude-plugin
cat .clawhubignore
```
Expected output: includes `README.md`, `LICENSE`, `.gitignore`, `.git/`, `.env`, `.env.local`, `.env*.local`. If not, re-copy.

- [ ] **Step 3: Adjust `.clawhubignore` for the plugin-repo context.**

The plugin repo has additional top-level files that should NOT publish to ClawHub: `.claude-plugin/`, `.mcp.json`, `skills/` (those are the Claude Code skills, separate from ClawHub bundle), `examples/`, `docs/`, `CHANGELOG.md`, `CONTRIBUTING.md`, `jira-ticket-oauth-mcp.md`. Append these to `.clawhubignore`:

Open `.clawhubignore` in an editor and append:
```
.claude-plugin/
.mcp.json
skills/
examples/
docs/
CHANGELOG.md
CONTRIBUTING.md
jira-ticket-oauth-mcp.md
```

Verify after edit:
```bash
cat .clawhubignore
```
Expected: contains all original lines PLUS the new appended block.

- [ ] **Step 4: Create `.clawhub/plugin.json`.**

Format depends on Phase 0.1 findings. Placeholder content (refine after Phase 0):
```json
{
  "name": "shippo-official",
  "version": "1.0.3",
  "skills": [
    "clawhub/shippo-official"
  ]
}
```
Write this to `.clawhub/plugin.json`. **Update the schema after Phase 0.1 returns the canonical ClawHub plugin manifest format.**

- [ ] **Step 5: Skip commit — batched in 1.5.**

### Task 1.5: Validate the layout, then commit Phase 1

**Files:** all of the above

- [ ] **Step 1: Validate with the ClawHub CLI (per Phase 0.1 findings).**

Run (exact command per Phase 0.1):
```bash
cd /Users/wyatt.reid/Desktop/official/shippo-claude-plugin
npx clawhub@latest skill validate clawhub/shippo-official
```
Expected: exit 0, no errors. Resolve any errors before committing — common fixes: missing reference files, malformed frontmatter, `.clawhubignore` excluding something the SKILL.md depends on.

- [ ] **Step 2: Inspect the staged file set.**

Run:
```bash
git status
git diff --stat
```
Expected: ~7 new files (`SKILL.md`, 4 references, `.clawhubignore`, `.clawhub/plugin.json`). No modifications to existing tracked files.

- [ ] **Step 3: Stage and commit.**

```bash
git add clawhub/ .clawhub/ .clawhubignore
git commit -m "$(cat <<'EOF'
Add ClawHub bundle as sibling distribution target

Move shippo-official skill content into clawhub/shippo-official/ inside
this plugin repo, alongside the existing Claude Code skills/ tree. Source
files copied from shippo-clawhub-skill@wyatt/sync-with-plugin (typo-fixes
+ MCP transport migration commits). Adds .clawhub/ manifest dir and
.clawhubignore configured for the plugin-repo context.

Standalone shippo-clawhub-skill repo will be archived in Phase 4 once
ClawHub publish source is migrated.
EOF
)"
```

- [ ] **Step 4: Verify the commit.**

Run:
```bash
git log --oneline -3
git status
```
Expected: new commit at HEAD, working tree clean.

---

## Phase 2: Reconcile content drift between plugin and clawhub views

Surgical hand-curation. Each task picks a content area, compares the consolidated clawhub `SKILL.md` (now at `clawhub/shippo-official/SKILL.md`) to the corresponding plugin `skills/<name>/SKILL.md` + relevant references, and brings them into parity. Each task is its own commit on `wyatt/clawhub-alignment`.

The audit identified two definite deltas the plugin is missing:
- **Response envelope** structure (Speakeasy wrapper)
- **Non-envelope errors** (MCP-protocol-level errors with `isError: true`, JSON-RPC `-32602`)

Both are transport-agnostic and apply to both channels. Task 2.1 lands them. Tasks 2.2–2.7 audit each workflow and reconcile any further drift.

### Phase 2 audit results (2026-05-08, four parallel Explore agents)

| Workflow | Verdict | Action taken |
|---|---|---|
| 2.1 Response envelope + non-envelope errors | drift confirmed | Added `skills/shippo/references/response-envelope.md`; appended "Non-envelope MCP-protocol errors" section to `error-reference.md`. Commit `0777bc0`. |
| 2.2 Address Validation | match (plugin is superset — adds Re-validate, Duplicate Addresses, Quick Reference subsections) | No edit. |
| 2.3 Rate Shopping | match (plugin's `rate-shopping-guide.md` is more detailed on Dimensional Weight worked example and Flat Rate decision tree than clawhub's brief table; this is appropriate structural difference, not drift) | No edit. |
| 2.4 Label Purchase | match (plugin's `label-formats.md` extends clawhub's brief table with batch `label_filetype` field-name note + S3 URL truncation warning, both already in workflow skills) | No edit. |
| 2.5 Tracking | match | No edit. |
| 2.6 Batch Shipping | match | No edit. |
| 2.7 Shipping Analysis | match | No edit. |
| 2.8 Error Handling | "drift" reported (parcel-string rule + S3-URL rule missing from `error-reference.md` General Rules) — but verified both already exist in workflow skills (`rate-shopping/SKILL.md:10`, `batch-shipping/SKILL.md`, `label-purchase/SKILL.md`). Centralizing would duplicate. | No edit — plugin's per-workflow rule placement is preferred over error-reference centralization. |
| 2.8 Security & Data Transparency | intentionally divergent — different transports, different stories | Skipped per plan. |

**Net Phase 2 actions:** one commit (`0777bc0` for Task 2.1). Tasks 2.2–2.8 had no drift requiring plugin edits — the plugin is a content superset with appropriate structural differences for its 6-skill + 10-reference layout vs. the consolidated single-SKILL.md ClawHub view.

### Task 2.1: Add response-envelope reference and non-envelope-error guidance to plugin

**Files:**
- Create: `skills/shippo/references/response-envelope.md`
- Modify: `skills/shippo/references/error-reference.md` (append section)

- [ ] **Step 1: Extract the two paragraphs from the consolidated clawhub SKILL.md.**

Run:
```bash
cd /Users/wyatt.reid/Desktop/official/shippo-claude-plugin
grep -A 1 "Response envelope:" clawhub/shippo-official/SKILL.md
grep -A 1 "Non-envelope errors:" clawhub/shippo-official/SKILL.md
```
You should see both paragraphs. Copy their full content (including the surrounding context — examples, payload field naming, error code mapping).

- [ ] **Step 2: Create `skills/shippo/references/response-envelope.md`.**

Write content with this structure:
```markdown
# Response Envelope

Reference for the Speakeasy wrapper structure that the Shippo MCP places around API responses.

## Envelope shape

Most successful and 4xx responses come wrapped in:

\`\`\`json
{
  "ContentType": "application/json",
  "StatusCode": <code>,
  "RawResponse": {},
  "<PayloadName>": { ...actual response... }
}
\`\`\`

The payload field is named after the response schema on success (e.g.
`ParsedAddress`, `AddressPaginatedList`, `AddressValidationResultV2`,
`AddressWithMetadataResponse`, `Shipment`, `CarrierAccountPaginatedList`).
On some errors the payload is named after the HTTP status code instead
(e.g. `fourHundredAndNineApplicationJsonObject` for a 409 — body may be
`{}`).

## Extracting the payload

Find the field whose key is NOT one of `ContentType`, `StatusCode`,
`RawResponse`. That is the payload. Branch on `StatusCode` for success
vs. error.

## When the envelope is bypassed

See [error-reference.md](error-reference.md#non-envelope-mcp-protocol-errors)
for failures that surface as MCP-level errors instead of envelope-wrapped
responses.
```

(Backtick escaping: the `\`\`\`json` lines should be literal triple backticks in the saved file. Use a plain `Write` to the path; do not let your editor normalize them.)

- [ ] **Step 3: Append a "Non-envelope MCP-protocol errors" section to `skills/shippo/references/error-reference.md`.**

Open `skills/shippo/references/error-reference.md`. Append (after the existing "General Debugging Steps" section):

```markdown
---

## Non-envelope MCP-protocol errors

Some failures bypass the Speakeasy [response envelope](response-envelope.md)
entirely and surface as MCP-protocol-level errors instead. Two flavors:

### Tool-result errors (`isError: true`)

The MCP tool response has `isError: true` with a single text block
containing a plaintext message:

\`\`\`
Unexpected API response status or content-type:
Status 404 Content-Type application/json Body: {"detail":"Not found."}
\`\`\`

These typically indicate the upstream Shippo API returned an unexpected
shape (e.g. a 404 on a tracking lookup the SDK didn't anticipate).
Report the plaintext body to the user verbatim — it carries the actual
error detail.

### Argument-validation errors (JSON-RPC `-32602`)

Pre-call validation failures (missing required field, type mismatch)
return JSON-RPC error code `-32602` ("Invalid params") with a `message`
field describing the failure. These are MCP-client-side; correct the
arguments and retry.

### Handling both paths

When reporting errors:
1. Check for `isError: true` on the tool result first.
2. If absent, check `StatusCode` inside the envelope.
3. If JSON-RPC `-32602` is returned, the call never reached Shippo —
   it's an arg-shape problem, not an API problem.
```

- [ ] **Step 4: Verify the cross-link works.**

Run:
```bash
grep "response-envelope.md" skills/shippo/references/error-reference.md
grep "error-reference.md" skills/shippo/references/response-envelope.md
```
Expected: both grep commands find the link in the other direction. Cross-references intact.

- [ ] **Step 5: Commit.**

```bash
git add skills/shippo/references/response-envelope.md skills/shippo/references/error-reference.md
git commit -m "$(cat <<'EOF'
Add response-envelope reference and non-envelope-error guidance

The Speakeasy wrapper structure and MCP-protocol-level error paths apply
regardless of MCP transport (hosted Gram URL or local npm). Mirrors
content already present in the consolidated ClawHub SKILL.md so both
channels surface the same response-handling guidance.

EOF
)"
```

### Task 2.2: Reconcile Address Validation workflow

**Files:**
- Read: `clawhub/shippo-official/SKILL.md` (Address Validation section)
- Read: `skills/address-validation/SKILL.md`
- Read: `skills/shippo/references/address-formats.md`
- Modify (if drift found): one or more of the above

- [ ] **Step 1: Extract the Address Validation section from the consolidated clawhub SKILL.md.**

Run:
```bash
cd /Users/wyatt.reid/Desktop/official/shippo-claude-plugin
sed -n '/^## Address Validation/,/^## Rate Shopping/p' clawhub/shippo-official/SKILL.md > /tmp/clawhub-address.md
wc -l /tmp/clawhub-address.md
```
Expected: a non-empty file containing the section (until the next H2).

- [ ] **Step 2: Read the plugin's address-validation skill and references in parallel.**

Read:
- `skills/address-validation/SKILL.md`
- `skills/shippo/references/address-formats.md`

- [ ] **Step 3: List drift line-by-line.**

For each subsection in the clawhub Address Validation block (Field Format, Validate a Structured Address, Parse a Freeform Address, International Addresses, Bulk Address Validation), check whether the equivalent content exists in the plugin (either in `skills/address-validation/SKILL.md` or in `skills/shippo/references/address-formats.md`). Note any:
- Missing subsections
- Different step numbering or sequence
- Different example values
- Different terminology

Write findings to a scratch buffer (do NOT commit).

- [ ] **Step 4: Decide which side is correct, per drift item.**

For each drift item, the plugin version usually wins (it's already split into focused skill+reference and tends to be more current). Exception: anything specific to the post-2026-03-18 work (response envelope, MCP transport) — clawhub wins there because it has the newer content. Note your choice per drift item.

- [ ] **Step 5: Apply the fixes.**

Edit the plugin file(s) to absorb the chosen content. Keep the plugin's split structure (don't merge address-formats reference back into the SKILL.md). If the clawhub version wins, copy that prose into the plugin SKILL.md or address-formats.md, NOT the other way around.

- [ ] **Step 6: Verify by re-running the diff.**

Run:
```bash
sed -n '/^## Address Validation/,/^## Rate Shopping/p' clawhub/shippo-official/SKILL.md > /tmp/clawhub-address-new.md
diff /tmp/clawhub-address.md /tmp/clawhub-address-new.md
```
Expected: no diff (we didn't edit clawhub side). The plugin side now has parity with the clawhub content.

- [ ] **Step 7: Commit (only if drift was found and fixed).**

```bash
git add skills/address-validation/SKILL.md skills/shippo/references/address-formats.md
git commit -m "Reconcile Address Validation content with ClawHub bundle"
```

If no drift was found, skip the commit and add a note to this task: "No drift found — content already in parity."

### Task 2.3: Reconcile Rate Shopping workflow

**Files:**
- Read: `clawhub/shippo-official/SKILL.md` (Rate Shopping section)
- Read: `skills/rate-shopping/SKILL.md`
- Read: `skills/shippo/references/rate-shopping-guide.md`
- Modify (if drift found): one or more of the above

Repeat the Task 2.2 procedure, with these section markers:

- [ ] **Step 1: Extract.** `sed -n '/^## Rate Shopping/,/^## Label Purchase/p' clawhub/shippo-official/SKILL.md > /tmp/clawhub-rates.md`
- [ ] **Step 2: Read the plugin counterparts.** `skills/rate-shopping/SKILL.md` + `skills/shippo/references/rate-shopping-guide.md`
- [ ] **Step 3: List drift.** Subsections in clawhub: Get Rates, Dimensional Weight, Flat Rate, Filter by Speed, International Rates, Checkout Rates (Line Items), Recommendation, Troubleshooting: No Rates.
- [ ] **Step 4: Decide per drift item.** Plugin usually wins.
- [ ] **Step 5: Apply.** Edit the plugin side.
- [ ] **Step 6: Re-diff to confirm parity.**
- [ ] **Step 7: Commit.** `git commit -m "Reconcile Rate Shopping content with ClawHub bundle"` (or skip if no drift).

### Task 2.4: Reconcile Label Purchase workflow

**Files:**
- Read: `clawhub/shippo-official/SKILL.md` (Label Purchase section)
- Read: `skills/label-purchase/SKILL.md`
- Read: `skills/shippo/references/customs-guide.md`, `label-formats.md`, `international-shipping.md`

- [ ] **Step 1: Extract.** `sed -n '/^## Label Purchase/,/^## Tracking/p' clawhub/shippo-official/SKILL.md > /tmp/clawhub-labels.md`
- [ ] **Step 2: Read counterparts.**
- [ ] **Step 3: List drift.** Subsections: Purchase Confirmation Gate, Domestic Label, International Label, Contents Type Decision Tree, Incoterms Decision Logic, Label Format Options, Label Customization Options, Return Labels, Voiding a Label, Orders and Packing Slips.
- [ ] **Step 4–7: Reconcile, verify, commit.** `git commit -m "Reconcile Label Purchase content with ClawHub bundle"`

### Task 2.5: Reconcile Tracking workflow

**Files:**
- Read: `clawhub/shippo-official/SKILL.md` (Tracking section)
- Read: `skills/tracking/SKILL.md`
- Read: `skills/shippo/references/test-mode.md`

- [ ] **Step 1: Extract.** `sed -n '/^## Tracking/,/^## Batch Shipping/p' clawhub/shippo-official/SKILL.md > /tmp/clawhub-tracking.md`
- [ ] **Step 2–7: Reconcile, verify, commit.** Subsections: Track by Number, Status Values, Test Mode Tracking, Find Trackable Packages, Register a Tracking Webhook. Commit message: `Reconcile Tracking content with ClawHub bundle`.

### Task 2.6: Reconcile Batch Shipping workflow

**Files:**
- Read: `clawhub/shippo-official/SKILL.md` (Batch Shipping section)
- Read: `skills/batch-shipping/SKILL.md`
- Read: `skills/shippo/references/csv-format.md`

- [ ] **Step 1: Extract.** `sed -n '/^## Batch Shipping/,/^## Shipping Analysis/p' clawhub/shippo-official/SKILL.md > /tmp/clawhub-batch.md`
- [ ] **Step 2–7: Reconcile, verify, commit.** Subsections: Purchase Confirmation Gate, CSV Batch Processing, Polling and Batch Size, Batch with Rate Shopping, Managing an Existing Batch, End-of-Day Manifest. Commit message: `Reconcile Batch Shipping content with ClawHub bundle`.

### Task 2.7: Reconcile Shipping Analysis workflow

**Files:**
- Read: `clawhub/shippo-official/SKILL.md` (Shipping Analysis section)
- Read: `skills/shipping-analysis/SKILL.md`

- [ ] **Step 1: Extract.** `sed -n '/^## Shipping Analysis/,/^## Error Handling/p' clawhub/shippo-official/SKILL.md > /tmp/clawhub-analysis.md`
- [ ] **Step 2–7: Reconcile, verify, commit.** Subsections: Geographic Cost Analysis, Package Optimization, Carrier Comparison, Historical Cost Optimization, Output Conventions. Commit message: `Reconcile Shipping Analysis content with ClawHub bundle`.

### Task 2.8: Reconcile Error Handling and Security sections

**Files:**
- Read: `clawhub/shippo-official/SKILL.md` (Error Handling, Security & Data Transparency sections)
- Read: `skills/shippo/references/error-reference.md`

- [ ] **Step 1: Extract.** `sed -n '/^## Error Handling/,/^---$/p' clawhub/shippo-official/SKILL.md > /tmp/clawhub-errors.md`

- [ ] **Step 2: Compare to error-reference.md.**

The clawhub Error Handling section is bullet-list-of-rules style. The plugin's `error-reference.md` is structured per-error-type (Address validation failures, No rates returned, etc.) and is much more comprehensive. Plugin wins on this content — likely no backport needed in this direction.

- [ ] **Step 3: Check for any rule in the clawhub list that isn't in error-reference.md.**

Cross-check each rule:
- "Never guess parcel dimensions, weight, customs values, HS codes, or signer names. Ask the user." — exists in plugin error-reference.md "General Rules" section.
- "Do not auto-retry transport, auth, or rate-limit errors." — exists.
- "Parcel dimensions and weight must be strings." — verify in plugin (likely in rate-shopping skill).
- "Label URLs are S3 signed URLs. Always display the complete URL — truncating breaks the signature." — verify in plugin (likely in label-purchase skill).
- "Rates expire after 7 days." — verify in plugin.
- "No rates? Validate addresses first, then check dimensions, then carrier-accounts-list." — exists in plugin error-reference.md.
- "Not found" errors: verify API key mode matches the data — verify in plugin test-mode.md.

For any rule missing in the plugin, add it to the appropriate plugin file.

- [ ] **Step 4: Reconcile Security & Data Transparency.**

The clawhub version describes the local-npm transport ("All data is sent to Shippo's API via the local @shippo/shippo-mcp server (stdio transport) running on the user's machine.").

The plugin's hosted-Gram transport has a *different* security story ("All data is sent to Shippo's API via the MCP server hosted at app.getgram.ai/mcp/shippo-mcp-beta.") — this lives in the plugin's top-level README.md, not in any skill or reference.

**These should NOT be merged.** Each channel correctly describes its own transport. Skip the security-section reconciliation. Add a note to this task: "Security sections intentionally divergent — different transports, different stories."

- [ ] **Step 5: Commit any plugin-side rule additions found in Step 3.**

```bash
git add skills/shippo/references/error-reference.md  # and any other touched files
git commit -m "Backport general error-handling rules from ClawHub bundle"
```

If no additions were needed, add a note: "No drift found — plugin's error coverage is a superset."

---

## Phase 3: Wire ClawHub publish from new location (no live publish yet)

### Task 3.1: Finalize the `.clawhub/plugin.json` manifest

**Files:**
- Modify: `.clawhub/plugin.json`

- [ ] **Step 1: Reconcile against Phase 0.1 findings.**

Open `.clawhub/plugin.json`. Compare its schema against whatever ClawHub's CLI expects (per Phase 0.1 findings). Update `name`, `version`, `skills` array, and any other required fields (likely `description`, `author`, `homepage`, `license`, `keywords` — mirror the existing `clawhub/shippo-official/SKILL.md` frontmatter).

- [ ] **Step 2: Validate.**

Run:
```bash
cd /Users/wyatt.reid/Desktop/official/shippo-claude-plugin
npx clawhub@latest skill validate clawhub/shippo-official
```
(Or `plugin validate`, per Phase 0.1 findings.)

Expected: exit 0, no errors.

- [ ] **Step 3: Commit.**

```bash
git add .clawhub/plugin.json
git commit -m "Finalize ClawHub plugin manifest"
```

### Task 3.2: Dry-run publish from the new location

**Files:** none (verification only)

- [ ] **Step 1: Run the publish command in dry-run mode.**

Per Phase 0.1 findings, the CLI may support `--dry-run`, `--check`, or a sandbox slug. Run whichever is correct. Example shape:
```bash
cd /Users/wyatt.reid/Desktop/official/shippo-claude-plugin
npx clawhub@latest skill publish clawhub/shippo-official --dry-run
```

- [ ] **Step 2: Inspect the dry-run output.**

The output should describe what would be published — file list, slug, version. Verify:
- Slug is `shippo-official` (matches the existing live skill)
- Version is `1.0.3` (matches the SKILL.md frontmatter)
- File list excludes the .clawhubignore'd paths (`.claude-plugin/`, `.mcp.json`, `skills/`, `examples/`, `docs/`, etc.) — none of these should appear in the publish bundle.
- File list INCLUDES `clawhub/shippo-official/SKILL.md` and the four references.

If the file list is wrong, fix `.clawhubignore` before proceeding. **Do not proceed to Phase 4 until dry-run output exactly matches what's expected.**

- [ ] **Step 3: Save the dry-run output to a scratch file for diff in Phase 4.**

Run (use the actual command from Step 1):
```bash
npx clawhub@latest skill publish clawhub/shippo-official --dry-run > /tmp/clawhub-dryrun-new.txt 2>&1
```

- [ ] **Step 4: Run the same dry-run from the standalone repo for comparison.**

```bash
cd /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill
git checkout wyatt/sync-with-plugin
npx clawhub@latest skill publish . --dry-run > /tmp/clawhub-dryrun-old.txt 2>&1
```

- [ ] **Step 5: Diff the two dry-runs.**

```bash
diff /tmp/clawhub-dryrun-old.txt /tmp/clawhub-dryrun-new.txt
```

Expected differences:
- Source-path lines (the new bundle is rooted at `clawhub/shippo-official/`; the old at `.`)
- File-path lines reflect the new sub-directory layout

NOT expected:
- Different file *count* (the same logical files should publish from both)
- Different slug
- Different version
- Different file *content* (the SKILL.md and references should be byte-identical)

If any non-expected differences appear, debug `.clawhubignore` and `.clawhub/plugin.json` until they go away.

- [ ] **Step 6: Skip commit — verification only.**

---

## Phase 4: Cutover (live publish + redirect)

**Do not start Phase 4 until Phase 3 dry-run diff is clean and Phase 0.2 redirect mechanism is confirmed.**

### Task 4.1: Open the redirect ticket / PR (per Phase 0.2 findings)

**Files:** depends on Phase 0.2 findings

- [ ] **Step 1: File the redirect.**

Per Phase 0.2 findings:
- If ClawHub has a CLI command: run it.
- If it's a PR against `openclaw/clawhub`: open the PR with the new source-repo URL.
- If it's an issue/email to ClawHub maintainers: send it now.

Reference the new repo source: `github.com/goshippo/shippo-claude-plugin` at path `clawhub/shippo-official/`.

- [ ] **Step 2: Wait for redirect confirmation before publishing.**

If the redirect is gated on maintainer review, do NOT proceed to Task 4.2 until confirmation is received. Park here.

If the redirect is self-service (CLI command or instant config), proceed.

- [ ] **Step 3: Document the redirect path in this plan.**

Append a `### Task 4.1 record` subsection with:
- Redirect mechanism used
- Timestamp
- Confirmation reference (PR URL, ticket ID, etc.)

### Task 4.2: Merge the plugin branch to `main` (gates the publish)

**Files:** none (git operation)

- [ ] **Step 1: Run the full validation chain one more time.**

```bash
cd /Users/wyatt.reid/Desktop/official/shippo-claude-plugin
git status   # expect: clean working tree, on wyatt/clawhub-alignment
npx clawhub@latest skill validate clawhub/shippo-official   # expect: pass
npx clawhub@latest skill publish clawhub/shippo-official --dry-run   # expect: same output as Phase 3
```

- [ ] **Step 2: Open a PR against `goshippo/shippo-claude-plugin` `main`.**

```bash
gh pr create --title "Consolidate ClawHub bundle into plugin repo" --body "$(cat <<'EOF'
## Summary
- Move the shippo-official ClawHub skill content into clawhub/shippo-official/ inside this repo
- Add .clawhub/ manifest dir and .clawhubignore for ClawHub publish from this repo
- Backport response-envelope and non-envelope-error guidance into plugin reference docs
- Reconcile any content drift between the per-skill plugin views and the consolidated ClawHub view

## Test plan
- [ ] `npx clawhub@latest skill validate clawhub/shippo-official` passes
- [ ] `npx clawhub@latest skill publish clawhub/shippo-official --dry-run` lists expected files
- [ ] ClawHub redirect filed/confirmed (Task 4.1)
- [ ] After merge, run actual `clawhub skill publish` (Task 4.3) and verify shippo-official slug shows new version live
- [ ] After publish, `npx clawhub install shippo` returns the new content (Task 4.4)
EOF
)"
```

- [ ] **Step 3: Wait for review and merge.**

Do not proceed until the PR is merged into `main`.

### Task 4.3: Publish from the new repo location

**Files:** none (publish operation)

- [ ] **Step 1: Pull `main` locally.**

```bash
cd /Users/wyatt.reid/Desktop/official/shippo-claude-plugin
git checkout main
git pull
```

- [ ] **Step 2: Run the live publish.**

```bash
npx clawhub@latest skill publish clawhub/shippo-official
```
Expected: success message with new published version. The slug `shippo-official` should now be served from the new repo source.

- [ ] **Step 3: Verify the publish.**

Open `https://clawhub.ai/skills/shippo-official` (or whatever the canonical install URL is) in a browser. Verify:
- Version reflects the new publish
- Source repo link points at `goshippo/shippo-claude-plugin`
- Description and metadata match SKILL.md frontmatter

### Task 4.4: Verify install path

**Files:** none (verification)

- [ ] **Step 1: Install fresh from ClawHub in a clean directory.**

```bash
cd /tmp
mkdir clawhub-verify-install
cd clawhub-verify-install
npx clawhub@latest install shippo
```
Expected: install succeeds. The skill folder dropped into the working directory should match `clawhub/shippo-official/` from the plugin repo (same SKILL.md, same references, byte-identical).

- [ ] **Step 2: Diff against the source.**

```bash
diff -r .skills/shippo-official /Users/wyatt.reid/Desktop/official/shippo-claude-plugin/clawhub/shippo-official
```
(Adjust install path per Step 1 actual output.)
Expected: no diff (byte-identical content).

- [ ] **Step 3: Document the verified install path.**

Append a `### Task 4.4 record` subsection with the install command tested, the resulting on-disk path, and the diff result.

### Task 4.5: Update standalone `shippo-clawhub-skill` repo to point at new home

**Files:**
- Modify: `shippo-clawhub-skill/README.md` (entirely replaced with a redirect notice)

- [ ] **Step 1: Switch to the standalone repo `main` and create a new branch.**

```bash
cd /Users/wyatt.reid/Desktop/official/shippo-clawhub-skill
git checkout main
git pull
git checkout -b wyatt/archive-redirect
```

- [ ] **Step 2: Replace `README.md` with a redirect notice.**

Write:
```markdown
# Shippo ClawHub Skill — moved

This skill has moved to `goshippo/shippo-claude-plugin` at path
[`clawhub/shippo-official/`](https://github.com/goshippo/shippo-claude-plugin/tree/main/clawhub/shippo-official).

The ClawHub install path is unchanged:

\`\`\`bash
npx clawhub@latest install shippo
\`\`\`

This repo is archived. New issues / PRs should be filed against
[goshippo/shippo-claude-plugin](https://github.com/goshippo/shippo-claude-plugin).
```

- [ ] **Step 3: Commit and push.**

```bash
git add README.md
git commit -m "Archive: skill moved to goshippo/shippo-claude-plugin"
gh pr create --title "Archive: redirect to consolidated plugin repo" --body "Skill content moved to goshippo/shippo-claude-plugin/clawhub/shippo-official/. ClawHub install slug unchanged."
```

- [ ] **Step 4: After merge, archive the GitHub repo.**

```bash
gh repo archive goshippo/shippo-clawhub-skill --yes
```

This locks the repo as read-only. Existing clones still work; new contributions go to the consolidated repo.

- [ ] **Step 5: Confirm archival.**

```bash
gh repo view goshippo/shippo-clawhub-skill --json isArchived
```
Expected: `{"isArchived": true}`.

---

## Phase 5: Memory and documentation

### Task 5.1: Update auto-memory

**Files:**
- Modify: `/Users/wyatt.reid/.claude/projects/-Users-wyatt-reid-Desktop-official/memory/project_shippo_skills_in_gram.md`
- Modify: `/Users/wyatt.reid/.claude/projects/-Users-wyatt-reid-Desktop-official/memory/MEMORY.md`

- [ ] **Step 1: Update the three-repo memory.**

The "shippo-clawhub-skill" repo no longer exists as a separate distribution surface. Edit `project_shippo_skills_in_gram.md`:
- Update the table to show `shippo-claude-plugin` covering both Claude Code and ClawHub channels
- Note the consolidation date and the redirect status
- Remove `shippo-clawhub-skill` row, replace with a one-line "archived" mention

- [ ] **Step 2: Update the MEMORY.md index pointer.**

Adjust the one-line description to match.

### Task 5.2: Update plugin README

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Add a Distribution section.**

After the existing Quick Start, add:
```markdown
## Distribution

This repo is the source of truth for two distribution channels:
- **Claude Code plugin** — install via `git clone` + `claude --plugin-dir`. Skills live in `skills/`.
- **ClawHub skill** (`shippo-official`) — install via `npx clawhub install shippo`. Source lives in `clawhub/shippo-official/`.

Both channels point at the same Gram-hosted Shippo MCP server for tool execution. Workflow content is hand-curated to stay in parity between the two views; updates touch both in the same PR.
```

- [ ] **Step 2: Commit.**

```bash
git add README.md
git commit -m "Document Claude Code + ClawHub distribution channels"
```

---

## Self-review checklist (run before handoff)

- [ ] Every Phase 0 task has a `### findings` or `### decision` subsection where the executor will record what they learned. The plan does not assume answers it doesn't have.
- [ ] No tasks reference scripts, build pipelines, or generation tooling. Per user constraint: surgical hand-edits only.
- [ ] No commits to `main` of either repo before Phase 4. Phase 1–3 all live on `wyatt/clawhub-alignment` only.
- [ ] The branch `wyatt/sync-with-plugin` on `shippo-clawhub-skill` is treated as the canonical source for content copies (Phase 1.2, 1.3, 1.4) — NOT `main`.
- [ ] `.clawhubignore` is configured for the *plugin-repo* context (Phase 1.4 Step 3), not just copied verbatim from the standalone repo.
- [ ] `customs-guide.md` and `tool-reference.md` exist in TWO locations after Phase 1 (`clawhub/shippo-official/references/` and `skills/shippo/references/`) — by design, because ClawHub bundles must be self-contained. File-structure section calls this out.
- [ ] Phase 2 has explicit "no drift found" exit paths so the executor doesn't fabricate edits where parity already exists.
- [ ] The Security & Data Transparency divergence (Phase 2.8 Step 4) is preserved, not merged — different transports, different stories.
- [ ] Phase 4 publish is gated on Phase 0.2 redirect confirmation (Phase 4.1 Step 2).
- [ ] OAuth transport rewrite (~2026-05-15) is explicitly out-of-scope.
- [ ] Phase 5 updates auto-memory so future sessions know `shippo-clawhub-skill` is archived.
