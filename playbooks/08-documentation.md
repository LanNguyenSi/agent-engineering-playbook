# 08 — Documentation

> Code tells you *how*. Documentation tells you *why*.

## Why Documentation Matters

**Without docs:**
- ❌ New team members take weeks to onboard
- ❌ "How does this work?" questions interrupt deep work
- ❌ Decisions are forgotten (why did we choose Prisma over TypeORM?)
- ❌ AI agents hallucinate (no context = wrong assumptions)

**With docs:**
- ✅ Self-service onboarding (README → running app in 10 minutes)
- ✅ Async collaboration (read docs instead of asking)
- ✅ Historical context (ADRs preserve reasoning)
- ✅ AI agents work autonomously (`.ai/` files guide behavior)

**The key:** Documentation is not an afterthought. It's part of the workflow.

## The Documentation Stack

### 1. README.md (Project Overview)

**Purpose:** First file anyone reads. Answers "What is this and how do I run it?"

**Structure:**

```markdown
# Project Name

One-sentence description of what this does.

## What It Does

2-3 paragraphs explaining the problem and solution.

## Quick Start

\`\`\`bash
git clone <repo>
cd project
npm install
cp .env.example .env.local
npm run dev
\`\`\`

Open http://localhost:3000

## Tech Stack

- Next.js 14 (App Router)
- TypeScript
- Prisma + PostgreSQL
- Tailwind CSS

## Project Structure

\`\`\`
src/
├── app/         # Next.js pages
├── components/  # React components
├── lib/         # Business logic
└── types/       # TypeScript types
\`\`\`

## Development

\`\`\`bash
npm run dev        # Start dev server
npm test           # Run tests
npm run build      # Build for production
\`\`\`

## Deployment

See [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md)

## License

MIT
```

**Real example:** Agent Control Platform README got a developer from clone to running app in 8 minutes.

### 2. CONTRIBUTING.md (Workflow Guide)

**Purpose:** How to contribute code without breaking things.

```markdown
# Contributing

## Workflow

1. Create feature branch: \`git checkout -b feat/your-feature\`
2. Make changes + write tests
3. Run \`npm test\` and \`npm run lint\`
4. Commit with descriptive message
5. Push and open PR
6. Wait for review

## Code Standards

- **TypeScript strict mode:** No \`any\`, no implicit returns
- **ESLint:** Fix all warnings before committing
- **Prettier:** Auto-format on save
- **Tests:** Unit tests for business logic, E2E for critical flows

## Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

- \`feat: Add booking cancellation API\`
- \`fix: Prevent double-booking race condition\`
- \`docs: Update deployment guide\`
- \`test: Add E2E test for event creation\`

## Review Process

All PRs are reviewed by the Lead (Ice). Average review time: 1-4 hours.

Scoring system (1-10):
- 9-10: Merge immediately
- 7-8: Merge with post-merge fixes
- <7: Request changes

See [05-development-workflow.md](playbooks/05-development-workflow.md) for details.
```

### 3. Architecture Decision Records (ADRs)

**Purpose:** Preserve *why* decisions were made, not just *what* was decided.

**Location:** `.ai/DECISIONS.md` or `docs/adr/`

**Format:**

```markdown
# ADR-001: Use Next.js App Router Over Pages Router

**Date:** 2026-03-15  
**Status:** Accepted  
**Deciders:** Lan, Ice, Lava

## Context

We need to choose between Next.js App Router (new) and Pages Router (stable).

## Decision

We will use App Router.

## Rationale

**Pros:**
- React Server Components (better performance)
- Streaming and Suspense support
- Improved data fetching patterns
- Better TypeScript support

**Cons:**
- Newer, less mature ecosystem
- Some libraries don't support RSC yet
- Learning curve for team

**Why we chose it anyway:**
- Long-term bet on React's direction
- Performance benefits matter for our use case
- Team is comfortable with learning new tech

## Alternatives Considered

**Pages Router:**
- Pros: Mature, well-documented, stable
- Cons: Legacy API, no RSC, worse performance

**Remix:**
- Pros: Great DX, excellent data loading
- Cons: Smaller ecosystem, team unfamiliar

## Consequences

- Migration from Pages Router to App Router will take 1-2 days
- Some third-party libraries may need workarounds
- Better performance for users
- Easier to hire (App Router is the future)

## References

- [Next.js App Router Docs](https://nextjs.org/docs/app)
- [React Server Components RFC](https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md)
```

