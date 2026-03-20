# Checklist: Project Start

## Before First Line of Code

- [ ] Requirements documented (what, not how)
- [ ] Tech stack decided (ADR written)
- [ ] Repository created
- [ ] `.ai/` context files initialized
  - [ ] AGENTS.md (roles, workflow)
  - [ ] ARCHITECTURE.md (structure, patterns)
  - [ ] TASKS.md (initial backlog)
  - [ ] DECISIONS.md (first ADR: tech stack)
- [ ] `.gitignore` (node_modules, .env, dist, .next)
- [ ] Package.json with scripts (dev, build, test, lint)
- [ ] TypeScript strict mode configured
- [ ] Linter configured (ESLint)
- [ ] Database schema designed (Prisma schema)
- [ ] `.env.example` with all required variables
- [ ] README.md with Quick Start

## Before First Deploy

- [ ] Docker multi-stage Dockerfile
- [ ] docker-compose.yml (local dev)
- [ ] docker-compose.traefik.yml (production)
- [ ] Health endpoint (`GET /health`)
- [ ] DNS configured
- [ ] SSL working (Let's Encrypt)
- [ ] Admin user created
- [ ] Default passwords changed
