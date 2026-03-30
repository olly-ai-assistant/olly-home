---
name: dependency-changelog
description: Generates a changelog of dependency changes between two commits or tags. Use when asked to "show dependency changes", "what changed in dependencies", "dependency diff", or when reviewing dependency updates, auditing security changes, or preparing release notes focused on third-party changes.
---

# Dependency Changelog

This skill generates a structured changelog of dependency changes between two commits, tags, or branches in a project.

## When to Use This Skill

Use this skill when:
- Asked to "show dependency changes", "what changed in dependencies", or "dependency diff"
- Reviewing dependency updates or security patches
- Preparing release notes focused on third-party changes
- Auditing dependency updates between versions
- Comparing dependency states across branches
- Investigating dependency-related bugs or regressions

## Core Commands

### npm/yarn/pnpm Projects

```bash
# Show dependency tree changes between commits
git diff <commit1> <commit2> -- package.json package-lock.json yarn.lock pnpm-lock.yaml

# Use npm-check-updates to show outdated packages
npx npm-check-updates

# Use npm-check to show installed version vs required
npx npm-check

# Diff lockfiles
git diff <commit1> <commit2> -- '**/package-lock.json' '**/yarn.lock' '**/pnpm-lock.yaml'
```

### Python (requirements.txt, Pipfile, pyproject.toml)

```bash
# Freeze current state
pip freeze > current_requirements.txt

# Compare with another commit
git diff <commit1> <commit2> -- requirements.txt Pipfile pyproject.toml

# Use pipreqs to generate requirements
pipreqs . --diff
```

### Go (go.mod, go.sum)

```bash
# Show module changes
git diff <commit1> <commit2> -- go.mod go.sum

# Tidy and show changes
go mod tidy && git diff go.mod
```

### Docker (requirements in Dockerfiles)

```bash
# Show base image changes
git diff <commit1> <commit2> -- Dockerfile* | grep -i "from\|run.*pip\|run.*npm\|run.*apt"

# Multi-stage diff
git diff <commit1> <commit2> -- 'Dockerfile*'
```

## Output Format

Present findings as a structured changelog:

```markdown
## Dependency Changelog

**From:** `<commit/tag>` → **To:** `<commit/tag>`

### Added
| Package | Version | Purpose |
|---------|---------|---------|
| `package-name` | `^1.2.0` | Short description |

### Updated
| Package | Old Version | New Version | Breaking? |
|---------|-------------|-------------|-----------|
| `lodash` | `4.17.20` | `4.17.21` | No |
| `react` | `17.0.0` | `18.0.0` | ⚠️ Yes |

### Removed
| Package | Last Version | Reason |
|---------|--------------|--------|
| `deprecated-pkg` | `2.0.0` | Replaced by `new-pkg` |

### Security Updates
| Package | Version | CVE/Issue |
|---------|---------|-----------|
| `axios` | `0.21.0 → 0.21.2` | GHSA-wf5p-g76f-4vq3 |

### Recommendations
- Run `npm audit` / `pip audit` / `go audit` to verify
- Review breaking change logs for major updates
- Test in staging before production deployment
```

## Automated Tools

| Tool | Language | Command |
|------|----------|---------|
| `npm-check-updates` | JS/TS | `npx npm-check-updates` |
| `depcheck` | JS/TS | `npx depcheck` |
| `pipdate` | Python | `pipdate` |
| `pur` | Python | `npx pur` |
| `go-modiff` | Go | `go run github.com/igoihman/gomodiff/cmd/gomodiff@latest` |
| ` Renovate Bot` | All | GitHub App/Bot |

## Lockfile-Only Diff

For a cleaner diff that ignores lockfile formatting changes:

```bash
# npm
git diff <commit1> <commit2> -- package-lock.json | grep -E "^\+|\-" | grep -v "^\-\-\-\|^\+\+\+" | head -100

# yarn
git diff <commit1> <commit2> -- yarn.lock | grep "^[@| ].*" | sort -u

# Show just version bumps
npm diff --diff-modules
```

## Tips

- Use `--name-only` first for a quick summary before full diff
- Lockfiles can have massive diffs due to formatting; focus on `package.json` changes first
- Major version bumps often indicate breaking changes — always flag these
- For monorepos, check each sub-package's lockfile separately
- CI systems (Dependabot, Renovate) often provide nice changelogs in PR descriptions
