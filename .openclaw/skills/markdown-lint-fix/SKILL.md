---
name: markdown-lint-fix
description: Lints, validates, and fixes Markdown files for style and syntax issues. Use when asked to "lint markdown", "fix markdown", "validate markdown", "check markdown style", or when fixing documentation quality, formatting inconsistencies, or markdown parsing errors in docs, READMEs, or Obsidian vaults.
---

# Markdown Lint & Fix

This skill lints, validates, and fixes Markdown files for style, syntax, and consistency issues. Works with standalone markdown files, Obsidian vaults, documentation sites, and README files.

## When to Use This Skill

Use this skill when:
- Asked to "lint markdown", "fix markdown", or "validate markdown"
- Checking documentation quality or formatting consistency
- Fixing markdown parsing errors or broken links
- Pre-commit hook validation for markdown
- Obsidian vault maintenance
- GitHub wiki or static site documentation (Docusaurus, MkDocs, Hugo, etc.)
- Reviewing markdown-heavy pull requests

## Core Tools

### markdownlint (CLI)

```bash
# Install
npm install -g markdownlint-cli

# Lint a file
npx markdownlint path/to/file.md

# Lint recursively
npx markdownlint "**/*.md"

# Fix auto-fixable issues
npx markdownlint-cli --fix "**/*.md"

# With config file
npx markdownlint -c .markdownlint.json "**/*.md"
```

### markdownlint-cli2

```bash
# Install
npm install -g markdownlint-cli2

# Lint with glob
markdownlint-cli2 "src/**/*.md"

# Fix and fail on unfixable
markdownlint-cli2 "**/*.md" --fix
```

### Textlint (extensible)

```bash
# Install
npm install -g textlint textlint-rule-preset-jtf-style

# Run
npx textlint README.md

# With preset
npx textlint --preset preset-name README.md
```

### prettier (markdown)

```bash
# Install
npm install -D prettier

# Format a file
npx prettier --write path/to/file.md

# Format all markdown files
npx prettier --write "**/*.md"

# Check only (no changes)
npx prettier --check "**/*.md"
```

### remark (unified ecosystem)

```bash
# Install
npm install -g remark-cli

# Lint markdown
npx remark README.md

# Fix
npx remark README.md --output

# With plugins
npx remark --use remark-lint --use remark-preset-lint-consistent README.md
```

## Common .markdownlint.json Config

```json
{
  "default": true,
  "MD013": false,
  "MD033": false,
  "MD041": false,
  "MD004": { "style": "dash" },
  "MD029": { "style": "ordered" },
  "MD036": false,
  "line-length": false,
  "no-hard-tabs": false,
  "single-hierarchy": false
}
```

## Typical Issues & Fixes

| Issue | Rule | Fix |
|-------|------|-----|
| Line too long | MD013 | Break lines at 80/120 chars |
| Multiple headings | MD022 | Add blank line between headings |
| Missing blank line around headers | MD012 | Add blank line before/after |
| Ordered list starting at >1 | MD029 | Use 1. for all ordered items |
| Inline HTML | MD033 | Convert to markdown or disable rule |
| First heading not h1 | MD041 | Start with # heading |
| Fenced code blocks without language | MD040 | Add language: ```python, ```bash |
| No newline at EOF | MD047 | Add trailing newline |
| Multiple consecutive blank lines | MD012 | Use single blank line |
| Trailing spaces | MD009 | Remove trailing whitespace |
| Hard tabs | MD010 | Replace tabs with spaces |

## Obsidian-Specific

```bash
# Install obsidian-linter (community plugin)
# Or use the CLI

# Vault-wide lint
find /path/to/vault -name "*.md" -exec npx markdownlint --fix {} \;

# Fix wikilinks
npx markdownlint --fix "**/*.md" -- --ext .md

# Check internal links
grep -r '\[\[.*\]\]' --include="*.md" . | grep -v '\[\[.*|.*\]\]'
```

## Pre-commit Hook Setup

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/igorshubovych/markdownlint-cli
    rev: v0.33.0
    hooks:
      - id: markdownlint

  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: v3.0.0
    hooks:
      - id: prettier
        args: [--parser, markdown]
```

## Output Format

```markdown
## Markdown Lint Report

**File:** `README.md`
**Issues found:** 5

| Line | Rule | Description | Severity | Auto-fixable |
|------|------|-------------|----------|--------------|
| 23 | MD013 | Line length exceeded (156 > 120) | Warning | No |
| 45 | MD029 | Ordered list item should start at 1 | Info | Yes |
| 67 | MD040 | Fenced code block should have language | Warning | Yes |
| 89 | MD009 | Trailing spaces | Info | Yes |
| 112 | MD041 | First line should be a heading | Warning | No |

**Auto-fixed:** 3 issues
**Manual review needed:** 2 issues
```

## Quick Fix All

```bash
# One-shot fix most common issues
npx prettier --write "**/*.md" && npx markdownlint --fix "**/*.md"
```

## Tips

- Always commit before bulk fixes — markdownlint can introduce unexpected changes
- Obsidian wikilinks (`[[note]]`) aren't standard markdown — disable MD013 for vaults
- Some rules are style preferences (MD004, MD029) — configure to match team conventions
- GitHub renders markdown differently than some parsers — test after fixing
- For very large vaults, process in batches to avoid timeouts
