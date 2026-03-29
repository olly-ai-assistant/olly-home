---
name: dead-code-detector
description: Detects unused code, dead code paths, and unreachable functions across a codebase. Use when asked to "find dead code", "detect unused code", "find unreachable functions", "analyze code coverage gaps", or when doing code cleanup, refactoring, or tech debt assessment. Helps identify code that can be safely removed without affecting functionality.
---

# Dead Code Detector

This skill detects unused code, dead code paths, and unreachable functions across a codebase, helping reduce tech debt and improve maintainability.

## When to Use This Skill

Use this skill when:
- Asked to "find dead code", "detect unused code", or "analyze code for removal"
- Performing code cleanup or refactoring
- Assessing tech debt in a codebase
- Looking for code coverage gaps
- Preparing for a major refactor or migration
- Conducting a code review and wanting to identify removable code

## Detection Strategies

### 1. Static Analysis

**For JavaScript/TypeScript:**
```bash
# Using eslint with no-unused-vars
npx eslint . --no-eslintrc --env es6 --env node --rule '{"no-unused-vars":"warn"}' 2>&1 | grep "unused"

# Using ts-prune (for TypeScript)
npx ts-prune

# Using madge (for dependency analysis)
npx madge --circular --extensions ts,js
```

**For Python:**
```bash
# Using vulture
npx vulture . --min-confidence 80

# Using autoflake
autoflake --remove-unused-variables --recursive .
```

**For Go:**
```bash
# Using staticcheck
staticcheck ./...
```

**For Multiple Languages:**
```bash
# Using decipherdetect
npx decipherdetect analyze <path>
```

### 2. Pattern-Based Detection

Look for common dead code patterns:

| Pattern | Description | Detection Method |
|---------|-------------|------------------|
| Unused exports | Functions/classes exported but never imported | AST analysis or grep for usage |
| Commented-out code | Code blocks wrapped in `/* ... */` or `//` | Regex: `/\/\*[\s\S]*?\*\/\|^\s*\/\/.+$/m` |
| Dead code paths | `if (false)`, `else` blocks after return | Control flow analysis |
| Unreachable code | After throw/return/break/continue | AST traversal |
| Empty destructors | Destructors with no logic | Pattern match |
| TODO/FIXME without context | Comments indicating abandoned work | Grep for patterns |

### 3. Coverage-Based Detection

Cross-reference runtime coverage reports:

```bash
# Jest coverage
npx jest --coverage --coverageReporters=json
# Analyze coverage JSON for 0% functions

# Istanbul/NYC
npx nyc --report=json npm test
# Look for functions with 0 executions
```

### 4. Git History Analysis

Find code that hasn't been modified in a long time:

```bash
# Files not modified in 1+ years
git log --all --full-history --diff-filter=U --name-only --pretty=format: | sort -u

# Code age analysis
git log --format=format: --name-only -S"function_name" -- .
```

## Output Format

Present findings in a structured report:

```markdown
## Dead Code Report

### Summary
- **Total findings:** N
- **By severity:** Critical (N), Warning (N), Info (N)

### Unused Exports
| File | Export | Last Used |
|------|--------|-----------|
| src/utils.js | `formatDate` | Never imported |
| src/api.js | `legacyFn` | 2 years ago |

### Unreachable Code
| File | Line | Description |
|------|------|-------------|
| src/main.js | 45-52 | Empty else block |
| src/app.js | 120 | Code after return |

### Commented-Out Code
| File | Lines | Preview |
|------|-------|---------|
| src/old.js | 10-25 | `// old implementation` |

### Recommendation
Priority removal list:
1. `src/utils.js:formatDate` - Never used, remove safely
2. `src/old.js:10-25` - Commented out since 2024, remove
```

## Safe Removal Process

1. **Verify no usage**: Search entire codebase for references
2. **Check tests**: Ensure no tests depend on the code
3. **Check dynamic usage**: Look for `require()`, `eval()`, `Reflect.metadata()`
4. **Create backup branch**: `git checkout -b cleanup/remove-dead-code`
5. **Remove incrementally**: Remove one piece at a time
6. **Run tests**: Verify nothing breaks
7. **Document removals**: Note what was removed and why

## Exclusions

Do NOT flag as dead code:
- Code used via reflection or dynamic patterns
- Code in `test/` or `__tests__/` directories
- Example code in documentation
- Code guarded by feature flags
- Polyfills and browser compatibility code
- Code with intentionally low coverage (documented)

## Tools & Resources

- **ESLint:** `npx eslint --rule 'no-unused-vars: warn'`
- **ts-prune:** `npx ts-prune` (TypeScript)
- **vulture:** Python dead code detection
- **madge:** `npx madge --circular` (dependency analysis)
- **decipherdetect:** `npx decipherdetect` (multi-language)
- **gdead:** Go dead code detector
