---
name: git-churn-analyst
description: Analyzes git history to identify high-churn files (files that change frequently), technical debt hotspots, unstable interfaces, and problematic modules. Use when asked to "analyze git churn", "find unstable files", "identify tech debt hotspots", "find frequently changed files", or when doing code health assessments, planning refactors, or prioritizing code review efforts.
---

# Git Churn Analyst

Identify files that change frequently — a key signal of technical debt, unstable interfaces, or poorly designed modules.

## When to Use This Skill

- Asked to "analyze git churn" or "find frequently changed files"
- Code health assessments or tech debt reviews
- Planning refactors — start with high-churn files
- Prioritizing code review bandwidth
- Identifying unstable interfaces or fragile modules
- Onboarding — understand which parts of the codebase are "hot"

## Core Analysis Commands

### 1. File Churn (Most Changed Files)

```bash
# Show files changed most in recent N commits
git log --since="90 days ago" --format=format: --name-only | grep -v '^$' | sort | uniq -c | sort -rn | head -30

# Churn across entire history
git log --format=format: --name-only | grep -v '^$' | sort | uniq -c | sort -rn | head -30

# Churn for specific directory
git log --since="1 year" --format=format: --name-only -- "src/api/" | grep -v '^$' | sort | uniq -c | sort -rn

# Churn grouped by directory
git log --since="90 days" --name-only --format='' | sed 's|/[^/]*$||' | sort | uniq -c | sort -rn
```

### 2. Commit Frequency Over Time

```bash
# Commits per week (last 6 months)
git log --since="6 months ago" --format=format:"%ai" -- . | cut -d' ' -f1 | sort | uniq -c | awk '{print $2, $1}'

# Show commits by author
git log --since="90 days" --format=format:"%aN" -- . | sort | uniq -c | sort -rn

# Find "busy" times — files changed together
git log --since="30 days" --format=format:"%h %s" --name-only | grep -A10 "api\|router\|controller" | head -40
```

### 3. Hotspot Analysis

```bash
# Lines added/removed per file (churn + volume)
git log --since="1 year" --format=format: --numstat -- . | awk '
  { added += $1; removed += $2; files++ }
  END { print "Total changes:", files, "files | +" added "lines | -" removed "lines" }
'

# Files with most commits AND most lines changed
git log --since="90 days" --format=format: --name-only --numstat | paste -d'|' - - | \
  awk -F'|' '{print NR, $1, $3, $4}' | head -20

# Identify "volatile" files (many commits, few lines = unstable interface)
git log --since="90 days" --format=format:"%H" --name-only -- . | grep -v '^$' | sort | uniq | wc -l
```

### 4. Blame-Weighted Churn

```bash
# Files with most lines touched by many authors (shared instability)
git blame --line-porcelain HEAD -- src/ | grep "^author " | sort | uniq -c | sort -rn | head -20

# Find files touched by 5+ different authors in 90 days
git log --since="90 days" --format=format:"%aN" --name-only -- "src/" | grep -v '^$' | \
  sort -u | awk '{print NR, $0}' | sort -k2 | uniq -c | awk '$1 >= 5 {print $2, "(touched by", $1, "authors)"}'
```

## Output Format

```markdown
## Git Churn Report

**Repo:** `{name}` | **Period:** `{last 90 days}` | **Date:** `{YYYY-MM-DD}`

### Top 10 Most-Changed Files
| File | Changes | Authors | Risk |
|------|---------|---------|------|
| `src/api/routes.js` | 47 | 6 | 🔴 High |
| `src/utils/helpers.py` | 38 | 4 | 🟡 Medium |
| `src/core/engine.ts` | 29 | 3 | 🟡 Medium |

### Hotspot Directories
| Directory | Changes | Files |
|-----------|---------|-------|
| `src/api/` | 112 | 14 |
| `src/core/` | 87 | 9 |

### Risk Signals
- **Unstable interfaces:** 4 files changed by 6+ authors in 90 days
- **High-churn warning:** 3 files exceed 30 changes in 90 days
- **Bus factor risk:** `src/utils/` touched by only 1 author

### Recommendations
1. 🔴 Investigate `src/api/routes.js` — 47 changes, 6 authors = unstable interface, needs refactor
2. 🟡 `src/utils/helpers.py` — consider splitting by concern
3. ✅ `src/core/engine.ts` — stable given domain nature, document if business-critical
```

## Interpretation Guide

| Churn Level | Changes/90d | Action |
|-------------|-------------|--------|
| 🔴 Critical | 30+ | Immediate refactor candidate |
| 🟡 Medium | 15-30 | Monitor, consider splitting |
| ✅ Low | <15 | Acceptable for domain |

**Red flags:**
- Same file changed by many authors (design disagreement)
- High churn in core domain logic (fundamental design issues)
- No churn in thick wrappers (untested code?)

## Quick One-Liners

```bash
# Top 10 most changed files (last year)
git log --format=format: --name-only | sort | uniq -c | sort -rn | head -10

# Files changed in last 30 days
git diff --name-only HEAD~30 HEAD

# Who touched what (last 90 days)
git log --format=format:"%aN" --since="90 days" | sort | uniq -c | sort -rn

# Churn by extension
git log --since="90 days" --name-only --format='' | grep '\.[jtsx]\|py\|go$' | sed 's/.*\.//' | sort | uniq -c | sort -rn
```
