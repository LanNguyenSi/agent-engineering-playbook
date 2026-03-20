# 02 — Architecture

> From monoliths to microservices: patterns that scale.

## Core Principles

Good architecture is:
- **Simple** — Solve today's problems, not future maybes
- **Modular** — Change one part without breaking others
- **Testable** — Easy to verify correctness
- **Observable** — Easy to debug in production

**Anti-pattern:** Over-engineering. You don't need microservices for 100 users.

## Architecture Patterns

### Layered Architecture

Most web apps follow this structure:

```
┌─────────────────────────────────┐
│  Presentation Layer             │  ← UI components, pages
│  (React, Next.js, HTML)         │
├─────────────────────────────────┤
│  API Layer                      │  ← Route handlers, controllers
│  (Next.js API routes, Express)  │
├─────────────────────────────────┤
│  Business Logic Layer           │  ← Core domain logic
│  (Services, use cases)          │
├─────────────────────────────────┤
│  Data Access Layer              │  ← Database queries
│  (Prisma, repositories)         │
├─────────────────────────────────┤
│  Database                       │  ← PostgreSQL, MongoDB
│  (Persistent storage)           │
└─────────────────────────────────┘
```

**Why this works:**
- Clear separation of concerns
- Easy to test (mock each layer)
- Easy to swap implementations (change DB without changing business logic)

**Real example from Agent Control Platform:**

```typescript
// ❌ BAD: Everything in the API route
export async function POST(req: Request) {
  const data = await req.json();
  const user = await prisma.user.findUnique({ where: { email: data.email } });
  if (!user) return Response.json({ error: 'Not found' }, { status: 404 });
  // ... business logic, validation, DB updates all mixed together
}

// ✅ GOOD: Layered approach
// API Layer (app/api/users/route.ts)
export async function POST(req: Request) {
  const data = await req.json();
  const result = await userService.createUser(data);
  return Response.json(result);
}

// Business Logic Layer (lib/services/userService.ts)
export const userService = {
  async createUser(data: CreateUserInput) {
    // Validation
    if (!data.email) throw new Error('Email required');
    
    // Business logic
    const hashedPassword = await hash(data.password);
    
    // Delegate to data layer
    return userRepository.create({
      ...data,
      password: hashedPassword,
    });
  }
};

// Data Access Layer (lib/repositories/userRepository.ts)
export const userRepository = {
  async create(data: User) {
    return prisma.user.create({ data });
  },
  async findByEmail(email: string) {
    return prisma.user.findUnique({ where: { email } });
  }
};
```

**Benefits:**
- API routes stay thin (routing only)
- Business logic is testable without HTTP
- Database queries are centralized
- Easy to mock for testing

### Domain-Driven Design (DDD) Lite

For complex projects, organize by **domain**, not layer:

```
src/
├── domains/
│   ├── users/
│   │   ├── user.service.ts       # Business logic
│   │   ├── user.repository.ts    # Data access
│   │   ├── user.types.ts         # TypeScript types
│   │   └── user.test.ts          # Tests
│   ├── events/
│   │   ├── event.service.ts
│   │   ├── event.repository.ts
│   │   ├── event.types.ts
│   │   └── event.test.ts
│   └── bookings/
│       └── ...
└── app/                          # UI + API routes
```

**When to use:**
- Multiple bounded contexts (users, events, bookings)
- Large teams (each team owns a domain)
- Complex business logic (event sourcing, CQRS)

**When NOT to use:**
- Simple CRUD apps
- Small teams (<5 people)
- MVPs or prototypes

### Monolith vs Microservices

**Monolith: One codebase, one deployment**

```
┌─────────────────────────────────┐
│                                 │
│   Single Application            │
│   (Next.js, Express, etc.)      │
│                                 │
│   ┌──────┐ ┌──────┐ ┌──────┐  │
│   │Users │ │Events│ │Billing│  │
│   └──────┘ └──────┘ └──────┘  │
│                                 │
└─────────────────────────────────┘
           │
           ▼
    Single Database
```

**Pros:**
- ✅ Simple deployment (one Docker container)
- ✅ No network calls (everything is local)
- ✅ Easy to develop and debug
- ✅ Transactions work naturally