**When to create an ADR:**
- Technology choices (framework, database, hosting)
- Architecture patterns (monolith vs microservices)
- Breaking changes (API versioning, schema migrations)
- Controversial decisions (trade-offs were debated)

**When NOT to create an ADR:**
- Obvious choices (using TypeScript in a TypeScript project)
- Temporary hacks (will be refactored soon)
- Implementation details (variable naming)

**Real example from Memory-Weaver v2:**
- **ADR-001:** 4-layer architecture (why not 3 or 5 layers?)
- **ADR-002:** PostgreSQL over MongoDB (why relational?)
- **ADR-003:** Vector search with pgvector (why not dedicated vector DB?)

### 4. API Documentation

**Purpose:** How to use your API without reading code.

**Tools:**

**Option A: OpenAPI (Swagger)**

```typescript
// app/api/events/route.ts
/**
 * @openapi
 * /api/events:
 *   get:
 *     summary: List all events
 *     tags: [Events]
 *     responses:
 *       200:
 *         description: List of events
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/Event'
 */
export async function GET() {
  const events = await prisma.event.findMany();
  return Response.json(events);
}
```

Generate docs: `npm run generate-docs` → `/api/docs`

**Option B: Manual docs**

```markdown
# API Reference

## Events

### List Events

\`GET /api/events\`

**Query Parameters:**
- \`limit\` (optional): Max results (default: 50)
- \`offset\` (optional): Pagination offset

**Response:**
\`\`\`json
{
  "events": [
    {
      "id": "abc123",
      "title": "Summer Festival",
      "date": "2026-07-01T10:00:00Z",
      "capacity": 100
    }
  ],
  "total": 42
}
\`\`\`

### Create Event

\`POST /api/events\`

**Headers:**
- \`Authorization: Bearer <token>\`

**Body:**
\`\`\`json
{
  "title": "New Event",
  "date": "2026-08-15T14:00:00Z",
  "capacity": 50
}
\`\`\`

**Response:**
\`\`\`json
{
  "id": "xyz789",
  "title": "New Event",
  "date": "2026-08-15T14:00:00Z",
  "capacity": 50
}
\`\`\`

**Errors:**
- \`400\`: Invalid request body
- \`401\`: Unauthorized (missing or invalid token)
- \`500\`: Internal server error
\`\`\`
```

**When to use:**
- **OpenAPI:** Public APIs, large teams, auto-generated clients
- **Manual:** Internal APIs, small teams, fast iteration

### 5. .ai/ Context Files (AI Agent Collaboration)

**Purpose:** Guide AI agents working on your project.

**Critical files:**

**`.ai/AGENTS.md`** — Who does what

```markdown
# Team

## Agents
- Ice (@ice) — Architecture, backend, review
- Lava (@lava) — Frontend, testing, implementation

## Human
- Lan — Product owner, final decisions

## Workflow
1. Lan defines what to build
2. Ice creates task specs
3. Lava implements tasks
4. Ice reviews PRs
5. Ice deploys
6. Lan tests
```

**`.ai/ARCHITECTURE.md`** — System overview

```markdown
# Architecture

## Stack
- Next.js 14 (App Router)
- PostgreSQL + Prisma ORM
- Redis (caching, sessions)
- Tailwind CSS + shadcn/ui

## Key Patterns
- Layered architecture (API → Service → Repository)
- Server Components for data fetching
- Client Components for interactivity
- Optimistic UI updates

## Database Schema
See \`prisma/schema.prisma\` for the source of truth.

Key tables:
- \`users\` — Authentication
- \`events\` — Event metadata
- \`bookings\` — Reservations (foreign key to events)

## External Services
- Stripe (payments)
- SendGrid (emails)
- Vercel (hosting)
```

