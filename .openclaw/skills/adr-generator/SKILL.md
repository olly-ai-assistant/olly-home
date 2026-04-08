name: adr-generator
description: Creates Architecture Decision Records (ADRs) using lightweight Markdown templates. Use when asked to "write an ADR", "document an architectural decision", "create a decision record", "log a technical decision", "capture architecture choice", or when starting a significant technical change, evaluating alternatives, or reviewing past decisions.
---

# ADR Generator

Creates Architecture Decision Records (ADRs) — lightweight documents that capture important technical decisions, the context that drove them, and their consequences.

## When to Use This

- Starting a significant technical change
- Evaluating competing approaches or tools
- Reviewing past decisions during oncall/debugging
- Auditing why the system is built a certain way
- Team alignment on architecture choices

## ADR Format (Nygard-style)

Place in `docs/adr/ADR-XXX-title.md`:

```markdown
# ADR-XXX: Title

**Date:** YYYY-MM-DD
**Status:** Proposed | Accepted | Deprecated | Superseded by ADR-YYY
**Deciders:** Name1, Name2

## Context

What is the issue that we're seeing that is motivating this decision?

- Problem statement
- Forces at play (technology, team, constraints)
- What options were considered

## Decision

What is the change that is being proposed and/or agreed?

## Consequences

### Positive
- Benefit 1
- Benefit 2

### Negative
- Drawback 1
- Drawback 2

### Neutral
- Trade-off noted
- Something that will need to be monitored
```

## Quick Generation

```bash
# Create from template
cat > docs/adr/ADR-001-$(echo "$TITLE" | tr ' ' '-').md << 'TEMPLATE'
# ADR-001: TITLE

**Date:** $(date +%Y-%m-%d)
**Status:** Proposed
**Deciders:** TODO

## Context

TODO — describe the situation and forces at play.

## Decision

TODO — describe what was decided.

## Consequences

### Positive
-

### Negative
-

### Neutral
-
TEMPLATE

echo "Created: docs/adr/ADR-001-title.md"
```

## ADR Numbering

- Start at `ADR-0001`
- Never reuse numbers (even if an ADR is deprecated)
- A "Superseded" ADR should reference who supersedes it
- Use leading zeros for sorting (001, 002, not 1, 2)

## Common ADR Triggers

| Trigger | Example |
|---------|---------|
| New technology | "We'll use PostgreSQL instead of MongoDB" |
| Architecture change | "Move from monolith to microservices" |
| API design | "REST vs GraphQL for mobile client" |
| Infrastructure | "Self-hosted vs managed Kubernetes" |
| Process | "Adopt trunk-based development" |
| Deprecation | "Drop support for Python 2" |

## ADR Workflow

```
Proposed → Accepted → (Deprecated | Superseded)
     ↓
  Rejected
```

## Index File

Maintain `docs/adr/README.md`:

```markdown
# Architecture Decision Records

| ID | Title | Status | Date |
|----|-------|--------|------|
| ADR-001 | Use PostgreSQL for user data | Accepted | 2026-04-08 |
| ADR-002 | REST API with OpenAPI spec | Accepted | 2026-04-08 |

---

## Recent Decisions

### ADR-001: Use PostgreSQL for user data
We chose PostgreSQL over MongoDB because... [link]
```

## Tools

- **adr-tools** (Python): `pip install adr-tools` — CLI for creating/managing ADRs
- **adr-viewer**: Visualize ADR relationships
- **marick/architecture-decision-records**: The original MADR template format

## Tips

- Keep it short — a good ADR fits on 1-2 pages
- Write for future readers who weren't in the room
- Capture *why*, not just *what*
- Link to meeting notes, Slack threads, or RFCs for more context
- Update status when the decision is revisited
