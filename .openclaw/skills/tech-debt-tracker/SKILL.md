---
name: tech-debt-tracker
description: Tracks, categorizes, and reports on technical debt across a codebase. Use when asked to "track tech debt", "assess code quality", "identify technical debt", "tech debt review", or when planning refactoring work. Helps maintain a living inventory of known technical compromises, their impact, and remediation status.
---

# Tech Debt Tracker

This skill helps track, categorize, and report on technical debt across a codebase, enabling systematic prioritization and management of code quality issues.

## When to Use This Skill

Use this skill when:
- Asked to "track tech debt", "assess code quality", or "identify technical debt"
- Conducting a tech debt review or audit
- Planning refactoring or cleanup work
- Writing tech debt reports or documentation
- Prioritizing engineering work
- Onboarding to a new codebase and assessing quality
- Preparing for a major architectural change

## Tech Debt Categories

### Category Definitions

| Category | Description | Examples |
|----------|-------------|----------|
| **Legacy** | Old patterns/frameworks | jQuery, AngularJS, Backbone |
| **Complexity** | Overly complex code | Deeply nested conditionals, god functions |
| **Duplication** | Repeated code blocks | Copy-paste functions, duplicated logic |
| **Dependencies** | Outdated or vulnerable deps | Old package versions, known CVEs |
| **Documentation** | Missing docs or outdated docs | No JSDoc, stale README |
| **Testing** | Low test coverage | Untested core logic, missing edge cases |
| **Performance** | Known performance issues | N+1 queries, missing indexes |
| **Security** | Known security concerns | Hardcoded secrets, missing auth |
| **API Design** | Poor API contracts | Breaking changes, inconsistent interfaces |
| **Type Safety** | TypeScript/typing issues | `any` types, missing types |

## Detection Methods

### Automated Analysis

**Code Quality:**
```bash
# SonarQube (self-hosted)
sonar-scanner

# CodeClimate (cloud)
codeclimate analyze

# Better Code Hub
npx better-code-hub
```

**Dependency Analysis:**
```bash
# npm audit
npm audit --audit-level=moderate

# Snyk
npx snyk test

# Dependabot status
gh api graphql -f query='{ repository(owner:"owner", name:"repo") { vulnerabilityAlerts(first: 20) { nodes { packageName, severity, dependency, vulnerableVersionRange } } } }'
```

**Type Coverage:**
```bash
# TypeScript
npx tsc --noEmit --strict

# Python typing
mypy . --ignore-missing-imports
```

### Manual Detection

| Type | What to Look For |
|------|------------------|
| Commented warnings | `// HACK`, `// TODO: fix this`, `// FIXME` |
| Workaround patterns | Comments explaining why code exists |
| Ticket references | `// Ref: JIRA-1234` in old code |
| Test skips | `it.skip`, `@pytest.mark.skip`, `test.skip` |
| Dead code | Unused exports, unreachable paths |
| Circular deps | Import cycles detected by madge |

```bash
# Find tech debt markers
grep -rn "TODO\|FIXME\|HACK\|XXX\|NP\|BUGBUG" --include="*.js" --include="*.ts" --include="*.py" .

# Find skipped tests
grep -rn "skip\|pending" --include="*.test.*" --include="*.spec.*" -i .
```

## Tech Debt Inventory Format

Maintain a `TECH_DEBT.md` file in the project root:

