# Checklist: Code Review

## Quick Review (< 5 min)

- [ ] PR description explains what and why
- [ ] Branch is based on latest `main`
- [ ] Build passes (no TypeScript errors)
- [ ] No `any` types
- [ ] Follows existing naming conventions
- [ ] No hardcoded secrets or credentials

## Standard Review (5-15 min)

All of Quick Review, plus:

- [ ] DB mutations use transactions where needed
- [ ] Error handling present (try/catch, graceful failures)
- [ ] Auth checks on protected endpoints
- [ ] Consistent with existing patterns (localStorage keys, API responses)
- [ ] Edge cases handled (null, empty, error states)
- [ ] Responsive design (mobile + desktop)
- [ ] German strings for UI (if applicable)

## Deep Review (15-30 min)

All of Standard Review, plus:

- [ ] Architecture makes sense (separation of concerns)
- [ ] No N+1 queries
- [ ] Performance considerations (large datasets, pagination)
- [ ] Security review (XSS, injection, auth bypass)
- [ ] Tests added for new functionality
- [ ] Documentation updated (README, API docs)

## Scoring Guide

| Score | Criteria |
|-------|----------|
| 10 | Perfect. No changes. Ship it. |
| 9-9.5 | Excellent. Minor notes, merge immediately. |
| 8-8.5 | Good. Small fixes needed, can fix post-merge. |
| 7-7.5 | Acceptable. Issues found but approach is correct. |
| <7 | Needs work. Wrong approach or missing critical pieces. |