**`.ai/TASKS.md`** — Current work

```markdown
# Tasks

## In Progress
- [ ] Task 007: Email notifications (Lava) — PR #7 open

## Backlog (Prioritized)
- [ ] Task 008: Stripe integration (P0)
- [ ] Task 009: Event analytics dashboard (P1)
- [ ] Task 010: Multi-language support (P2)

## Done ✅
- [x] Task 001: Event CRUD API
- [x] Task 002: Booking creation
- [x] Task 003: Admin dashboard
```

**`.ai/DECISIONS.md`** — ADRs (see above)

**Why `.ai/` matters for agents:**
- Agents read these files FIRST (before touching code)
- Prevents hallucinations (context is explicit)
- Enables autonomous work (no constant questions)
- Preserves team knowledge (even after agents change)

**Real example:** Agent Control Platform `.ai/` files enabled Ice + Lava to work in parallel without conflicts. Ice updated TASKS.md → Lava picked next task → implemented → opened PR → Ice reviewed. Zero context-switching overhead.

### 6. Inline Code Comments

**When to comment:**

```typescript
// ✅ GOOD: Explain WHY, not WHAT
// We use a transaction here because booking and payment must succeed together.
// If payment fails, the booking should not exist.
await prisma.$transaction(async (tx) => {
  const booking = await tx.booking.create({ ... });
  const payment = await tx.payment.create({ ... });
  return { booking, payment };
});

// ✅ GOOD: Warn about gotchas
// WARNING: Prisma Client must be regenerated after schema changes.
// Run: npx prisma generate
const prisma = new PrismaClient();

// ✅ GOOD: Reference external context
// See ADR-003 for why we chose pgvector over Pinecone.
const vector = await db.query('SELECT * FROM memories ORDER BY embedding <-> $1 LIMIT 10', [query]);
```

**When NOT to comment:**

```typescript
// ❌ BAD: Obvious
// Get all users
const users = await prisma.user.findMany();

// ❌ BAD: Outdated
// TODO: Fix this later (written 2 years ago, still not fixed)

// ❌ BAD: Commenting out code
// const oldFunction = () => { ... };
// Delete it. Use git history if you need it back.
```

**The rule:** If the code is self-explanatory, don't comment. If it's surprising or complex, explain WHY.

## Documentation Workflow

### When to Write Docs

**During development (not after):**

```
1. Write task spec (TASKS.md) BEFORE coding
2. Update ARCHITECTURE.md when adding new patterns
3. Create ADR when making controversial decisions
4. Write API docs when creating endpoints
5. Update README when changing setup steps
```

**Anti-pattern:** "We'll document it later" (it never happens).

### Who Writes Docs

**Team responsibility:**

| Doc Type | Owner |
|----------|-------|
| README.md | Lead (Ice) |
| CONTRIBUTING.md | Lead (Ice) |
| ADRs | Whoever made the decision |
| API docs | Developer who built the API |
| .ai/ files | Lead (Ice) |
| Code comments | Developer |

**The rule:** If you write the code, you write the docs.

### Keeping Docs Up-to-Date

**Strategies:**

**1. Doc reviews in PRs**

```markdown
## PR Checklist
- [ ] Code changes
- [ ] Tests added/updated
- [ ] Docs updated (if API changed)
```

**2. Automated checks**

```yaml
# .github/workflows/docs.yml
- name: Check for outdated docs
  run: |
    # Fail if README setup steps don't match package.json scripts
    npm run validate-docs
```

**3. Regular audits**

Every quarter, review:
- Is README accurate? (does `npm run dev` still work?)
- Are ADRs up-to-date? (any decisions we reversed?)
- Is ARCHITECTURE.md current? (did we migrate to a new stack?)

## Real-World Examples

### Event Booking System

