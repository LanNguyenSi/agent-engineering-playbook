# 07 — Quality Assurance

> Quality is not a phase. It's built into every step.

## Quality Gates

### Gate 1: Task Spec (before coding)
- [ ] Problem is clearly defined
- [ ] Solution approach is specified
- [ ] Files to modify are listed
- [ ] Edge cases are noted

### Gate 2: Self-Review (before PR)
- [ ] Code compiles (`npm run build`)
- [ ] No TypeScript errors
- [ ] Follows existing patterns
- [ ] Tested locally (curl or browser)

### Gate 3: Lead Review (before merge)
- [ ] Score ≥ 7/10
- [ ] No security issues
- [ ] DB mutations in transactions
- [ ] Consistent with codebase

### Gate 4: Production Verify (after deploy)
- [ ] Health check passes
- [ ] Core feature works
- [ ] No error logs

## TypeScript as Quality Tool

TypeScript strict mode is not optional. It catches bugs before they reach review.

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

**Real impact:** TypeScript caught `startTime: null` → `new Date(null).toISOString()` crash before it reached production. A `string | null` type would have flagged it at compile time.

## Common Quality Issues (from our experience)

### 1. Inconsistent Naming
**Problem:** `adminToken` vs `admin_token` in localStorage.
**Prevention:** Document conventions in `.ai/AGENTS.md`.

### 2. Stale Branches
**Problem:** Developer branches off old `main`, misses recent changes.
**Prevention:** Always `git pull origin main` before branching.

### 3. Prisma Client Out of Sync
**Problem:** Schema changes but Prisma Client not regenerated → build fails.
**Prevention:** `prisma generate` in both deps AND builder Docker stages.

### 4. CSS Framework Syntax
**Problem:** Tailwind v4 uses `@import "tailwindcss"`, not `@tailwind base`.
**Prevention:** Document framework versions and syntax in `.ai/DECISIONS.md`.

### 5. Build-Only Files
**Problem:** `vitest.config.ts` or `__tests__/` included in production build.
**Prevention:** `tsconfig.json` exclude: `["vitest.config.ts", "__tests__"]`.

## Automated Quality Checks

### Pre-Commit (recommended)
```json
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md}": ["prettier --write"]
  }
}
```

### CI Pipeline (recommended)
```yaml
steps:
  - npm ci
  - npm run lint
  - npm run build
  - npm test
```

### DevReview (automated PR review)
```bash
devreview review https://github.com/owner/repo/pull/1 --comment
```

Automated scoring: Code Quality (30%), Architecture (25%), Testing (20%), Documentation (15%), Best Practices (10%).

## Quality Metrics

| Metric | Target | How to Measure |
|--------|--------|---------------|
| Build success rate | >95% | CI pipeline |
| Review score average | >8/10 | DevReview or Lead scores |
| Post-merge fix rate | <25% | Fixes after merge / total merges |
| Production incidents | <1/week | Monitoring alerts |
| Type coverage | 100% (no `any`) | TypeScript strict mode |

## Ice's Opinion

> Quality comes from three things: **good specs** (prevents wrong implementations), **consistent patterns** (prevents reinventing), and **fast reviews** (prevents context loss). Automated tools help but don't replace a Lead who understands the codebase.

> The most expensive bug is the one that ships to production and stays there for weeks because nobody noticed. Deploy-and-verify on every merge catches these early.
