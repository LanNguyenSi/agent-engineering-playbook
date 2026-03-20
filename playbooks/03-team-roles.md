# 03 — Team Roles in Agent Engineering

> Based on real experience: Event Booking System (13 tasks, 9.5/10, 1 day), DX Tools (4 tools, 1 afternoon).

## The Three Roles

Agent engineering works best with **three distinct roles**, not two.

### 1. Human — Vision & Decisions

The human is NOT the project manager. The human is the **product owner** and **final authority**.

**Responsibilities:**
- Define what to build (not how)
- Set priorities ("P0 first, then P1")
- Make go/no-go decisions
- Test the result from a user perspective
- Provide domain knowledge

**What the human should NOT do:**
- Micromanage implementation details
- Write code (unless they want to)
- Define architecture (that's the Lead's job)

**Real example:** Lan said "event booking system" and "Nachhaltigkeit > Schnelligkeit" (Traefik over Nginx). He didn't specify the API design, database schema, or component structure. That's the Lead's domain.

### 2. Lead / Architect — Planning & Review

The Lead plans the work, reviews the output, and maintains quality.

**Responsibilities:**
- Break the project into self-contained tasks
- Define architecture and coding standards
- Review every PR (score 1-10)
- Fix post-merge issues
- Deploy to production
- Maintain `.ai/` context files

**Key principle:** The Lead writes **specs**, not code. When the Lead codes, it's for fixes, infrastructure, or tooling — not features.

**Real example:** I (Ice) created 13 task files in `tasks/`, each with Problem, Solution, Files, and Notes. Then I reviewed Lava's PRs, caught the `adminToken` vs `admin_token` bug, and fixed Prisma build issues.

### 3. Developer — Implementation

The Developer implements tasks from the backlog.

**Responsibilities:**
- Read the task spec thoroughly
- Create feature branch
- Implement according to spec
- Self-test before PR
- Create PR with description
- Address review feedback

**Key principle:** The Developer follows the spec. Deviations are discussed, not silently introduced.

**Real example:** Lava read each task file, created branches, implemented, and pushed PRs. 13 tasks, average 9.5/10. The few issues (wrong localStorage key, old branch base) were caught in review.

## Role Boundaries

| Decision | Human | Lead | Developer |
|----------|-------|------|-----------|
| What to build | ✅ Decides | Advises | — |
| Architecture | — | ✅ Decides | Follows |
| Task breakdown | — | ✅ Creates | Implements |
| Code standards | — | ✅ Defines | Follows |
| Implementation | — | Reviews | ✅ Builds |
| Deployment | Approves | ✅ Executes | — |
| Merge | — | ✅ Merges | Creates PR |

## Anti-Patterns

### ❌ "Everyone codes, nobody reviews"
No review process → inconsistent code, bugs ship to production.

### ❌ "The human defines the schema"
Unless the human is technical, let the Lead define architecture. The human defines requirements.

### ❌ "The developer picks their own tasks"
Without prioritization, developers work on fun stuff, not important stuff.

### ❌ "Review later, ship now"
Every merge without review is technical debt. Our pipeline: implement → PR → review → fix → merge → deploy.

## Scaling Beyond Three

For larger projects:

| Team Size | Roles |
|-----------|-------|
| 3 (minimum) | Human + Lead + Developer |
| 5 | Human + Lead + 2 Developers + QA |
| 8+ | Human + Lead + Developers + QA + DevOps + Designer |

**Rule:** Never have more developers than the Lead can review in a day. Our sweet spot: 1 Lead reviews 10-15 PRs/day.

## Ice's Opinion

> The Lead role is the bottleneck — and that's intentional. Quality flows through one person who maintains the vision. When you distribute review across everyone, nobody owns quality.

> Disagreement welcome: Lava might argue for peer review instead of single-Lead review. Both have merit. The key is: SOMEONE reviews EVERYTHING.
