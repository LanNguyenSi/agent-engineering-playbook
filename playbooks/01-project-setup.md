# 01 - Project Setup

**From idea to structured project in 10 steps**

## Overview

A solid project setup is the foundation of successful software development. This playbook guides you through structuring a new project following enterprise best practices.

## Prerequisites

- Node.js 18+ installed
- Git configured
- Package manager (npm, yarn, or pnpm)
- Code editor (VS Code recommended)

## 10-Step Project Setup

### Step 1: Define the Project

Before writing any code, clarify:

**✅ Requirements**
- What problem does this solve?
- Who are the users?
- What are the core features?
- What are the non-functional requirements (performance, security, scalability)?

**✅ Success Criteria**
- How do you measure success?
- What metrics matter?
- What does "done" look like?

**Example:**
```markdown
## Project: Event Booking System

### Problem
Users need to book events, manage attendees, and handle payments

### Core Features
1. Event creation and management
2. Ticket booking with payment
3. Email confirmations
4. Admin dashboard

### Success Criteria
- Handle 1000+ concurrent bookings
- 99.9% uptime
- < 2s page load time
- PCI DSS compliant payments
```

### Step 2: Choose Technology Stack

Select technologies based on:
- **Team expertise**
- **Project requirements**
- **Ecosystem maturity**
- **Long-term maintainability**

**Modern Full-Stack (Recommended)**
```typescript
// Frontend
- React 18+ or Next.js 14+
- TypeScript (mandatory for type safety)
- Tailwind CSS (utility-first styling)
- Radix UI / shadcn/ui (accessible components)

// Backend
- Next.js API Routes or Express.js
- Prisma ORM (database abstraction)
- PostgreSQL (relational data)
- Redis (caching, sessions)

// Infrastructure
- Docker (containerization)
- Vercel or Railway (deployment)
- GitHub Actions (CI/CD)
```

**Document your choices** in `.ai/DECISIONS.md` (Architecture Decision Records)

### Step 3: Initialize Project Structure

Use scaffolding tools or create manually:

**Option A: ScaffoldKit** (Recommended)
```bash
scaffoldkit new my-project --blueprint nextjs-fullstack
```

**Option B: Manual Setup**
```bash
# Create project
npx create-next-app@latest my-project --typescript --tailwind --app

# Initialize git
cd my-project
git init
git add .
git commit -m "Initial commit"

# Create standard directories
mkdir -p src/{components,lib,app,types}
mkdir -p prisma
mkdir -p tests/{unit,integration,e2e}
mkdir -p .ai
```

### Step 4: Set Up Development Environment

**Essential Configuration Files:**

**1. TypeScript (`tsconfig.json`)**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM"],
    "jsx": "preserve",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true
  }
}
```

**2. ESLint (`.eslintrc.json`)**
```json
{
  "extends": [
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "error"
  }
}
```

**3. Prettier (`.prettierrc`)**
```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100
}
```

**4. Git Hooks (`package.json`)**
```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{js,ts,tsx}": ["eslint --fix", "prettier --write"]
  }
}
```

### Step 5: Create `.ai/` Context Files

AI-native projects use `.ai/` for agent collaboration:

**`.ai/AGENTS.md`** - Team roles and workflows
```markdown
# Team

## Agents
- Ice (@ice) — Backend, architecture
- Lava (@lava) — Frontend, testing

## Workflow
1. Pick task from TASKS.md
2. Create feature branch
3. Implement + test
4. Open PR for review
5. Deploy after approval
```

**`.ai/ARCHITECTURE.md`** - System overview
```markdown
# Architecture

## Stack
- Next.js 14 (App Router)
- PostgreSQL + Prisma
- Tailwind CSS + shadcn/ui

## Structure
\`\`\`
src/
├── app/         # Next.js pages
├── components/  # React components
├── lib/         # Business logic
└── types/       # TypeScript types
\`\`\`
```

**`.ai/TASKS.md`** - Current work
```markdown
# Tasks

## In Progress
- [ ] User authentication (Ice)
- [ ] Event listing page (Lava)

## Backlog
- [ ] Payment integration
- [ ] Email notifications
```

**`.ai/DECISIONS.md`** - ADRs
```markdown
# Architecture Decisions

## ADR-001: Use Next.js App Router
**Date:** 2026-03-20
**Status:** Accepted
**Decision:** Use Next.js 14 App Router over Pages Router
**Rationale:** Better performance, React Server Components, improved DX
```

### Step 6: Set Up Database

**1. Choose Database**
- **PostgreSQL**: Relational data, ACID transactions
- **MongoDB**: Flexible schemas, document storage
- **Redis**: Caching, sessions, real-time

