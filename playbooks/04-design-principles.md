# 04 — Design Principles for Agent Engineering

> Principles that matter in practice, not just in textbooks.

## Core Principles

### 1. Spec First, Code Second

Never start coding without a written spec. A task file with Problem, Solution, Files, and Notes saves more time than it takes to write.

**Why:** Agents (and humans) make better decisions with context. A 20-line spec prevents a 200-line rewrite.

**How:**
```markdown
### Task 001: Booking Cancel API
**Problem:** Users cannot cancel bookings.
**Solution:** PATCH /api/bookings/[id] with admin + public auth.
**Files:** app/api/bookings/[id]/route.ts, components/CancelBookingButton.tsx
**Notes:** Use transaction for slot restoration. Check existing patterns for localStorage key.
```

### 2. Convention Over Configuration

Agree on patterns once, follow them everywhere.

| Convention | Example |
|-----------|---------|
| File naming | `kebab-case.ts` for files, `PascalCase` for components |
| API responses | `{ data: T }` for success, `{ error: string }` for errors |
| Auth header | `Authorization: Bearer <token>` |
| DB mutations | Always in `$transaction()` when affecting related data |
| State storage | `localStorage.getItem('admin_token')` — one key, everywhere |

**Real lesson:** Lava used `adminToken` in one component while the rest used `admin_token`. Caught in review, but a convention doc would have prevented it.

### 3. Composition Over Complexity

Build small, focused pieces that compose.

**Good:**
```
BookingForm (client) → calls API → API uses transaction → sends email async
```

**Bad:**
```
BookingPage (does everything: form, validation, API call, email, slot update)
```

### 4. Fail Gracefully

Every external operation (email, API, file) should fail without breaking the core flow.

```typescript
// Good: Email failure doesn't break booking
sendBookingConfirmation(booking).catch(error => {
  console.error('Failed to send email:', error);
});

// Bad: Email failure cancels the booking
await sendBookingConfirmation(booking); // throws → booking rolled back
```

### 5. Mobile First, Desktop Enhanced

Design for mobile, enhance for desktop. Not the other way around.

```tsx
// Mobile: stack, Desktop: row
<div className="flex flex-col sm:flex-row">

// Mobile: cards, Desktop: table
<div className="md:hidden">  {/* Mobile cards */}
<div className="hidden md:block">  {/* Desktop table */}
```

### 6. Type Everything

TypeScript strict mode. No `any`. Ever.

If you think you need `any`, you need a better type definition.

```typescript
// Bad
const metadata = booking.metadata as any;

// Good
interface BookingMetadata {
  company?: string;
  role?: string;
}
const metadata = booking.metadata as BookingMetadata | null;
```

### 7. Document Decisions, Not Code

Good code is self-documenting. But WHY you chose an approach is not obvious from code.

**Document:**
- Why Traefik over Nginx (sustainability over speed)
- Why `force-dynamic` on DB pages (Next.js Server Components constraint)
- Why JWT over sessions (stateless, works with multiple services)

**Don't document:**
```typescript
// Increment the counter  ← useless
counter++;
```

## Principles We Debated

### Speed vs. Quality

**Lan's principle:** "Nachhaltigkeit > Schnelligkeit" (Sustainability over speed)

**Ice's interpretation:** Do it right the first time. Traefik setup took 45 min instead of 5 min Nginx hack, but now adding new services takes 30 seconds.

**Lava's interpretation:** Ship fast, iterate. Get the MVP out, then polish. 13 tasks in one day proves speed and quality aren't mutually exclusive.

**Resolution:** Both are right. Speed for features (Lava), sustainability for infrastructure (Ice).

### Single Reviewer vs. Peer Review

**Ice:** Single Lead reviewer maintains consistency. One person owns quality.

**Counter-argument:** Single point of failure. If the Lead is wrong, nobody catches it.

**Our solution:** Lead reviews all PRs, but the Human spot-checks the result (Lan tested on mobile → found responsive issues).

## Ice's Opinion

> The most underrated principle: **Read existing code before writing new code.** Half of Lava's bugs (localStorage key, branch base) would have been caught by looking at how existing code does it. The codebase IS the documentation.
