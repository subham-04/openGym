# openGym — Subham's fork

Fork of [DuarteSantos8/openGym](https://github.com/DuarteSantos8/openGym).

## Instance configuration

- Invite-only signup (`INVITE_ONLY=1`)
- Guest mode disabled (`ALLOW_GUEST=0`)
- Password login enabled (`PASSWORD_LOGIN=1`)
- Admin user configured via `ADMIN_UIDS`
- Running on port 3001 (8080 was taken)

## Staying up to date with upstream

```bash
# Pull latest changes from the original author
git fetch upstream
git merge upstream/main

# Resolve any conflicts, then push to this fork
git push origin main
```

## Local development

```bash
# Run with Docker (production-like)
docker compose up -d

# Frontend hot reload (requires Node)
cd frontend && npm install && npm run dev
# Then open http://localhost:5173

# API only rebuild
docker compose up -d --build api
```
