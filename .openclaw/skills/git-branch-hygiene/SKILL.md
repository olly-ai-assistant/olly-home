---
name: git-branch-hygiene
description: Use when asked to clean up branches, merge strategies, stale branches, branch naming, or git workflow optimization. Triggers on "clean branches", "stale branches", "branch cleanup", "git workflow", "merge strategy", "branch hygiene".
---

# Git Branch Hygiene

Keep your repository clean and your team productive with systematic branch management.

## When to Use This Skill

- Cleaning up stale or merged branches
- Establishing branch naming conventions
- Planning merge or rebase strategies
- Finding branches that can be deleted
- Auditing branch protection rules
- Resolving merge conflicts

## Branch Lifecycle

```
create → develop → merge/delete → (optional) tag → archive
```

### Branch Types

| Type | Pattern | Lifetime |
|------|---------|----------|
| Feature | `feat/<description>` | Hours to weeks |
| Bugfix | `fix/<issue>` | Days to weeks |
| Hotfix | `hotfix/<description>` | Hours to days |
| Release | `release/<version>` | Weeks to months |
| Main | `main` (or `master`) | Forever |

## Naming Conventions

```
<type>/<ticket-id>-<short-description>
```

Examples:
- `feat/PROJ-123-user-authentication`
- `fix/PROJ-456-login-timeout`
- `hotfix/critical-security-patch`
- `refactor/api-client-cleanup`

### Anti-Patterns to Avoid

- `fix-bug` (too vague)
- `johns-fix` (personal branches)
- `temp` or `wip` (no cleanup culture)
- `latest` or `new-feature` (duplication chaos)

## Cleanup Workflow

### 1. Find Stale Branches

```bash
# Branches not merged to main and inactive for 30+ days
git fetch --prune
git branch -vv | grep ': gone]'

# Local branches already merged to main
git branch --merged main

# Remote branches already merged
git branch -r --merged main
```

### 2. Confirm Before Deleting

```bash
# Preview what will be deleted (local)
git branch --merged main | grep -v '^\*' | grep -v 'main'

# Confirm with remote
git push --dry-run <remote> --delete <branch-name>
```

### 3. Delete Safely

```bash
# Local branch
git branch -d <branch-name>

# Remote branch
git push <remote> --delete <branch-name>
```

### 4. Prune Remote References

```bash
git fetch --prune
git remote prune <remote>
```

## Merge vs Rebase Strategy

### Feature Branches: Rebase + Fast-Forward

```bash
git checkout feat/my-feature
git rebase main
git checkout main
git merge feat/my-feature  # Fast-forward
```

**Pros:** Clean linear history, no merge commits
**Cons:** Rewrites history (don't rebase shared branches)

### Release Branches: Merge (no rebase)

```bash
git checkout release/v1.2.0
git merge main          # Bring in latest main
git merge develop       # Merge release fixes
# ... test and release ...
git merge main         # Merge to main with merge commit
```

### Hotfix Branches: Merge + Tag

```bash
git checkout main
git checkout -b hotfix/critical-fix
# ... fix and test ...
git checkout main
git merge --no-ff hotfix/critical-fix
git tag -a v1.2.1 -m "Critical security patch"
git push origin main --tags
```

## Branch Protection

Protect important branches with these rules:

| Rule | Recommended For |
|------|-----------------|
| Require PR reviews | main, release/* |
| Require status checks | main, develop |
| Require signed commits | main |
| Restrict force pushes | main, release/* |
| Auto-delete merged | feature/*, fix/* |

Configure via GitHub/GitLab web UI or infrastructure-as-code.

## Health Metrics

Track branch hygiene health:

| Metric | Healthy | Warning |
|--------|---------|---------|
| Stale branches | 0 | >5 |
| Long-lived features | <2 weeks | >4 weeks |
| Merged but not deleted | 0 | >3 |
| Direct pushes to main | Never | Any |

## Automation Opportunities

- Auto-delete merged branches via CI (GitHub Actions, GitLab CI)
- Branch name validation in PR checks
- Stale branch reminders via scheduled cron
- Weekly cleanup digest to team

## Quick Commands Reference

```bash
# List all branches sorted by last commit
git for-each-ref --sort=-committerdate --format='%(refname:short)' refs/heads/

# Find branches with no upstream
git branch -vv | grep '^\s' | grep ': gone]'

# Delete all merged local branches
git branch --merged main | xargs -r git branch -d

# List branches by author (for ownership)
git for-each-ref --format='%(authorname) %(refname)' refs/heads/ | sort
```