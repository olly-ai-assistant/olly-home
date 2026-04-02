name: codebase-summary
description: Generates high-level architectural summaries of a codebase — its structure, tech stack, patterns, and purpose. Use when asked to "summarize this codebase", "explain the architecture", "what does this project do", "give me an overview of the code", "analyze project structure", or when onboarding to a new project, reviewing unfamiliar code, or writing documentation. Invaluable for researchers exploring codebases, coders understanding context, docs writers explaining projects, testers grasping test strategy, and reviewers getting quick architectural overviews.
---

# Codebase Summary

Generates concise, high-level architectural summaries of codebases — structure, tech stack, patterns, and purpose. Helps quickly understand an unfamiliar project or document an existing one.

## When to Use This Skill

Use this skill when:
- Asked to "summarize this codebase", "explain the architecture", or "what does this project do"
- Onboarding to a new project and need a quick overview
- Reviewing code in a PR and want architectural context
- Writing documentation and need to explain a project
- Testing and need to understand the system under test
- Researching a codebase for security audit or due diligence
- Comparing projects or extracting patterns across codebases

## Summary Process

### Step 1: Initial Reconnaissance

```bash
# Project metadata
cat package.json 2>/dev/null || cat pyproject.toml 2>/dev/null || cat Cargo.toml 2>/dev/null
cat README.md 2>/dev/null | head -50

# Language/framework detection
ls -la
file $(find . -maxdepth 2 -type f | head -20)
```

### Step 2: Directory Structure Analysis

```bash
# Directory tree (limit depth)
find . -maxdepth 3 -type d | head -50

# Key config files
ls -la *.json *.yaml *.yml *.toml Makefile Dockerfile 2>/dev/null

# Source directories
find . -maxdepth 2 -type d -name "src" -o -name "lib" -o -name "app" -o -name "packages"
```

### Step 3: Tech Stack Identification

**Node.js/TypeScript:**
```bash
cat package.json | jq '{name, version, dependencies, devDependencies, scripts}'
ls node_modules/.bin/ 2>/dev/null | head -20  # What tools are installed
```

**Python:**
```bash
cat requirements.txt 2>/dev/null || cat pyproject.toml | jq '.dependencies'
pip list 2>/dev/null | head -20
```

**Go:**
```bash
cat go.mod
ls *.go | head -10
```

**Multi-language/monorepo:**
```bash
ls -la
cat package.json 2>/dev/null
cat Cargo.toml 2>/dev/null
cat docker-compose.yml 2>/dev/null
```

### Step 4: Architecture Pattern Detection

```bash
# MVC / Layered?
find . -maxdepth 2 -type d | grep -iE "controller|service|repository|model|view"

# Domain-driven?
find . -maxdepth 3 -type d | grep -iE "domain|entity|aggregate|value"

# Event-driven / Microservices?
grep -r "kafka\|rabbitmq\|pubsub\|event" . --include="*.yaml" --include="*.json" 2>/dev/null | head -5

# API style?
grep -r "rest\|graphql\|grpc\|websocket" . --include="*.ts" --include="*.js" 2>/dev/null | head -5
```

### Step 5: Entry Points & Key Files

```bash
# Main entry points
cat index.* 2>/dev/null
cat main.* 2>/dev/null
cat src/index.ts 2>/dev/null || cat src/main.py 2>/dev/null

# CLI tools
grep -r "#!/usr/bin/env" . --include="*.py" --include="*.ts" --include="*.js" 2>/dev/null

# Config entry
cat app.config.* 2>/dev/null || cat config.* 2>/dev/null
```

## Output Format

Structure the summary as:

```markdown
# {Project Name} — Codebase Summary

## Overview
One-paragraph description of what the project does and its primary purpose.

## Tech Stack
- **Language:** TypeScript (Node.js)
- **Framework:** Express.js
- **Database:** PostgreSQL with Prisma ORM
- **Key libraries:** Zod (validation), JWT (auth), Bull (queues)

## Project Structure
.
├── src/
│   ├── api/          # HTTP handlers / routes
│   ├── services/     # Business logic
│   ├── repositories/  # Data access layer
│   ├── models/       # Domain entities / types
│   └── utils/        # Shared utilities
├── tests/            # Test suites
├── scripts/          # Build/dev scripts
└── config/           # Configuration files

## Architecture Pattern
- **Layered architecture** with clear separation: API → Service → Repository
- **Dependency injection** via constructor injection
- **Repository pattern** for data access abstraction

## Key Components

| Component | Purpose | Key Files |
|-----------|---------|-----------|
| AuthService | JWT authentication, session management | `src/services/auth.ts` |
| UserRepository | User CRUD via Prisma | `src/repositories/user.ts` |
| API Routes | REST endpoints | `src/api/routes/` |

## Data Flow
1. HTTP request → Express router → Controller
2. Controller validates request → calls Service
3. Service executes business logic → calls Repository
4. Repository queries database → returns to Service
5. Service transforms result → Controller returns HTTP response

## Patterns & Conventions

- **Error handling:** Custom AppError class with HTTP status codes
- **Validation:** Zod schemas for request/response validation
- **Logging:** Structured JSON logs with request ID correlation
- **Testing:** Jest with integration tests + unit tests

## Build & Run
```bash
npm install
npm run dev      # Development
npm run build    # Production build
npm test         # Run tests
```

## Notable Features
- Real-time updates via WebSocket
- Background job processing with Bull queues
- Rate limiting and request validation
- Health check endpoint at `/health`
```

## What to Look For

### For Security Reviews
- Authentication/authorization mechanisms
- Input validation approach
- Secrets management (look for hardcoded secrets, env vars)
- Dependency vulnerabilities (`npm audit`, `snyk`)
- SQL query patterns (SQL injection risks)

### For Code Quality
- Code organization and modularity
- Test coverage and testability
- Error handling patterns
- Duplication and potential refactoring targets
- Documentation quality

### For Performance
- Database query patterns (N+1 problems)
- Caching strategies
- Async processing (queues, workers)
- Connection pooling

## Aggregation Commands

Quick multi-file analysis:

```bash
# Count lines by language
find . -name "*.ts" -o -name "*.js" | xargs wc -l | tail -1
find . -name "*.py" | xargs wc -l 2>/dev/null | tail -1

# Find largest files
find . -type f \( -name "*.ts" -o -name "*.js" \) -exec wc -l {} + | sort -rn | head -10

# Count files by extension
find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -10

# Find all dependencies (flattened)
cat package.json | jq '.dependencies | keys[]' | tr -d '"'
```

## Summary Levels

| Level | Depth | Use Case |
|-------|-------|----------|
| Quick (30 sec) | Top-level structure, README | First pass |
| Standard (2 min) | Tech stack, architecture, key files | PR review context |
| Deep (5+ min) | Patterns, data flow, security, quality | Full audit |

Default to **Standard** unless asked for quick or deep.
