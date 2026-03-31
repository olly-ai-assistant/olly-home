---
name: endpoint-inventory
description: Auto-discovers and documents API endpoints from a codebase by scanning route files, controllers, and API definitions. Produces a structured inventory (method, path, handler file, description) for use by testers, reviewers, and documentation agents.
---

# Endpoint Inventory

This skill auto-discovers and documents API endpoints from a codebase by scanning route files, controllers, and API definitions. Produces a structured inventory useful for testers, reviewers, and documentation agents.

## When to Use This Skill

Use this skill when:
- Asked to "inventory endpoints", "map API routes", or "find all API endpoints"
- Need to audit an API for testing coverage
- Preparing documentation for an API
- Reviewing an API for security or design issues
- Onboarding to a new codebase and need to understand the API surface

## Supported Frameworks

### Express.js / Node.js
```bash
# Scan for Express routes
grep -rn "router\.\(get\|post\|put\|patch\|delete\|head\|options\)" --include="*.js" --include="*.ts" .
grep -rn "app\.\(get\|post\|put\|patch\|delete\|head\|options\)" --include="*.js" --include="*.ts" .

# Find route files
find . -name "*route*.js" -o -name "*route*.ts" -o -name "*router*.js" -o -name "*router*.ts"
```

### FastAPI / Python
```bash
# Scan for FastAPI routes
grep -rn "@app\.\(get\|post\|put\|patch\|delete\|head\|options\)" --include="*.py"
grep -rn "@router\.\(get\|post\|put\|patch\|delete\|head\|options\)" --include="*.py"

# Find router files
find . -name "*router*.py" -o -name "*route*.py"
```

### Flask / Python
```bash
# Scan for Flask routes
grep -rn "@app\.\(route\|get\|post\|put\|patch\|delete\)" --include="*.py"
grep -rn "@bp\.\(route\|get\|post\|put\|patch\|delete\)" --include="*.py"
```

### Django
```bash
# Scan Django views
grep -rn "path\|re_path" --include="*urls.py"
grep -rn "@method_decorator" --include="*views.py"

# Find URLconfs
find . -name "urls.py"
```

### Ruby on Rails
```bash
# Scan routes
grep -rn "resources\|resource\|get\|post\|put\|patch\|delete" config/routes.rb

# List all routes
rails routes
```

### Next.js / App Router
```bash
# Find page/route files
find . -path ./node_modules -prune -o \( -name "page.ts" -o -name "page.tsx" -o -name "route.ts" -o -name "route.tsx" \) -print

# API routes
find . -path ./node_modules -prune -o -path "*/api/*" \( -name "route.ts" -o -name "route.tsx" \) -print
```

### Go (net/http, Gin, Echo)
```bash
# Gin/Echo patterns
grep -rn "router\.\(GET\|POST\|PUT\|PATCH\|DELETE\|OPTIONS\)" --include="*.go"

# net/http patterns
grep -rn "http\.\(HandleFunc\|Handle\)" --include="*.go"
```

### Spring Boot (Java)
```bash
# Scan for request mappings
grep -rn "@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping\|@RequestMapping" --include="*.java"

# Find controllers
find . -name "*Controller.java"
```

### API Blueprints / OpenAPI
```bash
# Find spec files
find . \( -name "*.yaml" -o -name "*.yml" -o -name "*.json" \) | xargs grep -l "paths:" 2>/dev/null
```

## Output Format

### Structured Inventory Table

```markdown
## API Endpoint Inventory

**Total endpoints:** 24
**Base URL:** https://api.example.com/v1
**Generated:** 2026-03-31

| Method | Path | Handler | Description | Auth |
|--------|------|---------|-------------|------|
| GET | /users | controllers/users.js::getUsers | List all users | JWT |
| POST | /users | controllers/users.js::createUser | Create new user | JWT |
| GET | /users/:id | controllers/users.js::getUser | Get user by ID | JWT |
| PUT | /users/:id | controllers/users.js::updateUser | Update user | JWT |
| DELETE | /users/:id | controllers/users.js::deleteUser | Delete user | JWT |
| GET | /health | controllers/health.js::health | Health check | None |
```

### JSON Output

```json
{
  "generated": "2026-03-31T12:00:00Z",
  "baseUrl": "https://api.example.com/v1",
  "endpoints": [
    {
      "method": "GET",
      "path": "/users",
      "handler": "controllers/users.js::getUsers",
      "description": "List all users",
      "auth": "JWT",
      "params": [],
      "query": [],
      "body": null
    }
  ]
}
```

## Discovery Commands by Framework

### Express.js
```bash
#!/bin/bash
# endpoint-inventory-express.sh
echo "## Express.js Endpoint Inventory"
echo ""
echo "| Method | Path | Handler |"
echo "|--------|------|--------|"
grep -rn "router\.\(get\|post\|put\|patch\|delete\|head\|options\)" --include="*.js" --include="*.ts" . | \
  sed -E 's|^(.+):([0-9]+):(.+router)\.\((get|post|put|patch|delete|head|options)\)\(['"'"'"]?([^'"'"')\(]+)['"'"']?\).*|\4 | \5 | \3 |'
```

### FastAPI
```bash
#!/bin/bash
# endpoint-inventory-fastapi.sh
echo "## FastAPI Endpoint Inventory"
echo ""
echo "| Method | Path | Function |"
echo "|--------|------|----------|"
grep -rn "@\(app\|router\)\.\(get\|post\|put\|patch\|delete\|head\|options\)" --include="*.py" . | \
  sed -E 's|^(.+):([0-9]+):.*@\w+\.\((get|post|put|patch|delete|head|options)\)\(['"'"'"]?([^'"'"')\(]+)['"'"']?\).*|\3 | \5 | \1 |'
```

## Endpoint Documentation Extraction

### Extract JSDoc/TSDoc comments
```bash
grep -B5 "router\.\(get\|post" --include="*.js" --include="*.ts" . | grep -E "^\s*\*\s|^\s*/\*\*"
```

### Extract Python docstrings
```bash
grep -B3 "@app\.\(get\|post" --include="*.py" . | grep -E '^\s+"""'
```

## Usage Examples

### Basic inventory
```bash
# Run the appropriate scanner for your framework
./scripts/endpoint-inventory-express.sh > endpoint-inventory.md
```

### Full inventory with auth detection
```bash
# Combine with auth middleware detection
grep -rn "auth\|middleware\|verify" --include="*.js" controllers/ | head -20
```

### Generate OpenAPI from inventory
```bash
# Convert inventory to OpenAPI format
cat endpoint-inventory.json | jq '.endpoints[] | {path: .path, method: .method}'
```

## Tips

- Run from project root for absolute paths
- Use `--include` flags to restrict to relevant file types
- Combine with `wc -l` to count endpoints per controller
- Use `sort -u` to deduplicate paths
- Check for dynamic segments (`:id`, `{id}`, `<id>`) and document them

## Integration with Other Skills

- **tester**: Use inventory to generate test cases for each endpoint
- **docs**: Use inventory to build API documentation
- **reviewer**: Use inventory to audit endpoint design and security
- **api-contract-test**: Use inventory to verify implementation matches spec
