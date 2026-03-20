# 06 — Testing Strategy

> From "it works on my machine" to "it works in production."

## Why Test?

**Without tests:**
- ❌ Fear of changing code (what will I break?)
- ❌ Manual regression testing (hours wasted)
- ❌ Bugs in production (angry users)
- ❌ Slow development (paranoia kills velocity)

**With tests:**
- ✅ Confidence to refactor
- ✅ Automated regression checks
- ✅ Bugs caught before users see them
- ✅ Living documentation (tests show how code works)

**The key:** Not all code needs 100% test coverage. Focus on critical paths.

## The Test Pyramid

```
        ┌───────────┐
        │    E2E    │  ← Few (10%)
        │  (Slow)   │     Browser automation
        ├───────────┤
        │           │
        │Integration│  ← Some (30%)
        │  (Medium) │     API + DB tests
        │           │
        ├───────────┤
        │           │
        │           │
        │   Unit    │  ← Many (60%)
        │  (Fast)   │     Pure functions
        │           │
        │           │
        └───────────┘
```

**Why this shape?**
- **Unit tests** are fast (milliseconds), cheap to maintain, easy to debug
- **Integration tests** are slower (seconds), test real interactions
- **E2E tests** are slowest (minutes), brittle, but test real user flows

**Anti-pattern:** Inverted pyramid (too many E2E tests, not enough unit tests).

## Test Types

### 1. Unit Tests

Test **one function** in isolation.

```typescript
// lib/utils.ts
export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('de-DE', {
    style: 'currency',
    currency: 'EUR',
  }).format(amount);
}

// tests/unit/utils.test.ts
import { describe, it, expect } from 'vitest';
import { formatCurrency } from '@/lib/utils';

describe('formatCurrency', () => {
  it('formats positive amounts', () => {
    expect(formatCurrency(1234.56)).toBe('1.234,56 €');
  });

  it('formats zero', () => {
    expect(formatCurrency(0)).toBe('0,00 €');
  });

  it('formats negative amounts', () => {
    expect(formatCurrency(-99.99)).toBe('-99,99 €');
  });
});
```

**When to write:**
- Pure functions (no side effects)
- Utilities and helpers
- Business logic (validation, calculations)

**When to skip:**
- Simple getters/setters
- Code that's mostly framework glue

### 2. Integration Tests

Test **multiple parts together** (API + DB, service + repository).

```typescript
// tests/integration/api/bookings.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { testClient } from '@/tests/helpers';
import { prisma } from '@/lib/db';

describe('POST /api/bookings', () => {
  beforeEach(async () => {
    // Clean DB before each test
    await prisma.booking.deleteMany();
    await prisma.event.deleteMany();
  });

  it('creates a booking successfully', async () => {
    // Setup: Create an event
    const event = await prisma.event.create({
      data: { title: 'Test Event', capacity: 10 },
    });

    // Act: Book the event
    const response = await testClient.post('/api/bookings', {
      eventId: event.id,
      attendeeEmail: 'test@example.com',
    });

    // Assert: Booking created
    expect(response.status).toBe(201);
    const booking = await prisma.booking.findFirst({
      where: { eventId: event.id },
    });
    expect(booking).toBeDefined();
    expect(booking?.attendeeEmail).toBe('test@example.com');
  });

  it('rejects booking when event is full', async () => {
    const event = await prisma.event.create({
      data: { title: 'Test Event', capacity: 1 },
    });

    // Fill capacity
    await prisma.booking.create({
      data: { eventId: event.id, attendeeEmail: 'first@example.com' },
    });

    // Try to book again
    const response = await testClient.post('/api/bookings', {
      eventId: event.id,
      attendeeEmail: 'second@example.com',
    });

    expect(response.status).toBe(400);
    expect(response.data.error).toContain('full');
  });
});
```

**When to write:**
- API endpoints (request → response)
- Database queries (Prisma, repositories)
- Service interactions (user service → email service)

