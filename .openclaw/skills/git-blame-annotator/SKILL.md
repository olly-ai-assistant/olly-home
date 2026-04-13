---
name: git-blame-annotator
description: Analyzes git blame to map code ownership, identify original authors, find blame-heavy lines, and understand responsibility distribution. Use when asked to "git blame", "who wrote this code", "find code ownership", "attribute code to authors", or when resolving disputes about who wrote what, planning refactors, or doing code review accountability.
---

# Git Blame Annotator

Use git blame data to understand code authorship, responsibility, and ownership patterns.

## When to Use This Skill

- Asked "who wrote this code" or "git blame for a file"
- Mapping code ownership across a codebase
- Finding who is responsible for specific code sections
- Resolving questions about code origins during reviews
- Identifying single points of knowledge (bus factor)
- Finding recently changed code vs. ancient code
- Code review: understanding why a line exists

## Core Commands

### 1. Basic Blame

```bash
# Full blame for a file
git blame path/to/file

# Blame with email (more compact)
git blame --email <file>

# Blame with relative dates
git blame --relative-date <file>

# Blame specific lines
git blame -L 10,20 path/to/file

# Ignore whitespace changes
git blame -w path/to/file
```

### 2. Blame by Author

```bash
# Count lines per author
git blame --line-porcelain <file> | grep "^author " | sort | uniq -c | sort -rn

# Blame output with author emails
git blame --format="%aN (%ae) %s" <file>

# Show only authors, no line numbers
git blame --format="%aN" <file> | sort | uniq -c | sort -rn
```

### 3. Blame Over Time

```bash
# Lines by age (how old is each line)
git blame --format="%aN %ad" --date=short <file> | awk '{print $2}' | sort | uniq -c

# Find recently added/modified code
git blame --since="2024-01-01" -- <file>

# Code age distribution
git blame <file> | awk '{print $2, $3}' | cut -d')' -f1 | sort | uniq -c | sort -rn
```

### 4. Hotspot Blame

```bash
# Lines with most commits touching them (shared code = unstable or critical)
git blame <file> | cut -d' ' -f1 | sort | uniq -c | sort -rn | head -20

# Find lines untouched for years (stable code)
git blame <file> | awk '{print $2}' | grep -v "^[0-9]" | head

# Blame across a diff (what changed in last commit)
git diff HEAD~1 --name-only | xargs -I{} git blame --line-porcelain {} | grep "^author "
```

### 5. Responsibility Mapping

```bash
# Build author→file ownership map
for f in $(git ls-files); do
  author=$(git blame --format="%aN" -- "$f" 2>/dev/null | head -1)
  echo "$author | $f"
done | sort | uniq

# Files owned primarily by one author
git blame <file> | awk '{print $2}' | sed 's/^(//;s/)$//' | sort | uniq -c | sort -rn

# Bus factor: files touched by only 1-2 authors
git log --format=format:"%aN" --name-only | grep -v '^$' | sort -u | \
  awk '{print NR, $0}' | sort -k1 | uniq -c | awk '$1 <= 2 {print $3}'
```

## Output Format

```markdown
## Git Blame Report

**File:** `src/auth/login.ts`
**Date:** {YYYY-MM-DD}

### Line Ownership
| Lines | Author | Email | Last Touched |
|-------|--------|-------|--------------|
| 245 | Jane Doe | jane@example.com | 2024-03-15 |
| 89 | John Smith | john@example.com | 2023-11-02 |
| 34 | Alice Wu | alice@example.com | 2022-06-19 |

### Age Distribution
- **< 3 months:** 12 lines (5%)
- **3-12 months:** 67 lines (27%)
- **1-2 years:** 156 lines (63%)
- **> 2 years:** 12 lines (5%)

### Responsibility Hotspots
- 🔴 `src/auth/login.ts` — 67% owned by Jane Doe (single point of knowledge)
- 🟡 `src/auth/session.ts` — shared among 4 authors (stable interface)

### Recommendations
1. 🔴 Cross-train `src/auth/login.ts` — Jane is sole owner, bus factor risk
2. 🟡 Document `src/auth/session.ts` — stable despite multiple authors
3. ✅ Review 34-line block by Alice (unchanged since 2022, may be deprecated)
```

## Quick One-Liners

```bash
# Who wrote the most lines currently?
git blame --line-porcelain <file> | grep "^author " | sort | uniq -c | sort -rn

# What did a specific author last touch?
git blame --author="Name" <file>

# Lines unchanged in 2+ years (stable code)
git blame <file> | awk -F'[()]' '{print $2}' | xargs -I{} git log --format="%ad" --until="2 years ago" -- {} | head

# Find your own code
git blame --author="$(git config user.name)" <file>
```

## Tips

- Use `-w` flag to ignore whitespace-only changes (reduces noise)
- `-M` flag detects moved/copied lines within a file
- Combine with `git log --follow` for renamed file history
- For large files, use `--score-debug` to see similarity scores
