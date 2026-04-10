---
name: secret-scanner
description: Scans codebases for accidentally committed secrets: API keys, passwords, tokens, private keys, credentials, and other sensitive data. Use when asked to "scan for secrets", "find leaked credentials", "check for API keys", "security audit", or when doing a code health check, onboarding review, or pre-release security sweep. Stops before exfiltrating findings — report locally only.
---

# Secret Scanner

Detect accidentally committed secrets before they reach production or public repositories.

## When to Use This Skill

- Pre-commit or pre-push security checks
- Onboarding reviews of new codebases
- Pre-release security sweeps
- After adding new integrations (API keys, tokens)
- Incident response — checking for leaked credentials
- GitHub/supabase integrations discovery

## Core Scanning

### 1. Pattern-Based Grep Scan

```bash
# High-confidence secret patterns
git log --since="1 year" --name-only --format=format: | sort -u | xargs grep -lE \
  '(password|passwd|pwd)\s*[:=]\s*["'"'"'][^"'"'"'\s]{4,}' 2>/dev/null

# API keys and tokens
grep -rEn \
  '(api[_-]?key|apikey|secret[_-]?key|access[_-]?token|auth[_-]?token|bearer)\s*[:=]\s*["'"'"'][A-Za-z0-9_\-]{20,}' \
  --include="*.py" --include="*.js" --include="*.ts" --include="*.json" --include="*.yaml" --include="*.yml" \
  --include="*.env*" --include="*.ini" --include="*.cfg" --include="*.conf" . 2>/dev/null

# Private keys
grep -rEn '-----BEGIN (RSA |EC |DSA |OPENSSH )PRIVATE KEY-----' . 2>/dev/null

# AWS keys
grep -rEn 'AKIA[0-9A-Z]{16}' . 2>/dev/null

# Slack tokens
grep -rEn 'xox[baprs]-[0-9a-zA-Z-]+' . 2>/dev/null

# GitHub tokens
grep -rEn 'ghp_[A-Za-z0-9]{36}|github_pat_[A-Za-z0-9_]{22,}' . 2>/dev/null

# Database connection strings with credentials
grep -rEn 'mongodb|postgres|mysql|redis|sqlite.*://[^:]+:[^@]+@' . 2>/dev/null

# JWT tokens
grep -rEn 'eyJ[A-Za-z0-9_-]+\.eyJ[A-Za-z0-9_-]+' . 2>/dev/null
```

### 2. File-Type Targeted Scan

```bash
# Scan specific file types likely to contain secrets
for ext in py js ts jsx tsx json yaml yml env cfg ini conf; do
  find . -name "*.$ext" ! -path "./node_modules/*" ! -path "./.git/*" ! -path "./venv/*" ! -path "./.venv/*" -exec grep -lE 'key|token|secret|password|credential' {} \;
done 2>/dev/null

# Check .env files for secrets (and if they're committed!)
find . -name ".env*" ! -path "./.git/*" -exec echo "=== {} ===" \; -exec cat {} \; 2>/dev/null
git log --all --full-history -- ".env*" | head -20
```

### 3. Full-Repository Scan with Tools

```bash
# Using truffleHog (deep scan, checks git history)
pip install truffleHog
trufflehog filesystem .

# Using git-secrets (if installed)
git secrets --scan -r .

# Using gitleaks (fast, CI-friendly)
docker run --rm -v "$(pwd):/repo" zricethezav/gitleaks:latest detect --source=/repo --verbose

# Using detect-secrets (baseline approach)
pip install detect-secrets
detect-secrets scan > .secrets.baseline
detect-secrets scan --baseline .secrets.baseline --update
```

### 4. Exclude Safe Patterns (Reduce Noise)

```bash
# Ignore test fixtures and documentation
grep -rEn 'key|token|secret' \
  --exclude-dir=node_modules \
  --exclude-dir=.git \
  --exclude-dir=venv \
  --exclude-dir=.venv \
  --exclude="*.test.*" \
  --exclude="*.spec.*" \
  --exclude="*_test.py" \
  --exclude="*.md" \
  --exclude="*.txt" \
  --exclude="*.lock" \
  . 2>/dev/null | grep -vE '(example|sample|test|placeholder|demo)' | head -30
```

## Output Format

```markdown
## Secret Scanner Report

**Repo:** `{name}` | **Date:** `{YYYY-MM-DD}` | **Scope:** `{full repo / last N commits}`

### Findings (sorted by severity)
| Severity | File | Line | Type | Match |
|----------|------|------|------|-------|
| 🔴 Critical | `config/prod.py` | 12 | AWS Key | `AKIAIOSFODNN7EXAMPLE` |
| 🔴 Critical | `src/auth.py` | 45 | Private Key | `-----BEGIN RSA PRIVATE KEY-----` |
| 🟡 High | `.env` | 3 | API Secret | `api_key = "sk-...` |
| 🟡 High | `settings.json` | 8 | Token | `bearer eyJ...` |
| 🟢 Medium | `tests/fixtures.py` | 22 | Password | `password: "test123"` |

### Commit History Exposure
- `.env` committed in `a3f2d1c` (2025-11-15) — still in repo history
- `config/old.py` contains hardcoded key in commit `b7c9e3a` (2025-10-02)

### Recommendations
1. 🔴 **Rotate** `config/prod.py` AWS key immediately (key exposed)
2. 🔴 **Remove** `.env` from git history: `git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch .env'`
3. 🟡 Add `git-secrets` or `detect-secrets` as pre-commit hook
4. 🟢 Replace test fixture passwords with environment variable references
```

## Response Protocol

**When secrets are found:**
1. **Do NOT log the actual secret value** — write `{REDACTED}` or `***`
2. Report the file, line number, and secret type only
3. Recommend rotation/remediation steps
4. If the repo is public or was public, escalate the risk

## Quick One-Liners

```bash
# Fast scan for common patterns
grep -rE 'api[_-]?key|secret[_-]?key|access[_-]?token' --include="*.py" --include="*.js" --include="*.json" . 2>/dev/null | grep -v node_modules

# Check if .env is committed
git log --all --full-history -- ".env*" | head -5

# Scan for AWS keys in git history
git log --all -p -S 'AKIA' --source --all | grep -E 'AKIA[0-9A-Z]{16}' | head -5

# List files containing "password"
find . -type f \( -name "*.py" -o -name "*.js" -o -name "*.json" -o -name "*.yaml" \) -exec grep -l 'password' {} \; 2>/dev/null

# Check for private keys in repo
find . -type f \( -name "*.pem" -o -name "*.key" -o -name "*.p8" \) ! -path "./.git/*" 2>/dev/null
```
