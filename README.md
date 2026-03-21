# Agent Engineering Playbook

Pragmatic guidance for building AI-assisted software that can survive enterprise expectations: security reviews, auditability, reliability, change control, and production operations.

## What This Repository Is

This repository is a working playbook for teams building production systems with human engineers and AI agents. It does not assume that speed alone is success. It treats delivery, safety, operability, and maintainability as first-class concerns.

The material is organized into nine playbooks:

1. [Project Setup](playbooks/01-project-setup.md)
2. [Architecture](playbooks/02-architecture.md)
3. [Team Roles](playbooks/03-team-roles.md)
4. [Design Principles](playbooks/04-design-principles.md)
5. [Development Workflow](playbooks/05-development-workflow.md)
6. [Testing Strategy](playbooks/06-testing-strategy.md)
7. [Quality Assurance](playbooks/07-quality-assurance.md)
8. [Documentation](playbooks/08-documentation.md)
9. [Production Readiness](playbooks/09-production.md)

## What Changed In This Hardening Pass

The original material had strong practical instincts, but several sections were still too opinionated for enterprise use:

- Roles were too rigid and depended too much on a single lead.
- Review and release guidance was sometimes optimized for local velocity over controlled change.
- Security, compliance, incident handling, disaster recovery, and supply-chain concerns were underrepresented.
- Some recommendations were tool-specific, fragile, or unsafe as general guidance.
- A few repository references pointed to files that did not exist.

This revision shifts the playbook toward:

- risk-based delivery instead of personality-based process
- explicit governance and traceability
- secure-by-default engineering practices
- operational readiness before production
- short feedback loops without lowering release discipline

## How To Use It

For a new initiative:

1. Start with [Project Start Checklist](checklists/project-start.md).
2. Write a project charter, architecture context, and initial ADRs.
3. Choose the smallest architecture and delivery model that satisfies current risk.
4. Set release gates before feature work begins.

For an existing product:

1. Run the [Pre-Production Checklist](checklists/pre-production.md).
2. Compare current process against [Development Workflow](playbooks/05-development-workflow.md) and [Quality Assurance](playbooks/07-quality-assurance.md).
3. Close missing controls in testing, observability, documentation, and rollout strategy.

## Core Principles

- Start with clear outcomes, constraints, and operating assumptions.
- Prefer simple architectures until complexity is justified by scale or risk.
- Keep changes small, testable, and reversible.
- Require evidence for shipping: tests, review, observability, and rollback.
- Treat documentation, runbooks, and decision records as part of the product.
- Use AI agents as force multipliers, not as substitutes for ownership and controls.

## Repository Structure

### Playbooks
- [playbooks/01-project-setup.md](playbooks/01-project-setup.md)
- [playbooks/02-architecture.md](playbooks/02-architecture.md)
- [playbooks/03-team-roles.md](playbooks/03-team-roles.md)
- [playbooks/04-design-principles.md](playbooks/04-design-principles.md)
- [playbooks/05-development-workflow.md](playbooks/05-development-workflow.md)
- [playbooks/06-testing-strategy.md](playbooks/06-testing-strategy.md)
- [playbooks/07-quality-assurance.md](playbooks/07-quality-assurance.md)
- [playbooks/08-documentation.md](playbooks/08-documentation.md)
- [playbooks/09-production.md](playbooks/09-production.md)

### Checklists
- [checklists/project-start.md](checklists/project-start.md)
- [checklists/code-review.md](checklists/code-review.md)
- [checklists/pre-production.md](checklists/pre-production.md)

### Templates
- [templates/task-template.md](templates/task-template.md)
- [templates/PR-template.md](templates/PR-template.md)
- [templates/ADR-template.md](templates/ADR-template.md)
- [templates/README-template.md](templates/README-template.md)
- [templates/SECURITY-template.md](templates/SECURITY-template.md)

### Repo Standards
- [CONTRIBUTING.md](CONTRIBUTING.md)
- [SECURITY.md](SECURITY.md)

## Intended Audience

- Engineering leads establishing delivery standards
- Staff and senior engineers designing architecture and release controls
- Product teams working with AI agents in the development loop
- Founders and CTOs moving from prototype behavior to production discipline

## Scope And Non-Goals

This repository is intentionally vendor-neutral. It may mention specific tools, but it does not assume a single stack, cloud, proxy, framework, or AI workflow.

It is not:

- a certification framework
- a substitute for legal, privacy, or security specialists
- a promise that every team needs heavyweight process from day one

The goal is proportional rigor: enough control for the risk profile you actually have.

## Contributing

Use [CONTRIBUTING.md](CONTRIBUTING.md). Proposed changes should explain:

- what weakness in the current guidance is being addressed
- what operational or engineering problem the revision prevents
- what tradeoff the new guidance introduces

## License

MIT