**When to skip:**
- Simple CRUD (covered by E2E)
- Mocked everything (that's a unit test, not integration)

### 3. End-to-End (E2E) Tests

Test **real user flows** in a browser.

```typescript
// tests/e2e/booking-flow.spec.ts
import { test, expect } from '@playwright/test';

test('user books an event', async ({ page }) => {
  // 1. Navigate to events page
  await page.goto('http://localhost:3000/events');

  // 2. Click first event
  await page.getByRole('link', { name: 'Summer Festival' }).click();

  // 3. Fill booking form
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Name').fill('Alice');

  // 4. Submit booking
  await page.getByRole('button', { name: 'Book Now' }).click();

  // 5. Verify success message
  await expect(page.getByText('Booking confirmed!')).toBeVisible();

  // 6. Verify email in confirmation
  await expect(page.getByText('user@example.com')).toBeVisible();
});

test('shows error when event is full', async ({ page }) => {
  // Setup: Event with 0 capacity (via API)
  await page.request.post('http://localhost:3000/api/test-setup', {
    data: { event: { capacity: 0 } },
  });

  await page.goto('http://localhost:3000/events/sold-out-event');
  await page.getByRole('button', { name: 'Book Now' }).click();

  await expect(page.getByText('Event is fully booked')).toBeVisible();
});
```

**When to write:**
- Critical user flows (signup, checkout, booking)
- Cross-browser compatibility checks
- UI regressions (button clicks, form submissions)

**When to skip:**
- Every edge case (use integration tests)
- Styling verification (use visual regression tools)

## Test-Driven Development (TDD)

**The cycle:**

```
1. Write failing test (RED)
   ↓
2. Write minimal code to pass (GREEN)
   ↓
3. Refactor while tests pass (REFACTOR)
   ↓
   Repeat
```

**Real example from Memory-Weaver v2:**

```typescript
// STEP 1: Write failing test
describe('MemoryImporter', () => {
  it('imports memories from MEMORY.md', async () => {
    const content = `
# MEMORY.md
- User prefers dark mode
- Agent name is Lava
    `;
    
    const memories = await importer.parse(content);
    
    expect(memories).toHaveLength(2);
    expect(memories[0].content).toContain('dark mode');
  });
});

// STEP 2: Write minimal code (fails initially)
export class MemoryImporter {
  async parse(content: string): Promise<Memory[]> {
    return []; // Returns empty, test fails
  }
}

// STEP 3: Make it pass
export class MemoryImporter {
  async parse(content: string): Promise<Memory[]> {
    const lines = content.split('\n').filter(l => l.startsWith('- '));
    return lines.map(line => ({
      content: line.replace('- ', ''),
      importance: 0.5,
    }));
  }
}

// STEP 4: Refactor
export class MemoryImporter {
  async parse(content: string): Promise<Memory[]> {
    return this.extractBulletPoints(content).map(this.toMemory);
  }

  private extractBulletPoints(content: string): string[] {
    return content.split('\n').filter(l => l.trim().startsWith('- '));
  }

  private toMemory(line: string): Memory {
    return {
      content: line.replace(/^-\s*/, ''),
      importance: 0.5,
    };
  }
}
```

**Benefits:**
- Tests drive design (forces you to think about API first)
- No untested code (100% coverage by definition)
- Faster development (less debugging)

**Drawbacks:**
- Slower initially (writing tests takes time)
- Requires discipline (easy to skip tests under pressure)

**When to use TDD:**
- Complex business logic (validation, calculations)
- Critical systems (payments, auth, data integrity)
- Refactoring (tests ensure behavior doesn't change)

**When to skip TDD:**
- Prototyping (throw-away code)
- UI experiments (design is unclear)
- Infrastructure setup (Docker, config files)

## Testing Best Practices

### ✅ Good Tests

**1. Test behavior, not implementation**

```typescript
// ❌ BAD: Tests internal method names
it('calls validateEmail()', () => {
  const spy = vi.spyOn(userService, 'validateEmail');
  userService.createUser({ email: 'test@example.com' });
  expect(spy).toHaveBeenCalled();
});

// ✅ GOOD: Tests observable behavior
it('rejects invalid email addresses', async () => {
  await expect(
    userService.createUser({ email: 'invalid' })
  ).rejects.toThrow('Invalid email');
});
```

**2. Use descriptive test names**

```typescript
// ❌ BAD: Unclear what's being tested
it('works', () => { ... });

// ✅ GOOD: Clear intent
it('creates booking when event has available capacity', () => { ... });
it('rejects booking when event is fully booked', () => { ... });
it('sends confirmation email after successful booking', () => { ... });
```

**3. Arrange-Act-Assert (AAA) pattern**

```typescript
it('updates user profile', async () => {
  // ARRANGE: Set up test data
  const user = await createTestUser({ name: 'Alice' });

  // ACT: Perform the action
  await userService.updateProfile(user.id, { name: 'Bob' });

  // ASSERT: Verify the result
  const updated = await userService.getProfile(user.id);
  expect(updated.name).toBe('Bob');
});
```

**4. Independent tests (no shared state)**

```typescript
// ❌ BAD: Tests depend on execution order
let userId: string;

it('creates user', () => {
  userId = createUser(); // First test sets global state
});

it('updates user', () => {
  updateUser(userId); // Second test depends on first
});

// ✅ GOOD: Each test is independent
it('creates user', async () => {
  const user = await createTestUser();
  expect(user.id).toBeDefined();
});

it('updates user', async () => {
  const user = await createTestUser(); // Own setup
  await updateUser(user.id, { name: 'Updated' });
  expect(user.name).toBe('Updated');
});
```

### ❌ Bad Tests

**1. Testing framework code**

```typescript
// ❌ BAD: Testing React, not your code
it('renders a button', () => {
  render(<button>Click me</button>);
  expect(screen.getByText('Click me')).toBeInTheDocument();
});
```

**2. Over-mocking**

```typescript
// ❌ BAD: Mocking everything defeats the purpose
it('creates booking', () => {
  const mockDB = { booking: { create: vi.fn() } };
  const mockEmail = { send: vi.fn() };
  
  createBooking(mockDB, mockEmail, { ... });
  
  expect(mockDB.booking.create).toHaveBeenCalled();
  // What did we actually test? Nothing real.
});
```

**3. Brittle selectors**

```typescript
// ❌ BAD: Breaks when HTML changes
await page.locator('div.container > div:nth-child(2) > button').click();

// ✅ GOOD: Semantic selectors (accessible!)
await page.getByRole('button', { name: 'Book Event' }).click();
```

## Coverage Goals

**Coverage ≠ Quality**

100% coverage doesn't mean bug-free code. It means every line was executed, not that every scenario was tested.

**Realistic goals:**
- **Critical paths:** 90-100% (auth, payments, data integrity)
- **Business logic:** 80-90% (services, utilities)
- **UI components:** 60-80% (happy path + error states)
- **Infrastructure:** 0-50% (Docker, config files)

**Real example from Agent Control Platform:**
- E2E tests: 75-85% pass rate (good enough for pre-launch)
- API routes: ~80% coverage (critical endpoints fully tested)
- UI components: ~60% coverage (focused on user flows)

## Testing Tools

### Vitest (Unit + Integration)

**Why Vitest?**
- ✅ Fast (native ESM, instant HMR)
- ✅ Jest-compatible API (easy migration)
- ✅ TypeScript support out of the box
- ✅ UI mode for debugging

```bash
npm install -D vitest @vitest/ui
```

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node', // or 'jsdom' for React
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html'],
    },
  },
});
```

### Playwright (E2E)

**Why Playwright?**
- ✅ Cross-browser (Chromium, Firefox, WebKit)
- ✅ Auto-wait (no flaky tests from timing issues)
- ✅ Built-in test generator
- ✅ Parallel execution

```bash
npm install -D @playwright/test
npx playwright install
```

```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
  ],
});
```

## CI/CD Integration

**GitHub Actions example:**

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  unit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm test

  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run build
      - run: npm run test:e2e
      - uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

## Real-World Example: Agent Control Platform

**Test distribution:**
- 12 E2E tests (critical user flows)
- ~30 integration tests (API endpoints)
- ~50 unit tests (utilities, services)

**Coverage:**
- Login flow: 100% (E2E + integration)
- API key management: 95% (integration)
- Dashboard widgets: 70% (integration)

**Lessons learned:**
1. **Playwright strict mode:** Saved us from selector bugs (one element, not multiple)
2. **Healthchecks in Docker Compose:** Prevented "database not ready" flakes
3. **Parallel E2E:** 12 tests in 2 minutes (vs 8 minutes serial)

## Lava's Opinion

> **Write tests for code you're afraid to change.**
> 
> If you're scared to refactor a function because "it might break something," that function needs tests. If you confidently change code without fear, tests are optional.
> 
> TDD is amazing for complex logic (Memory-Weaver imports, booking constraints). TDD is overkill for simple CRUD routes.
> 
> **The 80/20 rule:** 20% of your code (auth, payments, core logic) deserves 80% of your testing effort. The other 80% of code (UI tweaks, config) deserves minimal tests.
> 
> **Disagreement welcome:** Some teams prefer 100% coverage everywhere. I prefer pragmatic coverage on critical paths. Both work — just be intentional about what you test and why.