**2. Initialize Prisma** (for PostgreSQL)
```bash
npm install prisma @prisma/client
npx prisma init
```

**3. Define Schema** (`prisma/schema.prisma`)
```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

**4. Run Migration**
```bash
npx prisma migrate dev --name init
npx prisma generate
```

### Step 7: Environment Configuration

**`.env.local`** (never commit!)
```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"

# Auth
NEXTAUTH_SECRET="your-secret-key"
NEXTAUTH_URL="http://localhost:3000"

# External Services
STRIPE_SECRET_KEY="sk_test_..."
SENDGRID_API_KEY="SG...."
```

**`.env.example`** (commit this!)
```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"

# Auth
NEXTAUTH_SECRET="your-secret-key"
NEXTAUTH_URL="http://localhost:3000"

# External Services (get your own keys)
STRIPE_SECRET_KEY=""
SENDGRID_API_KEY=""
```

### Step 8: Set Up Testing

**Install Dependencies**
```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom
npm install -D playwright @playwright/test
```

**Vitest Config** (`vitest.config.ts`)
```typescript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./tests/setup.ts'],
  },
});
```

**Example Unit Test** (`tests/unit/utils.test.ts`)
```typescript
import { describe, it, expect } from 'vitest';
import { formatDate } from '@/lib/utils';

describe('formatDate', () => {
  it('formats dates correctly', () => {
    const date = new Date('2026-03-20');
    expect(formatDate(date)).toBe('March 20, 2026');
  });
});
```

### Step 9: Documentation

**Essential Docs:**

**`README.md`**
```markdown
# My Project

Brief description

## Setup
1. \`npm install\`
2. Copy \`.env.example\` to \`.env.local\`
3. \`npx prisma migrate dev\`
4. \`npm run dev\`

## Testing
- \`npm test\` — Unit tests
- \`npm run test:e2e\` — E2E tests

## Deployment
See [DEPLOYMENT.md](docs/DEPLOYMENT.md)
```

**`CONTRIBUTING.md`**
```markdown
# Contributing

1. Create feature branch: \`git checkout -b feature/my-feature\`
2. Make changes + write tests
3. Run \`npm test\` and \`npm run lint\`
4. Open PR with description
5. Wait for review
```

### Step 10: CI/CD Pipeline

**GitHub Actions** (`.github/workflows/ci.yml`)
```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

## Project Structure Example

Final structure after setup:

```
my-project/
├── .ai/
│   ├── AGENTS.md
│   ├── ARCHITECTURE.md
│   ├── TASKS.md
│   └── DECISIONS.md
├── src/
│   ├── app/              # Next.js pages (App Router)
│   ├── components/       # React components
│   ├── lib/              # Business logic + utilities
│   └── types/            # TypeScript types
├── prisma/
│   └── schema.prisma     # Database schema
├── tests/
│   ├── unit/             # Unit tests
│   ├── integration/      # API/DB tests
│   └── e2e/              # End-to-end tests
├── .github/
│   └── workflows/        # CI/CD
├── .env.example          # Environment template
├── .gitignore
├── package.json
├── tsconfig.json
├── README.md
└── CONTRIBUTING.md
```

## Checklist

Use this when starting a new project:

- [ ] Requirements defined and documented
- [ ] Technology stack chosen and recorded in ADRs
- [ ] Project initialized with proper structure
- [ ] TypeScript + ESLint + Prettier configured
- [ ] `.ai/` context files created
- [ ] Database set up with migrations
- [ ] Environment variables configured
- [ ] Testing framework installed
- [ ] Documentation written (README, CONTRIBUTING)
- [ ] CI/CD pipeline configured
- [ ] First commit pushed to main branch

## Next Steps

After project setup:
1. Read **[02-Architecture](02-architecture.md)** to design your system
2. Follow **[06-Testing Strategy](06-testing-strategy.md)** to implement TDD
3. Use **[05-Development Workflow](05-development-workflow.md)** for team collaboration

## Common Pitfalls

❌ **Starting without clear requirements** → Leads to scope creep and rework
❌ **Skipping TypeScript strict mode** → Type bugs in production
❌ **No testing from day one** → Hard to add later, bugs accumulate
❌ **Poor git hygiene** → Large commits, no history, hard to review
❌ **Missing documentation** → New team members struggle to onboard

✅ **Do this instead:** Follow all 10 steps systematically

## Real-World Example

This setup was used for:
- **Event Booking System** (Next.js + Prisma + PostgreSQL)
- **Agent Control Platform** (Full-stack monitoring)
- **Memory-Weaver v2** (4-layer architecture)

Every production project starts with solid foundations.

---

**Next:** [02 - Architecture →](02-architecture.md)