**Docs created:**
- README.md (Quick Start)
- .ai/ARCHITECTURE.md (Prisma schema, API design)
- .ai/TASKS.md (13 task specs)
- API.md (endpoint reference)

**Time spent:** 2 hours (upfront), saved 10+ hours (onboarding, questions, debugging).

### Memory-Weaver v2

**Docs created:**
- README.md (4-layer architecture explanation)
- ADR-001 (Why 4 layers?)
- ADR-002 (PostgreSQL vs MongoDB)
- PORTABILITY.md (Export/import guide)
- API.md (Memory operations)

**Result:** Ice and Lava collaborated on complex architecture without confusion. ADRs preserved reasoning for future review.

### Agent Control Platform

**Docs created:**
- README.md (Docker Compose setup)
- CLAWDBOT_INTEGRATION.md (How to integrate AI agents)
- QR_REGISTRATION_SYSTEM.md (Mobile setup flow)
- .ai/AGENTS.md (Team roles)

**Impact:** External contributors onboarded in <30 minutes. AI agents self-registered without human help.

## Documentation Anti-Patterns

### ❌ Over-Documentation

Writing docs for every trivial detail.

```markdown
# ❌ BAD: Excessive detail
## Function: `add(a, b)`
This function takes two numbers, `a` and `b`, and returns their sum.
It uses the `+` operator to perform addition.

# ✅ GOOD: Trust the code
## Math Utilities
See `lib/utils/math.ts` for arithmetic helpers.
```

### ❌ Stale Docs

Docs that contradict the code.

```markdown
# ❌ BAD: Outdated
Run `npm start` to start the server.

# Reality: package.json has `npm run dev`
```

**Solution:** Automate validation or delete outdated docs.

### ❌ Docs Instead of Good Code

Using comments to explain bad code instead of refactoring.

```typescript
// ❌ BAD: Comment explains confusing code
// This loops through the array and filters out null values,
// then maps to extract IDs, and finally removes duplicates.
const ids = [...new Set(arr.filter(x => x).map(x => x.id))];

// ✅ GOOD: Refactor for clarity (no comment needed)
const validItems = arr.filter(item => item !== null);
const ids = [...new Set(validItems.map(item => item.id))];
```

## Tools

### Documentation Generators

**TypeDoc** (TypeScript)
```bash
npm install -D typedoc
npx typedoc --out docs src/
```

**JSDoc** (JavaScript)
```typescript
/**
 * Creates a new booking.
 * @param {string} eventId - The event to book
 * @param {string} userEmail - User's email
 * @returns {Promise<Booking>} The created booking
 */
async function createBooking(eventId, userEmail) { ... }
```

**Docusaurus** (Full documentation sites)
```bash
npx create-docusaurus@latest my-docs classic
```

### Diagramming

**Mermaid** (Markdown diagrams)

```markdown
\`\`\`mermaid
graph TD
  A[User] --> B[API]
  B --> C[Database]
  B --> D[Redis Cache]
\`\`\`
```

**Excalidraw** (Hand-drawn style)
- Great for architecture diagrams
- Exports to PNG/SVG

## Lava's Opinion

> **Documentation is a love letter to your future self.**
> 
> When you return to a project 6 months later, you won't remember why you chose Prisma over TypeORM. You won't remember why events have a `soft_delete` flag. You won't remember the DB migration gotcha.
> 
> ADRs are your time machine. README is your onboarding buddy. `.ai/` files are your agent's brain.
> 
> **The best docs are:**
> 1. **Close to the code** (in the repo, not a wiki)
> 2. **Maintained like code** (reviewed in PRs, versioned with git)
> 3. **Concise** (1 page > 10 pages)
> 
> **Write docs like you're explaining to a smart colleague who just joined.** Not condescending, not assuming too much knowledge, just helpful.
> 
> **Disagreement welcome:** Some teams prefer comprehensive docs everywhere. I prefer minimal docs for obvious code, detailed docs for complex decisions. Both work — just don't skip docs entirely.
