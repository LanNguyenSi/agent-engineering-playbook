# ADR Template

Architecture Decision Records document WHY, not WHAT.

```markdown
# ADR-XXX: [Decision Title]

**Date:** YYYY-MM-DD
**Status:** Proposed / Accepted / Superseded / Deprecated
**Deciders:** [Who made this decision]

## Context

What is the problem or situation that requires a decision?

## Options Considered

### Option A: [Name]
- Pro: ...
- Con: ...

### Option B: [Name]
- Pro: ...
- Con: ...

## Decision

Which option was chosen and why.

## Consequences

### Positive
- What improves

### Negative
- What trade-offs we accept

### Migration
- What needs to change in existing code (if any)
```

## Example

```markdown
# ADR-004: Traefik over Nginx for Reverse Proxy

**Date:** 2026-03-18
**Status:** Accepted
**Deciders:** Lan, Ice

## Context
Need a reverse proxy for multiple services (Triologue, Event Booking, Health Dashboard) with automatic SSL.

## Options
- **Nginx:** Quick setup, manual SSL config, manual per-service config
- **Traefik:** Docker-native, automatic SSL, label-based routing

## Decision
Traefik. "Nachhaltigkeit > Schnelligkeit" (Lan). Initial setup takes longer but adding new services is trivial.

## Consequences
- Positive: New services in 30 seconds, automatic SSL
- Negative: Traefik-specific knowledge required, more complex initial setup
```