**Cons:**
- ❌ Tight coupling (change one thing, deploy everything)
- ❌ Scaling is all-or-nothing (can't scale just billing)
- ❌ One bug can bring down the whole system

**Microservices: Multiple services, multiple deployments**

```
┌──────────┐   ┌──────────┐   ┌──────────┐
│  Users   │   │  Events  │   │ Billing  │
│ Service  │   │ Service  │   │ Service  │
└────┬─────┘   └────┬─────┘   └────┬─────┘
     │              │              │
     ▼              ▼              ▼
  Users DB      Events DB      Billing DB
```

**Pros:**
- ✅ Independent deployment (update billing without touching users)
- ✅ Independent scaling (scale billing 10x during Black Friday)
- ✅ Tech diversity (users in Node, events in Go, billing in Python)
- ✅ Isolation (bug in billing doesn't crash users)

**Cons:**
- ❌ Complex deployment (orchestration, service discovery)
- ❌ Network overhead (service-to-service calls)
- ❌ Distributed transactions are HARD
- ❌ Debugging spans multiple services

**When to use microservices:**
- High traffic with different scaling needs
- Large teams (>20 developers)
- Need for independent deployment cycles
- Different tech requirements per service

**When to use monolith:**
- Startups and MVPs (ship fast)
- Small to medium teams (<20 people)
- Moderate traffic (<10k req/sec)
- Simple business logic

**Real example:** Agent Control Platform started as a monolith (Next.js + Prisma + PostgreSQL). If we hit 100k agents, we'd split into:
- **Agent Registry Service** (high read, low write)
- **Task Queue Service** (high write, needs horizontal scaling)
- **Dashboard Service** (user-facing, needs caching)

But today? Monolith is perfect.

## API Design

### REST vs GraphQL vs tRPC

**REST:** Endpoints per resource

```typescript
GET    /api/users          # List users
GET    /api/users/123      # Get user
POST   /api/users          # Create user
PATCH  /api/users/123      # Update user
DELETE /api/users/123      # Delete user
```

**Pros:** Simple, cacheable, widely supported
**Cons:** Over-fetching, under-fetching, versioning

**GraphQL:** Query language for APIs

```graphql
query {
  user(id: "123") {
    name
    email
    posts {
      title
    }
  }
}
```

**Pros:** Exact data needed, single endpoint, self-documenting
**Cons:** Caching is harder, N+1 query problem, complexity

**tRPC:** Type-safe RPC for TypeScript

```typescript
// Server
const appRouter = router({
  getUser: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(({ input }) => db.user.findUnique({ where: { id: input.id } })),
});

// Client (type-safe!)
const user = await trpc.getUser.query({ id: '123' });
```

**Pros:** End-to-end type safety, no code generation, fast
**Cons:** TypeScript only, couples frontend/backend, no public API

**When to use:**
- **REST:** Public APIs, simple CRUD, mobile apps
- **GraphQL:** Complex data relationships, multiple clients
- **tRPC:** Full-stack TypeScript, internal APIs only

## Database Design

### Relational (PostgreSQL) vs Document (MongoDB)

**PostgreSQL (relational):**

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255)
);

CREATE TABLE posts (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  title VARCHAR(255) NOT NULL,
  content TEXT
);
```

**When to use:**
- ✅ Strong relationships (users → posts → comments)
- ✅ Complex queries (JOINs, aggregations)
- ✅ ACID transactions (money, bookings)
- ✅ Data integrity (foreign keys, constraints)

**MongoDB (document):**

```javascript
// User document
{
  _id: ObjectId("..."),
  email: "user@example.com",
  name: "Alice",
  posts: [
    { title: "Post 1", content: "..." },
    { title: "Post 2", content: "..." }
  ]
}
```

**When to use:**
- ✅ Flexible schemas (user profiles with varying fields)
- ✅ Denormalized data (embed instead of JOIN)
- ✅ High write throughput (logs, analytics)
- ✅ Rapid prototyping (schema changes are easy)

**Real example:** Memory-Weaver v2 uses PostgreSQL because:
- Strong relationships (memories → agents, memories → tags)
- Complex queries (semantic search with vector similarity)
- ACID transactions (layer transitions must be atomic)

### Caching Strategy

**Layers:**

```
Request → Cache (Redis) → Database (PostgreSQL)
            ↑ hit (fast)      ↑ miss (slow)
```

**Cache what?**
- Expensive queries (JOINs, aggregations)
- Frequently accessed data (user profiles, config)
- Computed results (dashboard stats, leaderboards)

**Don't cache:**
- Critical data (balance, inventory)
- Rapidly changing data (live stock prices)
- Personalized data (unless keyed by user)

**Real example from Agent Control Platform:**

```typescript
// ❌ BAD: Query DB every time
export async function GET() {
  const agents = await prisma.agent.findMany();
  return Response.json(agents);
}

// ✅ GOOD: Cache for 5 minutes
export async function GET() {
  const cached = await redis.get('agents:list');
  if (cached) return Response.json(JSON.parse(cached));
  
  const agents = await prisma.agent.findMany();
  await redis.setex('agents:list', 300, JSON.stringify(agents));
  return Response.json(agents);
}
```

## Real-World Architecture Examples

### Event Booking System (Monolith)

```
Next.js App
├── app/
│   ├── page.tsx                  # Event listing
│   └── api/
│       ├── events/route.ts       # CRUD events
│       └── bookings/route.ts     # Create/cancel bookings
├── lib/
│   ├── services/
│   │   ├── eventService.ts       # Business logic
│   │   └── bookingService.ts
│   └── repositories/
│       ├── eventRepo.ts          # Data access
│       └── bookingRepo.ts
└── prisma/
    └── schema.prisma             # DB schema
```

**Why monolith:** Small team, moderate traffic, tight deadlines.

### Memory-Weaver v2 (4-Layer Architecture)

```
Layer 1: IdentityCore      (Permanent, 0.01ms access)
Layer 2: ActiveContext     (90-day retention)
Layer 3: SemanticArchive   (Searchable, 90-365 days)
Layer 4: DeepArchive       (Compressed, >365 days)
```

**Why layered:** Different access patterns, different performance requirements.

### Agent Control Platform (Modular Monolith)

```
Next.js + Express + TimescaleDB + Redis
├── Frontend (Next.js)        # Dashboard
├── Backend (Hono API)        # Agent management
├── TimescaleDB               # Time-series data (metrics)
├── Redis                     # Real-time updates (SSE)
└── Traefik                   # Reverse proxy
```

**Why modular monolith:** Single deployment, clear boundaries, easy to split later.

## Lava's Opinion

> **Start with a monolith. Always.**
> 
> Microservices are a solution to scaling problems. If you don't have scaling problems yet, you're adding complexity for no benefit.
> 
> We built Agent Control Platform as a monolith in 2 days. If we started with microservices, it would've taken 2 weeks. Ship first, scale later.
> 
> **When to split into microservices?** When you have concrete evidence that the monolith can't handle the load. Not before.
> 
> **Disagreement welcome:** Some teams prefer microservices from day one for organizational reasons (team autonomy, independent deployments). Both approaches work — just be intentional about the tradeoffs.
