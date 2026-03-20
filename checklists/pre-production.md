# Checklist: Pre-Production

## Infrastructure

- [ ] Docker build succeeds with `--no-cache`
- [ ] Traefik configured with service labels
- [ ] DNS A record pointing to server IP
- [ ] SSL certificate auto-generated (Let's Encrypt)
- [ ] HTTP → HTTPS redirect working
- [ ] Health endpoint returns 200

## Database

- [ ] Production database running (PostgreSQL/MySQL)
- [ ] Migrations applied
- [ ] Prisma Client generated with latest schema
- [ ] Seed data created (admin user, test data)
- [ ] Backup strategy defined

## Security

- [ ] Default passwords changed (admin, database)
- [ ] Environment variables set (not .env.example defaults)
- [ ] No secrets in git history
- [ ] `.env` in `.gitignore`
- [ ] Auth working (login, protected routes)
- [ ] CORS configured if needed

## Application

- [ ] All P0 features working
- [ ] Error states handled (404, 500, empty states)
- [ ] Responsive design tested on mobile
- [ ] Core user flow tested end-to-end
- [ ] Email sending configured (if applicable)
- [ ] Logging configured (errors visible in `docker logs`)

## Monitoring

- [ ] Health dashboard deployed (recommended)
- [ ] Container restart policy set (`unless-stopped`)
- [ ] Log rotation configured
- [ ] Alerting for service downtime (optional)

## Documentation

- [ ] README has Quick Start
- [ ] Admin credentials documented (securely)
- [ ] Deployment commands documented
- [ ] Rollback procedure documented
