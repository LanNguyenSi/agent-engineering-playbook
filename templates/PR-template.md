# Pull Request Template

```markdown
## Task: [Task ID] — [Title]

### What
Brief description of what this PR implements.

### Changes
- `path/to/file.ts` — What changed and why
- `path/to/new-file.ts` — NEW: Purpose

### Testing
How to test:
```bash
# API test
curl -X POST http://localhost:3000/api/endpoint -d '{"key":"value"}'

# Browser test
# Navigate to /page and verify X
```

### Checklist
- [ ] Build passes (`npm run build`)
- [ ] TypeScript clean (no `any`)
- [ ] Follows existing patterns
- [ ] Responsive (mobile + desktop)
- [ ] German UI strings (if applicable)

### Screenshots (if UI change)
Before | After
```
