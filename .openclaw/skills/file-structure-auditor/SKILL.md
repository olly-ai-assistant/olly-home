---
name: file-structure-auditor
description: Analyze and audit project directory structures for organization, convention compliance, and maintainability. Use when reviewing codebases, onboarding to a project, or assessing project health.
---

# File Structure Auditor

Audit and analyze project file structures for organization, convention adherence, and health.

## Usage

### Quick Audit
```bash
# List all files (exclude node_modules, .git, etc.)
find . -type f | grep -v -E 'node_modules|\.git|dist|build|\.next' | head -100

# Directory tree
find . -type d | head -50

# File count by extension
find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -20
```

### Deep Structure Analysis
```bash
# Lines of code per directory (JS/TS)
find . -name "*.js" -o -name "*.ts" -o -name "*.tsx" | xargs wc -l 2>/dev/null | sort -rn | head -20

# Detect oversized files (>500 lines = investigate)
find . -name "*.js" -exec wc -l {} \; 2>/dev/null | awk '$1 > 500 {print}'

# Detect deep nesting (path depth > 5 = consider flattening)
find . -type f | awk -F/ '{print NF-1, $0}' | sort -rn | head -10
```

## Health Checks

### ✅ Good Structure Indicators
- Clear separation of concerns (`/src`, `/lib`, `/tests`)
- Consistent naming conventions (kebab-case, camelCase, PascalCase)
- Flat directories (max depth 4-5)
- Small focused files (<300 lines)
- Test files co-located or in `/__tests__/`

### ⚠️ Warning Signs
- Files > 500 lines (needs splitting)
- Directory depth > 6 (complexity smell)
- Too many files in root (> 20 files = chaos)
- Mixed naming conventions
- Orphan files (untested, undocumented)
- Circular dependencies (`a → b → c → a`)

### ❌ Problem Indicators
- God files (> 1000 lines)
- Circular requires/imports
- Duplicate functionality across files
- Dead code (unused exports)
- Configuration in wrong places

## Common Patterns by Project Type

### Node.js / JavaScript
```
project/
├── src/           # Source code
├── tests/         # Test files
├── scripts/       # Build/utility scripts
├── package.json
├── README.md
```

### Python
```
project/
├── src/ or project_name/
├── tests/
├── scripts/
├── setup.py / pyproject.toml
├── README.md
```

### Next.js / React
```
project/
├── app/           # App router pages
├── components/    # Reusable components
├── lib/           # Utilities
├── hooks/         # Custom hooks
├── public/
├── package.json
```

### Monorepo
```
project/
├── packages/
│   ├── package-a/
│   └── package-b/
├── apps/
│   ├── web/
│   └── docs/
├── shared/
├── package.json   # Workspace root
```

## Audit Checklist

### Organization
- [ ] Logical grouping of related files
- [ ] Clear entry points (`index.js`, `main.py`)
- [ ] No root-level clutter

### Naming Conventions
- [ ] Consistent file naming (kebab vs camel vs Pascal)
- [ ] Descriptive names (avoid `util.js`, `helper.js`)
- [ ] Feature-named directories

### Separation of Concerns
- [ ] Business logic separate from framework code
- [ ] Configuration separate from application code
- [ ] Tests separate from source (or properly co-located)

### Maintainability
- [ ] Small, focused files
- [ ] Shallow directory depth
- [ ] Clear import/require structure

### Testing
- [ ] Test files present
- [ ] Tests near source or in dedicated test dir

## Output Format

When auditing, report:

```
## Structure Overview
Total files: X | Total dirs: Y | Max depth: Z

## Health Score: X/10
- [ ] Organization: X/5
- [ ] Naming: X/5
- [ ] Depth: X/5
- [ ] File size: X/5

## Issues Found
1. **[LEVEL]** file: Description and suggestion
2. **[LEVEL]** directory/: Description and suggestion

## Recommendations
1. ...
2. ...
```

## Quick Commands Reference

| Check | Command |
|-------|---------|
| File count | `find . -type f | wc -l` |
| By extension | `find . -type f | sed 's/.*\.//' | sort | uniq -c` |
| Deep nesting | `find . -type f -printf '%d %p\n' | sort -rn | head` |
| Large files | `find . -type f -exec wc -l {} \; | awk '$1>500'` |
| Empty dirs | `find . -type d -empty` |
| Newer than 30d | `find . -type f -mtime -30` |
