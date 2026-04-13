---
name: api-breaking-change-detector
description: Detects breaking changes between two API specifications (OpenAPI/Swagger, GraphQL schemas, protobuf). Use when asked to "compare API versions", "detect breaking changes", "check API compatibility", "validate API changes", or when doing API versioning reviews, CI/CD compatibility checks, or contract testing.
---

# API Breaking Change Detector

Detect breaking changes between two versions of an API specification. Essential for API versioning, CI/CD pipelines, and contract compatibility.

## When to Use This Skill

- Comparing two OpenAPI/GraphQL/protobuf versions
- Pre-release API compatibility validation
- CI/CD gate: block breaking changes to production APIs
- API migration planning (v1 → v2)
- Contract testing in service mesh/microservice architectures
- Reviewing PRs that modify API specs

## Supported Formats

| Format | Extensions | Tools |
|--------|-----------|-------|
| OpenAPI/Swagger | `.yaml`, `.yml`, `.json` | `swagger-diff`, `openapi-diff`, `spectral` |
| GraphQL | `.graphql`, `.gql`, schema files | `@graphql Inspector` |
| Protobuf | `.proto` | `buf`, `protolint` |
| AsyncAPI | `.yaml`, `.yml` | `asyncapi-diff` |

## Core Commands

### 1. OpenAPI / Swagger Diff

```bash
# Using openapi-diff (most comprehensive)
npx openapi-diff old spec.yaml new spec.yaml

# Using redocly (cleaner output)
npx @redocly/openapi-diff spec-v1.yaml spec-v2.yaml

# Using spectral for linting
npx spectral diff spec-v1.yaml spec-v2.yaml

# Docker alternative
docker run --rm -v $(pwd):/spec openapitools/openapi-diff spec-v1.yaml spec-v2.yaml
```

### 2. GraphQL Schema Diff

```bash
# Using graphql-inspector
npx @graphql-inspector/diff schema-v1.graphql schema-v2.graphql

# Using Apollo CLI
npx apollo schema:diff --graph=my-graph --variant=v1 --endpoint=https://api.example.com/graphql

# Using graphql-comare
npx graphql-codegen-compare schema-v1.graphql schema-v2.graphql
```

### 3. Protobuf Diff

```bash
# Using buf (recommended)
buf breaking old/file.proto --against new/file.proto

# protolint
protolint lint file.proto

# Manual proto diff
git diff --no-index old.proto new.proto
```

### 4. JSON Schema Comparison

```bash
# Using json-schema-compare
npx json-schema-compare schema-v1.json schema-v2.json

# Using ajv (validate compatibility)
ajv compile --spec=draft7 schema-v2.json
```

## Breaking Change Types

### 🔴 Breaking (Always Block)

| Change | Description | Detection |
|--------|-------------|-----------|
| Removed endpoint | DELETE `/users` | Endpoint missing in new |
| Removed field | DELETE `user.name` | Field missing in new schema |
| Required field added | ADD required param | New required param in new spec |
| Field type changed | `string` → `integer` | Type mismatch |
| Enum value removed | REMOVE `status=pending` | Enum value missing |
| Response code removed | DELETE `200 OK` | Status code missing |
| Parameter removed | DELETE `userId` param | Param missing |

### 🟡 Potentially Breaking (Review)

| Change | Description | Detection |
|--------|-------------|-----------|
| Field made required | `?name` → `name` | Required field added |
| Enum value added | ADD `status=archived` | Usually safe, check consumers |
| New required header | ADD `Authorization: required` | Check consumers |
| Response field added | ADD `user.avatar` | Safe (additive) |
| New endpoint | ADD `/v2/users` | Safe (additive) |
| Description change | Doc only | Safe |

### ✅ Safe (Non-Breaking)

| Change | Description |
|--------|-------------|
| New optional field | ADD `?sort` param |
| New response field | ADD `user.created_at` |
| New endpoint | ADD `/users/search` |
| New enum value | ADD `status=archived` |
| Deprecation added | Mark old field `@deprecated` |

## Output Format

```markdown
## API Breaking Change Report

**Old:** `openapi-v1.yaml` (2024-01-01)
**New:** `openapi-v2.yaml` (2024-03-15)
**Breaking:** 4 | **Potentially Breaking:** 2 | **Safe:** 8

### 🔴 Breaking Changes (BLOCK DEPLOY)
1. **Removed endpoint:** `DELETE /users/{id}` — was in v1, gone in v2
2. **Removed field:** `User.name` — removed from response schema
3. **Type changed:** `User.age` — `string` → `integer`
4. **Required param added:** `GET /search` — added required `query` param

### 🟡 Potentially Breaking (Review Required)
1. **Required field:** `User.email` — optional → required
2. **Enum value removed:** `Order.status=shipped` — removed

### ✅ Safe Changes
1. Added `GET /users/{id}/avatar` endpoint
2. Added `User.avatar_url` response field
3. Added optional `?include=metadata` query param
4. Added `User.created_at` response field

### Recommendations
1. 🔴 BLOCK: `DELETE /users/{id}` removal — document migration path
2. 🔴 BLOCK: `User.name` removal — backward incompatible
3. 🟡 Review: Add migration guide for `Order.status` enum change
4. ✅ Approve: All other changes are additive
```

## CI/CD Integration

### GitHub Actions Example
```yaml
- name: Check Breaking Changes
  run: |
    npx openapi-diff pr openapi.yaml ${{ github.event.pull_request.base.sha }}:openapi.yaml
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Pre-commit Hook
```bash
#!/bin/bash
# .git/hooks/pre-commit
openapi-diff old-spec.yaml new-spec.yaml --fail-on-breaking || exit 1
```

## Quick Reference

```bash
# OpenAPI breaking change check
npx openapi-diff old.yaml new.yaml

# GraphQL breaking + dangerous
npx @graphql-inspector/cli diff schema.graphql schema-new.graphql --breaking

# Buf breaking against main
buf breaking against https://github.com/org/repo.git#branch=main

# Spectral ruleset for breaking changes
npx spectral lint new.yaml --ruleset https://unpkg.com/@redocly/openapi-diff/ruleset.yaml
```
