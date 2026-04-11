name: config-validator
description: Validates configuration files (JSON, YAML, TOML, ENV) for correctness, schema compliance, and security issues. Use when asked to "validate config", "check configuration", "lint config files", "verify settings", or when deploying, testing, or auditing a project's configuration. Catches missing required fields, type errors, invalid values, and security misconfigurations.
---

# Config Validator

Validates configuration files for correctness, schema compliance, and security issues.

## When to Use This Skill

- Deploying or testing a new configuration
- Auditing a project's config for errors or security issues
- Onboarding to a new project and need to understand its config
- Pre-commit or CI quality gate checks
- Troubleshooting startup failures related to config

## Supported Formats

| Format | Extensions | Schema Support |
|--------|-----------|----------------|
| JSON | `.json`, `config.json` | JSON Schema (draft-07+) |
| YAML | `.yaml`, `.yml` | JSON Schema via conversion |
| TOML | `.toml` | DTD (.toml spec) |
| ENV | `.env`, `.env.*` | dotenv-linter |
| INI | `.ini`, `.cfg` | Basic key-value |

## Quick Validation

```bash
# Detect format automatically
CONFIG_FILE="config/production.json"
if [ -f "$CONFIG_FILE" ]; then
  case "$CONFIG_FILE" in
    *.json) cat "$CONFIG_FILE" | python3 -m json.tool > /dev/null && echo "✅ JSON valid" ;;
    *.yaml|*.yml) python3 -c "import yaml; yaml.safe_load(open('$CONFIG_FILE'))" && echo "✅ YAML valid" ;;
    *.toml) python3 -c "import toml; toml.load(open('$CONFIG_FILE'))" && echo "✅ TOML valid" ;;
  esac
else
  echo "❌ Config file not found: $CONFIG_FILE"
fi
```

## Schema Validation

### JSON Schema (Python jsonschema)
```bash
pip install jsonschema

python3 << 'EOF'
import json, jsonschema

schema = json.load(open("schemas/config-schema.json"))
config = json.load(open("config/production.json"))

try:
    jsonschema.validate(config, schema)
    print("✅ Config is valid")
except jsonschema.ValidationError as e:
    print(f"❌ Validation error: {e.message}")
    print(f"   Path: {'.'.join(str(p) for p in e.path)}")
EOF
```

### YAML Validation
```bash
pip install pyyaml

python3 << 'EOF'
import yaml

with open("config/settings.yaml") as f:
    try:
        data = yaml.safe_load(f)
        print("✅ YAML parses correctly")
        print(f"   Keys: {list(data.keys())}")
    except yaml.YAMLError as e:
        print(f"❌ YAML error: {e}")
EOF
```

### TOML Validation
```bash
pip install toml

python3 -c "import toml; toml.load(open('config/app.toml')); print('✅ TOML valid')"
```

## Security Checks

Check for common security misconfigs:

```bash
# Find hardcoded secrets or tokens
grep -rE "(password|secret|api_key|token)\s*[:=]\s*['\"][^'\"]{8,}" --include="*.json" --include="*.yaml" --include="*.yml" .

# Check for overly permissive settings
grep -rE "(0\.0\.0\.0|:true|:false)" --include="*.json" --include="*.yaml" | grep -v example | grep -v "#"

# Detect missing required security fields
python3 << 'EOF'
import json

config = json.load(open("config/production.json"))

issues = []
# Example: require auth section
if "auth" not in config:
    issues.append("Missing 'auth' section")
# Example: check for debug in production
if config.get("debug") == True and config.get("environment") == "production":
    issues.append("debug=true in production environment")
# Example: require HTTPS in production
if config.get("environment") == "production" and not config.get("https", False):
    issues.append("HTTPS not enabled for production")

if issues:
    for issue in issues:
        print(f"⚠️  {issue}")
else:
    print("✅ No security issues found")
EOF
```

## ENV File Validation

```bash
pip install dotenv-linter

# Lint all .env files
dotenv-linter .env .env.production .env.local

# Check for required vars
python3 << 'EOF'
import os

required = ["DATABASE_URL", "SECRET_KEY", "API_TOKEN"]
optional = ["DEBUG", "LOG_LEVEL"]

missing = [v for v in required if v not in os.environ]
if missing:
    print(f"❌ Missing required vars: {', '.join(missing)}")
else:
    print("✅ All required env vars present")

present = [v for v in os.environ if v.startswith("APP_")]
print(f"📋 Custom vars defined: {', '.join(present)}")
EOF
```

## Output Format

```markdown
## Config Validation Report

**File:** `config/production.json`
**Status:** ❌ Invalid

### Errors
| Line | Field | Error |
|------|-------|-------|
| 24 | `database.port` | Expected integer, got string |
| 31 | `cache.ttl` | Value must be ≥ 0 |
| 45 | `auth.secret` | Missing required field |

### Warnings
| Line | Field | Warning |
|------|-------|---------|
| 12 | `server.host` | Using 0.0.0.0 exposes all interfaces |
| 38 | `debug` | Debug mode should be false in production |

### Security
| Severity | Issue |
|----------|-------|
| 🔴 Critical | Hardcoded API token found at line 67 |
| 🟡 Warning | Missing rate limiting configuration |

### Recommendations
1. Fix type error on `database.port` (line 24)
2. Remove hardcoded secrets — use environment variables
3. Enable HTTPS for production environment
```

## Common Config Issues

| Issue | How to Fix |
|-------|------------|
| Wrong type (string vs int) | Cast or quote appropriately |
| Missing required field | Add the field or use a default |
| Trailing commas in JSON | Remove trailing commas |
| Tab vs space indentation in YAML | Use spaces consistently (2 or 4) |
| Unquoted strings with special chars | Wrap in quotes |
| Duplicate keys | Remove duplicates (last wins) |
| Case-sensitive key mismatch | Use exact case as schema requires |

## Tools

- **jsonschema** (Python): `pip install jsonschema` — JSON Schema validation
- **pyyaml** (Python): `pip install pyyaml` — YAML parsing
- **toml** (Python): `pip install toml` — TOML parsing
- **dotenv-linter**: `pip install dotenv-linter` — .env file linting
- **configparser** (Python stdlib): INI file parsing
- **ajv** (Node): `npx ajv validate` — Fast JSON Schema validator
