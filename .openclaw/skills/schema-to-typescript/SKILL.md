name: schema-to-typescript
description: Converts JSON Schema, OpenAPI, GraphQL, or similar schema definitions to clean TypeScript interfaces/types. Use when asked to "generate TypeScript types from schema", "convert schema to interfaces", "create types from JSON Schema", "OpenAPI to TypeScript", or when working with API schemas, data models, or any structured schema that needs TypeScript typing. Essential for coders building API clients, docs writers documenting data models, testers writing typed test fixtures, and reviewers auditing type consistency.
---

# Schema to TypeScript

Converts JSON Schema, OpenAPI, GraphQL SDL, Zod schemas, or similar definitions into clean, accurate TypeScript interfaces and types.

## When to Use This Skill

Use this skill when:
- Asked to "generate TypeScript from JSON Schema" or "convert schema to types"
- Building API clients and need type-safe request/response types
- Writing documentation and need TypeScript examples for data models
- Creating typed test fixtures from schema definitions
- Auditing type consistency between schema and implementation
- Converting OpenAPI/Swagger specs to TypeScript
- Working with GraphQL schemas and needing TypeScript types

## Supported Schema Formats

| Format | Input File Patterns | Key Tools |
|--------|---------------------|-----------|
| JSON Schema | `*.schema.json`, `schema/*.json` | `@apidevtools/json-schema-ref-parser` |
| OpenAPI 3.x | `openapi.yaml`, `api.yaml`, `swagger.json` | `openapi-typescript` |
| GraphQL SDL | `schema.graphql`, `*.gql` | `@graphql-codegen/cli` |
| Zod | TypeScript files with `z.object()` | Manual conversion |
| Protocol Buffers | `*.proto` | `protoc-gen-grpc-web` |

## Conversion Process

### Step 1: Identify Schema Type

```
# Detect JSON Schema
jq '. "$schema"' schema.json  # should contain json-schema.org

# Detect OpenAPI
grep -q '"openapi"' schema.json && echo "OpenAPI detected"

# Detect GraphQL
grep -q 'type Query\|type Mutation' schema.graphql && echo "GraphQL SDL detected"
```

### Step 2: Convert Based on Format

**For OpenAPI/Swagger (recommended):**
```bash
# Install openapi-typescript
npx openapi-typescript https://api.example.com/openapi.yaml --output types.ts

# From local file
npx openapi-typescript ./api/openapi.yaml --output types.ts

# With additional options
npx openapi-typescript ./openapi.yaml \
  --output types.ts \
  --default和非可空类型 \
  --postfix-components "" \
  --version openapi3
```

**For JSON Schema (manual or tool-assisted):**
```bash
# quicktype - excellent for JSON Schema to TypeScript
npx @quicktype/quicktype --schema schema.json --src-lang schema --lang typescript --out types.ts

# Or use json-schema-to-typescript
npx json-schema-to-typescript schema.json > types.ts
```

**For GraphQL:**
```bash
# Using GraphQL Code Generator
npx graphql-codegen init
# Configure codegen.yml:
# schema: schema.graphql
# generates:
#   types.ts:
#     plugins:
#       - typescript

npx graphql-codegen
```

### Step 3: Manual Refinement

After automated conversion, refine for:

- **Optional vs required fields**: Ensure `required` array from JSON Schema maps to non-optional properties
- **Null handling**: Decide between `field?: Type | null` vs `field: Type | null`
- **Discriminated unions**: Convert `oneOf`/`anyOf` to TypeScript union types
- **Naming conventions**: Apply camelCase consistently
- **Documentation**: Preserve descriptions as JSDoc comments

## Manual Conversion Patterns

### JSON Schema → TypeScript

| JSON Schema | TypeScript |
|-------------|------------|
| `{ "type": "string" }` | `string` |
| `{ "type": "integer" }` | `number` |
| `{ "type": "boolean" }` | `boolean` |
| `{ "type": "array", "items": { "$ref": "#/$defs/Item" } }` | `Item[]` |
| `{ "type": "object", "properties": { "name": {...} } }` | `{ name: string; }` |
| `{ "enum": ["a", "b"] }` | `"a" \| "b"` |
| `{ "oneOf": [...] }` | Union type |
| `{ "$ref": "#/$defs/User" }` | `User` |
| `{ "anyOf": [{ "type": "string" }, { "type": "null" }] }` | `string \| null` |

### Example: Complete JSON Schema → TypeScript

**Input schema.json:**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$defs": {
    "User": {
      "type": "object",
      "required": ["id", "email"],
      "properties": {
        "id": { "type": "string", "format": "uuid" },
        "email": { "type": "string", "format": "email" },
        "name": { "type": "string" },
        "role": { "type": "string", "enum": ["admin", "user", "guest"] },
        "createdAt": { "type": "string", "format": "date-time" },
        "metadata": { "type": "object", "additionalProperties": true }
      }
    }
  }
}
```

**Output types.ts:**
```typescript
/**
 * Auto-generated from schema.json
 * @fileoverview User and related types
 */

export interface User {
  /** Format: uuid */
  id: string;
  email: string; // format: email
  name?: string;
  role?: 'admin' | 'user' | 'guest';
  /** Format: date-time */
  createdAt?: string;
  [key: string]: unknown; // additionalProperties: true
}

// Re-export for convenience
export type { User as default };
```

### OpenAPI 3.x → TypeScript Patterns

```typescript
// From OpenAPI components
export interface User {
  id: string;
  email: string;
  name?: string;
  role?: 'admin' | 'user' | 'guest';
}

export interface Error {
  code: string;
  message: string;
  details?: Record<string, unknown>;
}

// Request/Response types from operations
export interface GetUserRequest {
  userId: string;
}

export interface GetUserResponse {
  data: User;
  meta?: {
    requestId: string;
  };
}
```

## Quality Checklist

After conversion, verify:

- [ ] All `$ref` references resolved (no dangling refs)
- [ ] Required fields correctly marked (non-optional)
- [ ] Union types for `oneOf`/`anyOf` handled
- [ ] Enum values preserved as literal types
- [ ] Array types use correct syntax (`Type[]` vs `Array<Type>`)
- [ ] Documentation/comments preserved from schema
- [ ] Naming follows project conventions (camelCase for props)
- [ ] No `any` types introduced (unless schema explicitly allows)

## Common Issues & Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| Generated `any` | Unresolvable `$ref` | Resolve refs manually |
| All fields optional | JSON Schema defaults | Add explicit `required` array |
| Missing union types | `oneOf` not expanded | Manually create union |
| Wrong array syntax | Nested arrays | Use `Array<Array<T>>` |
| Duplicate types | Multiple schema files | Consolidate with `$defs` |

## Output Format

Structure the generated types file as:

```typescript
/**
 * Auto-generated TypeScript types
 * Source: {schema_source}
 * Generated: {date}
 */

// Tipe definitions

export interface TypeName {
  field: string;
  optionalField?: string;
}

// Utility types

export type Status = 'active' | 'inactive';
```

## Tools

- **openapi-typescript:** `npx openapi-typescript <schema> --output <outfile>`
- **quicktype:** `npx @quicktype/quicktype --schema <file> --lang typescript`
- **json-schema-to-typescript:** `npx json-schema-to-typescript <file>`
- **GraphQL Code Generator:** `npx graphql-codegen`
- **ajv:** Validate instance data against schema
