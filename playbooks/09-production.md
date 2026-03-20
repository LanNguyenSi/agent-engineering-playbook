# 09 — Production Readiness

> From "works on my machine" to "works in production at 3am."

## Deployment Architecture

### Recommended Stack

```
Internet
  └── Traefik (ports 80/443, automatic SSL)
        ├── app.example.com → App Container (port 3000)
        ├── status.example.com → Health Dashboard
        └── api.example.com → API Container (if separate)
```

**Why Traefik:**
- Automatic SSL via Let's Encrypt
- Docker-native (reads labels, no config files)
- Add new services in 30 seconds (just add labels)
- HTTP/2 out of the box

### Docker Multi-Stage Build

```dockerfile
# Stage 1: Dependencies
FROM node:20-slim AS deps
COPY package*.json ./
COPY prisma ./prisma/
RUN npm ci
RUN npx prisma generate

# Stage 2: Build
FROM node:20-slim AS builder
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npx prisma generate  # CRITICAL: COPY . . overwrites generated client
RUN npm run build

# Stage 3: Production
FROM node:20-slim AS runner
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
CMD ["node", "server.js"]
```

**Critical lesson:** `prisma generate` must run in BOTH Stage 1 AND Stage 2. The `COPY . .` in Stage 2 overwrites the generated Prisma Client from Stage 1. We lost 30 minutes debugging this.

### Traefik Labels

```yaml
services:
  app:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.app.rule=Host(`app.example.com`)"
      - "traefik.http.routers.app.entrypoints=websecure"
      - "traefik.http.routers.app.tls.certresolver=letsencrypt"
      - "traefik.http.services.app.loadbalancer.server.port=3000"
      - "traefik.docker.network=traefik"
```

## Database Migrations

### Schema Change Workflow

```bash
# 1. Edit schema
vim prisma/schema.prisma

# 2. Build with --no-cache (forces Prisma Client regeneration)
docker compose build --no-cache app

# 3. Deploy
docker compose up -d app

# 4. Run migration
docker exec db psql -U user -d db -c "ALTER TYPE \"Status\" ADD VALUE IF NOT EXISTS 'NEW_VALUE'"

# 5. Verify
docker logs app --tail 10
```

**Why not `prisma migrate`?** In Docker, `npx prisma` often fails due to permissions. Direct SQL via `psql` is more reliable for enum additions and column changes.

## Health Monitoring

### Minimum Viable Monitoring

Every production service needs:

1. **Health endpoint:** `GET /health` → `{ status: "ok" }`
2. **Container health check:** Docker `HEALTHCHECK` directive
3. **External monitoring:** Status dashboard or uptime checker

### Health Dashboard

We built one: [status.opentriologue.ai](https://status.opentriologue.ai)

Monitors:
- Service availability (TCP port check)
- Container status (Docker API)
- System metrics (CPU, memory)
- Application metrics (active users, messages)

## Common Production Issues

### 1. Next.js + Server Components + Database

**Problem:** Server Components try to access DB at build time.
**Fix:** `export const dynamic = "force-dynamic"` on every page with DB queries.

### 2. Tailwind CSS v4

**Problem:** `@tailwind base; @tailwind components; @tailwind utilities;` doesn't work.
**Fix:** Use `@import "tailwindcss"` (v4 syntax).

### 3. Docker Network Isolation

**Problem:** Container can't reach `localhost` services.
**Fix:** Use `host.docker.internal` or `network_mode: host` for services that need host access.

### 4. SSL Certificate Failures

**Problem:** Let's Encrypt can't issue cert (DNS not propagated, rate limit).
**Fix:** Wait for DNS (`dig domain.com`), disable Traefik dashboard labels if domain doesn't exist.

### 5. Nginx Behind Traefik

**Problem:** Nginx has SSL config but Traefik terminates SSL.
**Fix:** Nginx listens on port 80 only (HTTP), Traefik handles SSL. Create separate `traefik.conf`.

## Production Checklist

Before going live:

- [ ] Docker build succeeds with `--no-cache`
- [ ] Health endpoint returns 200
- [ ] SSL certificate is valid
- [ ] Database migrations applied
- [ ] Environment variables set (not defaults)
- [ ] Error logging configured
- [ ] Admin password changed from default
- [ ] Sensitive files in `.gitignore` (.env, credentials)
- [ ] Responsive design tested on mobile
- [ ] Core user flow tested end-to-end

## Rollback Plan

```bash
# 1. Check what's running
docker ps

# 2. Rollback to previous image
docker compose pull  # pulls latest
# OR: tag images with version before deploying

# 3. Quick fix: restart containers
docker compose restart app

# 4. Nuclear option: redeploy from known-good commit
git checkout <good-commit>
docker compose build --no-cache app
docker compose up -d app
```

## Ice's Opinion

> Production readiness is 80% about deployment infrastructure and 20% about the code. If your deployment is one command (`docker compose up -d`), you can fix bugs in minutes. If deployment is a 15-step manual process, bugs live for days.

> Traefik over Nginx for new projects. Always. The initial setup is 30 minutes longer, but every new service after that is 30 seconds instead of 30 minutes.
