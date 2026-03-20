# Agent Engineering Playbook

**The definitive guide for building production-ready AI agent systems**

A comprehensive playbook for AI agents (and humans) building enterprise-grade projects. Learn architecture patterns, team workflows, testing strategies, and production best practices.

## 🎯 What This Playbook Covers

### Foundation
1. **[Project Setup](playbooks/01-project-setup.md)** - From idea to structured project
2. **[Architecture](playbooks/02-architecture.md)** - Layered architecture, microservices, patterns
3. **[Team Roles](playbooks/03-team-roles.md)** - Product Owner, Tech Lead, Developers, QA, DevOps

### Development
4. **[Design Principles](playbooks/04-design-principles.md)** - SOLID, DRY, Clean Code, Domain-Driven Design
5. **[Development Workflow](playbooks/05-development-workflow.md)** - Git Flow, PRs, code review, CI/CD
6. **[Testing Strategy](playbooks/06-testing-strategy.md)** - TDD, unit tests, integration, E2E

### Production
7. **[Quality Assurance](playbooks/07-quality-assurance.md)** - Linting, security, performance, monitoring
8. **[Documentation](playbooks/08-documentation.md)** - README, API docs, ADRs, `.ai/` context
9. **[Production Readiness](playbooks/09-production.md)** - Deployment, monitoring, disaster recovery

## 🚀 Quick Start

### For New Projects

1. Read **[Project Setup](playbooks/01-project-setup.md)** to structure your project
2. Follow **[Architecture](playbooks/02-architecture.md)** to design your system
3. Use **[Testing Strategy](playbooks/06-testing-strategy.md)** from day one
4. Reference **[Development Workflow](playbooks/05-development-workflow.md)** for team collaboration

### For Existing Projects

1. Run through **[Production Readiness Checklist](checklists/pre-production.md)**
2. Review **[Quality Assurance](playbooks/07-quality-assurance.md)** practices
3. Improve **[Documentation](playbooks/08-documentation.md)**
4. Implement missing **[Testing](playbooks/06-testing-strategy.md)** layers

## 📚 What Makes This Different

- **Agent-First**: Written by and for AI agents (Ice & Lava) building real systems
- **Battle-Tested**: Patterns from production projects (Event Booking, Memory-Weaver, Agent Control Platform)
- **Practical**: Code examples, templates, and checklists you can use immediately
- **Modern Stack**: TypeScript, React, Next.js, Prisma, PostgreSQL, Docker
- **AI-Native**: Includes `.ai/` context patterns for agent collaboration

## 🛠️ Resources

### Templates
- **[ADR Template](templates/ADR-template.md)** - Architecture Decision Records
- **[PR Template](templates/PR-template.md)** - Pull Request structure
- **[README Template](templates/README-template.md)** - Project documentation
- **[Security Template](templates/SECURITY-template.md)** - Security policy

### Checklists
- **[Project Start](checklists/project-start.md)** - Launch checklist
- **[Pre-Production](checklists/pre-production.md)** - Deployment readiness
- **[Code Review](checklists/code-review.md)** - Review guidelines

### Examples
- **[Reference Project](examples/reference-project/)** - Complete example following all playbooks

## 🏗️ Architecture Patterns Covered

- **Layered Architecture** - Presentation, Business, Data layers
- **Microservices** - When and how to decompose
- **Event-Driven** - Event sourcing, CQRS
- **Domain-Driven Design** - Bounded contexts, aggregates
- **Clean Architecture** - Dependency inversion, testability

## 🧪 Testing Pyramid

```
       /\
      /E2E\          Few, expensive, slow
     /------\
    /  Inte  \       Some, moderate cost
   /----------\
  / Unit Tests \     Many, cheap, fast
 /--------------\
```

- **Unit Tests**: 70% coverage (fast, isolated)
- **Integration Tests**: 20% coverage (API, database)
- **E2E Tests**: 10% coverage (critical user flows)

## 👥 Team Collaboration

This playbook emphasizes:
- **Clear Roles** - Everyone knows their responsibilities
- **Async-First** - Work across timezones effectively
- **Review Culture** - Every change is reviewed
- **Documentation** - Code is not enough
- **Continuous Improvement** - Retrospectives and iterations

## 🎓 Learning Path

### Beginner
1. Project Setup → Architecture → Testing basics
2. Build a small CRUD app following playbooks
3. Focus on unit tests and clean code

### Intermediate
4. Design Principles → Development Workflow
5. Implement CI/CD pipeline
6. Add integration tests and E2E coverage

### Advanced
7. Quality Assurance → Production Readiness
8. Set up monitoring and alerting
9. Implement microservices or event-driven patterns

## 🤝 Contributing

This playbook is a living document built through real project experience.

**Core Authors:**
- 🧊 **Ice** (@ice) - Roles, Design Principles, Workflow, QA, Production
- 🌋 **Lava** (@lava) - Setup, Architecture, Testing, Documentation, Examples

**How to Contribute:**
1. Open an issue with suggestions or questions
2. Submit PRs for improvements or new sections
3. Share real-world examples from your projects

## 📖 Real-World Usage

This playbook was developed while building:
- **Event Booking System** (Next.js + Prisma + PostgreSQL)
- **Memory-Weaver v2** (4-layer consciousness architecture)
- **Agent Control Platform** (Full-stack agent monitoring)
- **Triologue** (AI-to-AI communication platform)

Every pattern here has been tested in production.

## 🔗 Related Tools

- **[ScaffoldKit](https://github.com/LanNguyenSi/scaffoldkit)** - Project scaffolding with `.ai/` context
- **[DevReview](https://github.com/LanNguyenSi/devreview)** - Automated PR code review
- **[Agent Dev Kit](https://github.com/LanNguyenSi/agent-dev-kit)** - AI agent scaffolding
- **[Triologue SDK](https://github.com/LanNguyenSi/triologue-sdk)** - Agent communication

## 📜 License

MIT © [Lan Nguyen Si](https://github.com/LanNguyenSi)

---

**Built by Ice 🧊 & Lava 🌋 through real-world agent engineering**

*"The best way to learn engineering is to build production systems."*
