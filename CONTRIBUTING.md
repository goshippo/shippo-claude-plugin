# Contributing to the Shippo Claude Code Plugin

Thank you for your interest in contributing! This guide covers local setup, plugin structure, and how to submit changes.

## Local Development Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/goshippo/shippo-claude-plugin.git
   cd shippo-claude-plugin
   ```

2. **Set your Shippo API key**

   Copy the example env file and add your key:

   ```bash
   cp .env.example .env
   # Edit .env and set SHIPPO_API_KEY
   ```

   You can use a [Shippo test token](https://docs.goshippo.com/docs/guides_general/authentication/) (prefixed with `shippo_test_`) for development.

3. **Run the plugin locally**

   ```bash
   claude --plugin-dir ./shippo-claude-plugin
   ```

## Testing Changes

Load the plugin with `--plugin-dir` and exercise the skills:

```bash
claude --plugin-dir ./shippo-claude-plugin
```

Inside the session, try:

- `/shippo:address-validation`, `/shippo:rate-shopping`, `/shippo:label-purchase`, `/shippo:tracking`, `/shippo:batch-shipping`, `/shippo:shipping-analysis`

Before submitting, validate the plugin structure:

```bash
claude plugin validate .
```

## Plugin Structure

```
shippo-claude-plugin/
  .claude-plugin/
    plugin.json           # Plugin manifest (name, version, metadata)
  .mcp.json               # MCP server connection config
  skills/
    address-validation/
      SKILL.md
    rate-shopping/
      SKILL.md
    label-purchase/
      SKILL.md
    tracking/
      SKILL.md
    batch-shipping/
      SKILL.md
    shipping-analysis/
      SKILL.md
    shippo/
      references/         # Shared knowledge docs (10 reference files)
  examples/               # Sample input files (CSV, JSON)
  README.md
```

### Key concepts

- **`.claude-plugin/plugin.json`** -- Plugin manifest. Defines name, version, author, and keywords.
- **`.mcp.json`** -- Connects Claude Code to the Shippo MCP server (hosted via Speakeasy/Gram).
- **`skills/`** -- Each subdirectory is a skill. Claude auto-routes to the correct one based on context.
- **`skills/shippo/references/`** -- Domain knowledge files. Skills reference these for carrier details, field formats, error codes, etc.
- **`examples/`** -- Sample CSV and JSON files users can reference for batch and address workflows.

## Adding a New Skill

1. Create a new directory under `skills/`:

   ```
   skills/my-new-skill/
     SKILL.md
   ```

2. Add YAML frontmatter at the top of `SKILL.md`:

   ```yaml
   ---
   name: my-new-skill
   description: Short description of what this skill does
   ---
   ```

3. Write the skill body. Keep it **workflow-focused** -- step-by-step instructions for Claude, not domain knowledge dumps. Reference files in `references/` for domain details.

4. Validate:

   ```bash
   claude plugin validate .
   ```

## Adding a New Reference File

1. Create a Markdown file in `skills/shippo/references/`:

   ```
   skills/shippo/references/my-topic.md
   ```

2. Write clear, factual reference content. Reference files hold **domain knowledge** (field definitions, carrier rules, format specs) -- not workflows.

3. Reference it from any skill that needs it:

   ```markdown
   See `references/my-topic.md` for details.
   ```

## Code Style Guidelines

- **SKILL.md files** should be workflow-focused: numbered steps, decision tables, tool call sequences. They tell Claude *what to do*.
- **Reference files** should contain domain knowledge: field definitions, carrier lists, format specs, error codes. They tell Claude *what things mean*.
- Keep both concise. Claude reads these on every invocation -- brevity matters.
- Use Markdown tables for structured data (field mappings, decision guides).
- Avoid duplicating content across files. Put it in one place and reference it.

## Submitting Changes

1. Fork the repository.
2. Create a feature branch:

   ```bash
   git checkout -b my-feature
   ```

3. Make your changes and validate:

   ```bash
   claude plugin validate .
   ```

4. Commit with a clear message describing what changed and why.
5. Open a pull request against `main`.

### PR checklist

- [ ] `claude plugin validate .` passes
- [ ] New skills have frontmatter (`name`, `description`)
- [ ] Tested locally with `--plugin-dir`
- [ ] No API keys or secrets committed

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](LICENSE).
