---
name: dependency-release-monitor
description: Monitors project dependencies for new releases and generates changelog-style reports. Use when asked to "check for outdated dependencies", "monitor dependency releases", "what's new in [package] releases", "dependency alert", or when auditing a project's dependency freshness. Works with npm, PyPI, GitHub, and Docker Hub.
---

# Dependency Release Monitor

Check for new releases of your project's dependencies and generate actionable reports.

## When to Use This Skill

- Asked to "check for outdated dependencies" or "monitor releases"
- Auditing dependency freshness before a release
- Investigating what changed in a specific package
- Setting up automated dependency alerts
- Security review: known vulnerabilities in outdated packages

## Core Commands

### npm/yarn/pnpm Projects

```bash
# Interactive update checker
npx npm-check-updates

# Dry run — show what would be updated
npx npm-check-updates --dry

# Show latest version per package
npm outdated

# Audit for vulnerabilities
npm audit --audit-level=moderate
```

### Python (PyPI)

```bash
# Check outdated packages
pip list --outdated

# Upgrade single package
pip install --upgrade <package>

# Use pipdate for all upgrades
pipdate
```

### GitHub Repos (watch releases)

```bash
# Check latest release via GitHub API
curl -s https://api.github.com/repos/<owner>/<repo>/releases/latest | jq '.tag_name, .body'

# Watch a repo (requires gh cli)
gh repo watch <owner>/<repo>
```

### Docker Images

```bash
# Check for updated base images
docker pull <image>:latest
docker images | grep <image>

# Use Renovate Bot for Docker updates
```

## Output Format

```markdown
## Dependency Release Monitor Report

**Project:** `{project_name}`
**Date:** `{YYYY-MM-DD}`
**Checked:** `{count}` packages

### 🔴 Critical Updates (Security/Breaking)
| Package | Current | Latest | Type | Notes |
|---------|---------|--------|------|-------|
| `axios` | 0.21.0 | 0.21.2 | Security | GHSA-wf5p-g76f-4vq3 |
| `lodash` | 4.17.20 | 4.17.21 | Security | CVE-2021-23337 |

### 🟡 Available Updates
| Package | Current | Latest | Age | Changelog |
|---------|---------|--------|-----|-----------|
| `react` | 17.0.0 | 18.0.0 | 12mo | ⚠️ Breaking |
| `zod` | 3.19.0 | 3.20.0 | 3w | Minor |

### ✅ Up to Date
`express`, `typescript`, `eslint`, `prettier` (12 packages)

### Recommendations
- Run `npm audit` to verify security impact
- Test major version bumps in staging first
- Consider Renovate Bot or Dependabot for automated PRs
```

## Automated Monitoring Setup

### Dependabot (GitHub)
Already built into GitHub — enable in repo Settings → Code security and analysis → Dependabot

### Renovate Bot
```bash
# Install Renovate on a repo
npx renovate-configure
# Or use GitHub App: https://github.com/apps/renovate
```

### npm-alerts (CI/CD)
```yaml
# GitHub Actions example
- name: Check outdated dependencies
  run: npx npm-check-updates --exit-level=warn
```

### PyPI RSS/JSON Feeds
```bash
# Monitor PyPI via JSON API
curl -s https://pypi.org/pypi/<package>/json | jq '.info.version, .releases[].upload_time'
```

## Monitoring Multiple Sources

```bash
# All npm deps in one shot
npx npm-check-updates -u && npm install

# Python deps
pip freeze > requirements.txt
pipdate

# Docker base images — check regularly
docker pull alpine:latest
```

## Pro Tips

- **Lockfiles are your friend** — always commit `package-lock.json` / `requirements.lock`
- **Minor/patch updates** are usually safe to auto-merge
- **Major version bumps** always need human review
- **Set up Dependabot** on GitHub for automatic PRs with changelogs
- **Audit logs** — keep track of when you updated critical packages
