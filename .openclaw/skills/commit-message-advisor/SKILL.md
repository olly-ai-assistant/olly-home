---
name: commit-message-advisor
description: This skill should be used when writing, reviewing, or improving git commit messages. Triggers on "write a commit message", "conventional commits", "improve commit message", "commit message format", or when asked to help craft commits.
---

# Commit Message Advisor

This skill guides writing clear, consistent, and useful commit messages following Conventional Commits specification.

## When to Use This Skill

- Writing a new commit message
- Reviewing or improving an existing commit message
- Setting up commit message conventions for a project
- Explaining why commit messages matter

## Conventional Commits Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Types

| Type | When to Use |
|------|-------------|
| `feat` | New feature for the user |
| `fix` | Bug fix for the user |
| `docs` | Documentation only changes |
| `style` | Formatting, missing semicolons, etc. (no code change) |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance improvement |
| `test` | Adding or correcting tests |
| `chore` | Maintenance tasks, dependencies, build scripts |
| `ci` | CI configuration changes |
| `revert` | Reverting a previous commit |

### Rules

1. **Subject line max 72 characters** (hard limit at 100)
2. **Subject line: no capital letter, no period at end**
3. **Use imperative mood**: "add feature" not "added feature"
4. **Separate subject from body with blank line**
5. **Wrap body at 72 characters**
6. **Use body to explain WHAT and WHY, not HOW**

## Commit Message Analysis Checklist

Before finalizing a commit message, verify:

- [ ] Type is appropriate for the change
- [ ] Subject is ≤72 characters
- [ ] Subject starts with lowercase
- [ ] Subject uses imperative mood
- [ ] Subject doesn't end with a period
- [ ] Subject summarizes the change accurately
- [ ] Body explains the motivation (not implementation)
- [ ] Breaking changes are marked with `BREAKING CHANGE:` footer
- [ ] References issues/tickets in footer when applicable

## Generating a Commit Message

When asked to generate a commit message:

1. **Analyze the changes**: Read the diff to understand what changed
2. **Determine the type**: Match the change to the most appropriate type
3. **Write the subject**: Concise summary of the change in ≤72 chars
4. **Add context if needed**: Explain the "why" in the body

### Example Workflow

```
# Analyze: User added a login throttle to prevent brute-force attacks
# Type: feat (new security feature)
# Subject: add login rate limiting
# Body: Implement per-IP rate limiting on /login endpoint
#       Limits to 5 attempts per minute per IP address.
#       Returns 429 Too Many Requests when exceeded.
```

## Project-Specific Conventions

Check for project-specific commit message rules in:
- `.github/COMMIT_CONVENTIONS.md`
- `CONTRIBUTING.md`
- `.czrc` or `commitlint.config.js`

If none exist, use Conventional Commits as the default.

## Quick Reference

```
feat: add user authentication
fix: prevent session hijacking on expired tokens
docs: update API documentation for v2 endpoints
refactor: extract payment processing into separate service
fix: handle null pointer in user profile lookup
```

## Anti-Patterns

| Bad | Better |
|-----|--------|
| `fixed stuff` | `fix: handle null user in dashboard` |
| `Updated file.js` | `refactor: simplify user validation logic` |
| `asdf` | `fix: correct date parsing for timezone offset` |
| `WIP` | `feat: add webhook retry mechanism (wip)` |
| `fix bug` | `fix: prevent double-charging on payment retry` |
