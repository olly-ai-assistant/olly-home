---
name: changelog-auto-generator
description: This skill should be used when generating, updating, or managing changelogs. Triggers on "update changelog", "generate changelog", "auto changelog", "keep a changelog", or when commits need to be summarized into release notes.
---

# Changelog Auto-Generator

Automatically generate or update changelogs from git commit history.

## When to Use This Skill

- Before a release or deployment
- When asked to "update the changelog"
- After a significant number of commits that need summarizing
- When generating release notes from commit messages

## Quick Usage

```bash
# Generate changelog from commits since last tag
git log --pretty=format:"%h %s" --since="2 weeks ago" | head -50

# Show conventional commits since last release
git log --pretty=format:"%s" --grep="feat\|fix\|docs\|refactor" -i
```

## Changelog Format (Keep a Changelog)

Use this structure:

```markdown
# Changelog

All notable changes are documented here.

## [Unreleased]

### Added
- (new features)

### Changed
- (changes in existing functionality)

### Deprecated
- (soon-to-be removed features)

### Removed
- (removed features)

### Fixed
- (bug fixes)

### Security
- (security improvements)

## [1.2.3] - YYYY-MM-DD
### Added
- (features since last release)
```

## Step-by-Step Process

### 1. Collect Commit History

```bash
# Since last tag (best for releases)
git log $(git describe --tags --abbrev=0)..HEAD --pretty=format:"%h %s %b"

# Since specific date
git log --since="2026-01-01" --pretty=format:"%h %s"

# Since last changelog entry (manual approach)
git log --since=$(grep -m1 "## \[" CHANGELOG.md | grep -oP '\d{4}-\d{2}-\d{2}')..HEAD
```

### 2. Categorize Commits

Parse each commit and categorize by type:

| Type | Keywords | Section |
|------|----------|---------|
| feat | `feat:`, `feat(` | Added |
| fix | `fix:`, `fix(` | Fixed |
| docs | `docs:` | Changed |
| refactor | `refactor:` | Changed |
| perf | `perf:` | Changed |
| test | `test:` | Changed |
| chore | `chore:`, `bump`, `bump:` | Changed |
| BREAKING | `BREAKING CHANGE:` | (note in section) |

### 3. Group by Type

Organize commits into the changelog sections. Remove:
- Merge commits
- commits starting with `Merge`
- chore-only updates that aren't interesting to users

### 4. Insert into CHANGELOG.md

Insert new entries under `[Unreleased]` section with today's date:

```markdown
## [Unreleased] - YYYY-MM-DD

### Added
- feat: add user profile API endpoint
```

Or for a dated release:

```markdown
## [1.2.3] - YYYY-MM-DD

### Added
- feature description
```

## Automation with Release Please

For ongoing automation, suggest adding `release-please-action`:

```yaml
# .github/workflows/release-please.yml
name: Release Please
on:
  push:
    branches: [main]
jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
```

## Tips

- Keep descriptions **user-facing** — explain what the change does for users, not developers
- Link to issues/PRs when available: `- add rate limiting (#123)`
- Group related commits together when they form a feature
- Keep each entry concise (under 72 chars)
- Remove duplicate entries if multiple commits describe the same change

## Anti-Patterns

| Bad | Better |
|-----|--------|
| `Updated dependencies` | `Bump Node.js to v22 from v20` |
| `Fixed stuff` | `fix: prevent login timeout after 30 min inactivity` |
| `Multiple bug fixes` | `fix: correct timezone handling in date picker` |
| `WIP feature` | (don't include until ready) |
