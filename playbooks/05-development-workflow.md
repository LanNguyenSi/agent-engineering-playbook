# 05 — Development Workflow

> The workflow that delivered 13 features in one day.

## Overview

```
Human defines priorities
  → Lead creates task specs
    → Developer implements on branch
      → Developer creates PR
        → Lead reviews (score 1-10)
          → Lead fixes post-merge issues
            → Lead deploys
              → Human tests
```

## Task-Based Development

### Why Tasks, Not Issues

GitHub Issues are for bug reports and feature requests. **Tasks** are self-contained work packages for agents.

A task file lives in the repo (`tasks/` or `.ai/TASKS.md`) and contains everything an agent needs:

```markdown
### Task 001: Booking Cancel API
**Priority:** P0
**Estimate:** 2h
**Status:** Open

**Problem:** Users cannot cancel their bookings.
**Solution:** PATCH /api/bookings/[id] with dual auth.
**Files:** app/api/bookings/[id]/route.ts, components/CancelBookingButton.tsx
**Notes:** Use transaction. Check admin_token key.
```

**Why this works:**
- Agent reads one file, knows exactly what to build
- No context switching between GitHub UI and code
- Priorities are clear (P0 → P1 → P2)
- Status tracking is in the file itself

### Prioritization

| Priority | Meaning | Action |
|----------|---------|--------|
| P0 | Must have for production | Build first |
| P1 | Important for UX | Build after P0 |
| P2 | Nice to have | Build if time allows |

**Real example:** We had 13 tasks. P0 (5 tasks) → P1 (4 tasks) → P2 (4 tasks). All completed in order.

## Branch Strategy

```
main (production)
  └── feat/001-booking-cancel
  └── feat/002-event-soft-delete
  └── feat/013-localization-de
```

**Rules:**
1. `main` is always deployable
2. One branch per task
3. Branch name: `feat/<task-id>-<short-name>`
4. Always pull `main` before creating a new branch
5. PR to `main`, never direct push

**Lesson learned:** Lava branched task 013 before task 002 was merged → lost DeleteEventButton. Fix: Always `git pull origin main` before starting a new task.

## Review Process

### The Score System (1-10)

| Score | Meaning | Action |
|-------|---------|--------|
| 10/10 | Perfect, no changes needed | Merge immediately |
| 9-9.5 | Excellent, minor notes | Merge, fix post-merge if needed |
| 8-8.5 | Good, some issues | Merge with fixes |
| 7-7.5 | Acceptable | Request changes or fix post-merge |
| <7 | Needs work | Request changes, don't merge |

### What to Check

1. **Correctness** — Does it solve the problem from the task spec?
2. **Consistency** — Does it follow existing patterns? (localStorage keys, API response format, etc.)
3. **Safety** — Are DB mutations in transactions? Is auth checked?
4. **Completeness** — Are edge cases handled? (null values, empty states, errors)
5. **Build** — Does it compile? Does the Docker build succeed?

### Post-Merge Fixes

Sometimes it's faster to merge and fix than to request changes:

```
Merge PR → Fix localStorage key → Commit fix → Push → Deploy
```

This worked for us because the Lead (Ice) could fix small issues faster than sending the PR back.

**When NOT to merge-and-fix:**
- Architecture issues (wrong approach)
- Security issues (auth bypass)
- Data integrity issues (missing transaction)

## Deploy Pipeline

```
Code merged to main
  → Pull on server
    → Docker build (--no-cache if schema changed)
      → Docker up -d
        → DB migration if needed
          → Verify (curl health check)
```

**Critical:** After Prisma schema changes:
1. Docker build with `--no-cache` (Prisma Client must regenerate)
2. Run DB migration on production container
3. Verify the app starts without errors

## Velocity Metrics

What we tracked:

| Metric | Target | Actual |
|--------|--------|--------|
| Tasks per day | 5-8 | 13 |
| Average score | 8+ | 9.5 |
| Time per task | 1-2h | 45min avg |
| Deploy failures | <10% | 2 (Prisma, Tailwind) |
| Post-merge fixes | <20% | 3/13 (23%) |

## Ice's Opinion

> The biggest productivity hack: **task specs that an agent can read without asking questions.** Every question from the developer is a round-trip that costs 5-10 minutes. A good task spec has zero round-trips.

> Post-merge fixes are not failures — they're efficiency. Sending a PR back for a one-character fix (`adminToken` → `admin_token`) wastes more time than fixing it yourself in 30 seconds.