```markdown
# Technical Debt Inventory

Last updated: YYYY-MM-DD
Total items: N | Critical: N | High: N | Medium: N | Low: N

## Critical (Address Immediately)
| ID | Category | Description | Location | Estimated Effort | Tracked By |
|----|----------|-------------|----------|------------------|------------|
| TD-001 | Security | Hardcoded API key | src/config.js:23 | 1h | JIRA-456 |

## High (Address Soon)
| ID | Category | Description | Location | Estimated Effort | Tracked By |
|----|----------|-------------|----------|------------------|------------|
| TD-002 | Dependencies | Outdated lodash v3 → v4 | package.json | 2h | JIRA-789 |

## Medium (Schedule)
| ID | Category | Description | Location | Estimated Effort | Tracked By |
|----|----------|-------------|----------|------------------|------------|
| TD-003 | Complexity | God function processOrder | src/order.js:45-120 | 4h | JIRA-321 |

## Low (When Possible)
| ID | Category | Description | Location | Estimated Effort | Tracked By |
|----|----------|-------------|----------|------------------|------------|
| TD-004 | Documentation | Missing JSDoc on exportFn | src/utils.js | 30m | - |
```

## Assessment Framework

### Impact Assessment

Rate each tech debt item:

| Score | Impact Level | Description |
|-------|--------------|-------------|
| 5 | Critical | Blocks feature development, security risk |
| 4 | High | Significant slowdown, frequent bugs |
| 3 | Medium | Moderate slowdown, occasional issues |
| 2 | Low | Minor inconvenience |
| 1 | Negligible | Cosmetic, low priority |

### Effort Assessment

| Score | Effort | Description |
|-------|--------|-------------|
| 5 | XL | 2+ weeks |
| 4 | L | 1-2 weeks |
| 3 | M | 3-7 days |
| 2 | S | 1-3 days |
| 1 | XS | <1 day |

### Priority Matrix

```
Effort →
Impact ↓    XS (1)  S (2)   M (3)   L (4)   XL (5)
Critical(5)   P2      P1      P1      P0      P0
High (4)      P3      P2      P1      P1      P0
Medium(3)     P4      P3      P2      P2      P1
Low (2)       P5      P4      P3      P3      P2
Neglig.(1)    P5      P5      P4      P4      P3
```

- **P0:** Immediately — blocks core functionality
- **P1:** This sprint — significant business impact
- **P2:** Next sprint — should be addressed
- **P3:** This quarter — planned improvement
- **P4:** Backlog — when capacity allows
- **P5:** Monitor — revisit periodically

## Reporting

### Quick Status Report

```markdown
## Tech Debt Summary

| Category | Count | Oldest | Most Impactful |
|----------|-------|--------|----------------|
| Dependencies | 5 | 8 months | Critical CVE |
| Testing | 3 | 6 months | Core flow untested |
| Documentation | 7 | 1 year | API docs stale |

### Age Distribution
- < 1 month: 3 items
- 1-6 months: 8 items
- 6-12 months: 4 items
- > 1 year: 2 items

### Trend
📈 Debt increasing: 3 new items this week
📉 Debt decreasing: 2 items resolved
```

### Executive Summary

```markdown
## Tech Debt Overview

**Total tracked:** 47 items (down from 52 last month)
**Estimated total effort:** 3 weeks
**Business impact:** ~$50k/year in developer productivity

**Top 3 priorities:**
1. Upgrade authentication library (P0 - security)
2. Fix N+1 query in orders endpoint (P1 - perf)
3. Document public API surface (P2 - onboarding)

**Progress this month:**
- Resolved: 5 items (11% reduction)
- New additions: 3 items
- Net change: -2 items
```

## Integration with Workflow

### During Code Review
- Flag tech debt when noticed
- Suggest JIRA ticket or add to TECH_DEBT.md
- Rate impact and effort

### During Planning
- Check TECH_DEBT.md for related items
- Include debt servicing in sprint estimates
- Balance feature work with debt reduction

### During Onboarding
- Review TECH_DEBT.md first
- Ask about highest-impact items
- Add observations as new items

## Tools

| Tool | Purpose | Language |
|------|---------|----------|
| sonar-scanner | Code quality analysis | Multi |
| snyk | Vulnerability scanning | Multi |
| npm audit | Dependency vulnerabilities | Node.js |
| mypy | Python type checking | Python |
| golangci-lint | Go linting & debt | Go |
| better-code-hub | Code quality scoring | Multi |
