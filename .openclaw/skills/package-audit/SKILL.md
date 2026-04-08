---
name: package-audit
description: Audits npm/yarn/pip packages for vulnerabilities, outdated dependencies, license issues, and supply chain risks. Use when asked to "audit dependencies", "check for vulnerabilities", "find outdated packages", "scan npm audit", "check package security", or when doing dependency maintenance, security reviews, or CI/CD pipeline setup.
---

# Package Audit

This skill audits package dependencies across npm, yarn, pip, and other package managers for security vulnerabilities, outdated versions, license compliance issues, and supply chain risks.

## When to Use This Skill

Use this skill when:
- Asked to "audit dependencies", "check for vulnerabilities", or "scan package security"
- Performing routine dependency maintenance
- Setting up or reviewing CI/CD pipelines
- Conducting security reviews
- Preparing a release or deployment
- Responding to a vulnerability disclosure
- Checking license compliance for a project

## Package Manager Commands

### npm

```bash
# Basic audit for vulnerabilities
npm audit

# Audit with JSON output for parsing
npm audit --json

# Audit production dependencies only
npm audit --production

# Audit and automatically fix some issues
npm audit fix

# List outdated packages
npm outdated

# View available updates
npm update

# Check for security advisories
npm audit adv || true
```

### yarn

```bash
# Audit for vulnerabilities
yarn audit

# List outdated packages
yarn outdated

# Upgrade dependencies
yarn upgrade

# Interactive upgrade
yarn upgrade-interactive
```

### pip (Python)

```bash
# Check for vulnerabilities (using safety)
pip install safety 2>/dev/null
safety check

# Or using pip-audit
pip install pip-audit 2>/dev/null
pip-audit

# List outdated packages
pip list --outdated

# Check dependencies
pip freeze > requirements.txt
pip install -r requirements.txt --dry-run
```

### pnpm

```bash
# Audit
pnpm audit

# Check outdated
pnpm outdated
```

## Vulnerability Databases

| Database | URL | Coverage |
|----------|-----|----------|
| npm Advisory Database | `npmjs.com/advisories` | npm packages |
| Snyk Vulnerability Database | `security.snyk.io` | npm, pip, gem, etc. |
| OSV (Open Source Vulnerabilities) | `osv.dev` | Multiple ecosystems |
| GitHub Advisory Database | `github.com/advisories` | Multiple ecosystems |
| CVE Database | `cve.mitre.org` | General |

## License Compliance Checking

```bash
# Using license-checker (npm)
npx license-checker --json > licenses.json
# Review output for problematic licenses: GPL, AGPL, SSPL, etc.

# Using fossa or snyk for deeper analysis
npx snyk test --license
```

## Supply Chain Security

### Checking for Malicious Packages

```bash
# Check package download counts for anomalies
npm view <package> dist-tags.last-week

# Verify package integrity (lockfile)
npm ci  # Reproducible install
npm ls --depth=0  # List direct dependencies

# Check for typosquatting candidates
# Compare package name to known good: express vs expreess, express- vs @express/
```

### Package Signing Verification

```bash
# npm provenance (if supported)
npm install --verbose 2>&1 | grep " proved "

# Verify package shasum
npm view <package> dist.tarball | xargs curl -sL | sha256sum
```

## Output Format

Present findings in a structured report:

```markdown
## Package Audit Report

### Summary
- **Total dependencies:** N
- **Vulnerabilities:** Critical (N), High (N), Medium (N), Low (N)
- **Outdated packages:** N
- **License issues:** N

### Critical/High Vulnerabilities
| Package | Version | Vulnerability | Fix Available | Severity |
|---------|---------|---------------|---------------|----------|
| lodash | 4.17.15 | CVE-2021-23337 | 4.17.21 | High |
| express | 4.17.1 | Prototype Pollution | 4.17.3 | Critical |

### Outdated Packages
| Package | Current | Wanted | Latest | Type |
|---------|---------|--------|--------|------|
| axios | 0.21.0 | 0.21.4 | 1.6.0 | dependency |

### License Concerns
| Package | License | Risk | Use |
|---------|---------|------|-----|
| some-package | GPL-3.0 | Copyleft | Core functionality |

### Recommendations
1. **Immediate:** Fix critical vulnerabilities (run `npm audit fix`)
2. **Short-term:** Update packages with breaking changes in minor versions
3. **Long-term:** Evaluate license compliance for production use
```

## CI/CD Integration

### GitHub Actions

```yaml
- name: Run npm audit
  run: npm audit --audit-level=high
  continue-on-error: true

- name: Check for outdated packages
  run: npm outdated --json || true
  id: outdated
```

### GitLab CI

```yaml
dependency_scanning:
  script:
    - npm audit --json > audit-report.json
    - pipe: kubectl get pods -o wide
```

## Remediation Priority

| Priority | Severity | Action |
|----------|----------|--------|
| P0 | Critical | Patch immediately or remove package |
| P1 | High | Patch within 24-48h |
| P2 | Medium | Patch within sprint |
| P3 | Low | Patch in next release cycle |
| P4 | Info | Review and address as time permits |

## Tools & Resources

- **npm audit:** Built-in to npm CLI
- **snyk:** `npx snyk test` (free tier available)
- **ossf/scorecard:** `npx scorecard` (security score)
- **license-checker:** `npx license-checker`
- **safety:** Python vulnerability scanner
- **pip-audit:** Python vulnerability scanner
- **dependabot:** Automated dependency updates
- **renovate:** Automated dependency updates
