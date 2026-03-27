---
name: pr-description-generator
description: Automatically generate meaningful PR descriptions from git diffs, commit messages, and changelogs. Use when creating pull requests, writing changelogs, or documenting code changes.
---

# PR Description Generator

Generate clear, informative pull request descriptions from code changes.

## Usage

### Generate from git diff
```bash
git diff HEAD~1 --stat && git diff HEAD~1
```
Pass the output to this skill for description generation.

### Generate from commit messages
```bash
git log --oneline -10
```
Use the commit messages to build a structured PR description.

## PR Description Template

```markdown
## Summary
Brief description of what this PR does and why.

## Changes
- **Added**: New features or additions
- **Changed**: Modifications to existing functionality
- **Fixed**: Bug fixes
- **Removed**: Deleted features or files

## Technical Details
- Key implementation decisions
- Architecture changes
- Migration notes (if applicable)

## Testing
- How was this tested?
- Any manual testing steps?
- Test coverage impact

## Screenshots / Evidence
(If UI changes)
```

## Quick Generation Steps

1. Run `git diff HEAD~N --stat` to see changed files
2. Run `git log HEAD~N..HEAD --oneline` for commit history
3. Identify the PR goal from the commits
4. Categorize each changed file into: Added/Changed/Fixed/Removed
5. Fill in the template above

## Categorization Rules

| File Pattern | Category | Example |
|-------------|----------|---------|
| `**/new/**` | Added | New component, new utility |
| `**/test/**` | Added | New test files |
| `package.json` deps | Changed | Added dependency |
| `*.config.*` | Changed | Build/tooling config |
| `**/fix/**` or `fix-*.js` | Fixed | Bug fix |
| `CHANGELOG.md` | Documentation | Version notes |
| `*.css` / `*.scss` | Changed | Styling |
| `*.md` | Documentation | Docs update |

## Conventional Commits

If commits follow [Conventional Commits](https://www.conventionalcommits.org/):
```bash
git log --format="%s" HEAD~5..HEAD
```
Parse `feat:`, `fix:`, `docs:`, `chore:`, `refactor:` prefixes to auto-categorize.

## Example Output

```markdown
## Summary
Refactor user authentication flow to support OAuth 2.0 and add session persistence.

## Changes
- **Added**: OAuth 2.0 Google/GitHub login support
- **Added**: Session token generation and refresh logic
- **Changed**: `auth.js` middleware to support multiple providers
- **Changed**: Updated `package.json` with `passport` and `express-session`
- **Fixed**: Race condition in token refresh (closes #42)

## Technical Details
- Uses Passport.js for OAuth abstraction
- Sessions stored in Redis for horizontal scaling
- JWT tokens with 24h expiry and refresh tokens

## Testing
- Added unit tests for token generation (95% coverage)
- Manual OAuth flow tested with Google sandbox account
```

## Changelog Integration

If your project uses auto-changelogs:
```bash
# Generate changelog section
npx auto-changelog --output CHANGELOG.md --stdout
```
Use the output to populate the "Changes" section.
