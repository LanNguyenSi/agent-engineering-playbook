# 08 - Documentation

Documentation is part of the control plane of the product. It reduces onboarding cost, prevents repeated mistakes, and makes operations survivable.

## Documentation Principles

- write for the next engineer, not for the current author
- document decisions and operating assumptions, not obvious syntax
- keep docs close to the work they describe
- assign owners and review cadence for critical docs

## The Documentation Stack

### README

Should answer:

- what the system does
- who it serves
- how to run it locally
- where the important docs live

### CONTRIBUTING

Should define:

- branch and review expectations
- required local checks
- commit and PR hygiene
- how to propose changes to standards

### SECURITY

Should explain:

- how to report vulnerabilities
- what is considered sensitive
- what access and secret-handling rules apply

### ADRs

Use ADRs for decisions with lasting impact:

- architecture choices
- major tool or platform commitments
- data or security model changes
- rollout or migration strategy changes

### Runbooks

Every operated service should have runbooks for:

- deploy
- rollback
- incident triage
- backup and restore
- common failures

### Operational Docs

Maintain current references for:

- environment variables and secrets ownership
- dashboards and alerts
- on-call ownership
- service dependencies

### API And Contract Docs

Document:

- external APIs
- integration contracts
- authentication requirements
- error semantics
- deprecation policy

### Data Docs

For systems with meaningful data complexity, document:

- ownership of major entities
- retention and deletion rules
- schema migration expectations
- reporting or analytics boundaries

## AI-Specific Context

AI-oriented docs such as `.ai/` can be useful for task context, conventions, and agent boundaries. They should complement, not replace, standard engineering documentation.

If you rely on agents, document:

- what they are allowed to change
- where they get authoritative context
- how human review is expected to happen

## Documentation Freshness

Critical documents need owners.

Review periodically:

- release and incident runbooks
- onboarding instructions
- architecture diagrams
- environment and dependency inventories

Outdated runbooks are often worse than missing runbooks.

## Anti-Patterns

- docs that describe an idealized system instead of the real one
- one giant document with no ownership boundaries
- design decisions living only in chat history
- production procedures that only exist in one engineer's head

## Exit Criteria

Documentation quality is acceptable when:

- a new engineer can onboard without constant interruption
- reviewers can understand the why behind major decisions
- operators have current runbooks for common tasks and failures
- AI context files are aligned with the official engineering docs
