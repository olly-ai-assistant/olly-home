---
name: code-metrics-tracker
description: Tracks and reports code quality metrics — cyclomatic complexity, test coverage %, maintainability index, lines of code, and more. Use when asked to "track code metrics", "measure complexity", "coverage report", "maintainability score", "code health", or when monitoring technical debt, reviewing PR quality, or establishing quality gates. Supports JavaScript/TypeScript, Python, Go, and general file analysis.
---

# Code Metrics Tracker

Measure, track, and report code quality metrics across your project.

## When to Use This Skill

- Asked to "track code metrics" or "measure complexity"
- Reviewing PR quality or code health
- Establishing quality gates in CI/CD
- Monitoring technical debt over time
- Pre-release quality check
- Onboarding — understand codebase health at a glance

## Core Metrics

| Metric | What It Measures | Good | Warning | Critical |
|--------|-----------------|------|---------|----------|
| Cyclomatic Complexity | Code branching/fork count | 1-10 | 11-20 | 20+ |
| Coverage % | Lines/functions exercised by tests | 80%+ | 60-80% | <60% |
| Maintainability Index | Overall code health (0-100) | 80+ | 60-80 | <60 |
| Lines of Code | Project size (per file/module) | — | — | Context-dependent |
| Duplication % | Repeated code blocks | <3% | 3-5% | 5%+ |
| Comments % | Documentation ratio | 10-20% | <10% or >30% | — |

## JavaScript/TypeScript

```bash
# Complexity & coverage with ESLint + Jest
npm install --save-dev jest eslint

# Run with coverage
npx jest --coverage --coverageReporters=text-summary

# Complexity report
npx eslint src/ --format json | npx eslint-find-new-rules || true
npx complexity --threshold 10 src/

# SonarQube-style metrics (if using CI)
npx sonarqube-scanner
```

### ESLint Rules for Complexity
```json
{
  "max-depth": [2, 4],
  "max-lines-per-function": [2, 50],
  "max-complexity": [2, 10],
  "no-param-reassign": 2
}
```

## Python

```bash
# Install radon for complexity + maintainability
pip install radon

# Cyclomatic complexity
radon cc -a -s src/

# Raw SLOC
radon raw -s src/

# Maintainability index
radon mi -s src/

# Coverage
pip install coverage
coverage run -m pytest
coverage report --show-missing
```

### Example Output
```
$ radon cc -a -s src/
src/utils.py ... (M=8)
src/models.py ... (M=12)
src/api.py ... (M=15)

$ radon mi -s src/
src/utils.py ... 85
src/models.py ... 72
src/api.py ... 68
```

## Go

```bash
# Install gocyclo
go install github.com/fzipp/gocyclo/cmd/gocyclo@latest

# Complexity report
gocyclo -avg -top 20 ./...

# Go style check
go install golang.org/x/tools/go/cmd/goimports@latest
go vet ./...
```

## Output Format

```markdown
## Code Metrics Report

**Project:** `{project_name}`
**Branch:** `{branch}` | **Date:** `{YYYY-MM-DD}`

### Summary

| Metric | Value | Status |
|--------|-------|--------|
| Total Files | 142 | — |
| Lines of Code | 12,840 | — |
| Test Coverage | 74% | 🟡 |
| Avg Complexity | 6.2 | ✅ |
| Max Complexity | 18 | 🟡 |
| Maintainability | 76/100 | 🟡 |

### Complexity Distribution
```
1-5:   ████████████████████ 65%
6-10:  ████████ 25%
11-20: ██ 7%
20+:   █ 3%
```

### Files Needing Attention
| File | Complexity | Coverage | Maintainability |
|------|------------|----------|-----------------|
| `src/api/routes.ts` | 18 | 45% | 58 |
| `src/utils/parser.py` | 14 | 62% | 65 |
| `src/core/engine.go` | 12 | 71% | 70 |

### Quality Gate Status
| Gate | Threshold | Actual | Pass? |
|------|-----------|--------|-------|
| Coverage | ≥80% | 74% | ❌ |
| Max Complexity | ≤15 | 18 | ❌ |
| Avg Complexity | ≤10 | 6.2 | ✅ |
| Maintainability | ≥70 | 76 | ✅ |

### Recommendations
- Extract `src/api/routes.ts` into smaller functions (18 → aim for <10)
- Increase test coverage on `src/utils/parser.py` (+18% to reach 80%)
- Consider extracting shared logic from duplicated blocks
```

## CI/CD Integration

### GitHub Actions Quality Gate
```yaml
- name: Code Metrics
  run: |
    npx jest --coverage --coverageReporters=lcov
    npx sonar-scanner
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

- name: Quality Gate
  run: |
    COVERAGE=$(npx jest --coverage --silent 2>/dev/null | grep coverage | awk '{print $2}' | tr -d '%')
    if [ "$COVERAGE" -lt 80 ]; then echo "Coverage $COVERAGE% < 80%"; exit 1; fi
```

### Track Over Time
Save metrics to a JSON file each run, commit to a `metrics/` branch or use a tracking sheet.

## Quick One-Liners

```bash
# Count lines per language
find . -name "*.ts" -o -name "*.js" | xargs wc -l | tail -1

# Find most complex functions (ESLint)
npx eslint src/ --format=json | jq '[.[] | .messages[]] | map(select(.severity == 2)) | length'

# Python: show complexity by function
radon cc -a -s src/ | grep -E "F|\-"

# Go: list complex functions
gocyclo -top 10 ./...
```

## Pro Tips

- **Track metrics over time** — a rising complexity trend is more alarming than a single reading
- **Complexity spikes in PRs** — use pre-commit hooks to block high-complexity additions
- **Coverage isn't everything** — 80% coverage of shallow tests < 60% coverage of meaningful tests
- **Context matters** — generated code, test fixtures, and migration scripts skew metrics
