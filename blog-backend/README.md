# Blog Backend

Nest.js API with PostgreSQL (Prisma), JWT auth, full-text search, analytics, banner ads, and an embedded Vue admin panel.

## Stack

- Nest.js + TypeScript
- PostgreSQL + Prisma ORM
- Redis + BullMQ email queue (optional; in-process fallback when disabled)
- Resend free-tier transactional email
- JWT access/refresh tokens + bcrypt
- Zod validation, Helmet, CORS, throttling, XSS sanitization
- Vue 3 admin SPA (served at `/admin`)

## Docker (recommended)

From the **repository root**:

```bash
cp .env.example .env
docker compose up --build
```

This starts Postgres, this API, and the public frontend. See the root [README](../README.md).

Postgres-only (for local npm dev). Postgres is published on host port **5433** so it does not clash with a local install on 5432:

```bash
docker compose up -d
```

`DATABASE_URL` in `.env` should use `localhost:5433`.

## Local npm development

```bash
cp .env.example .env
# edit DATABASE_URL, JWT secrets, admin credentials

npm install
npx prisma migrate dev
# apply full-text search helpers (tsvector + trigger) — required for /api/articles/search
npm run db:fts
npm run db:seed

npm run dev          # Nest API on :4000
npm run admin:dev    # Admin Vite on :5174 (proxies /api)
# or build admin into the API image/static path:
npm run admin:build
```

Production-style local build:

```bash
npm run build
npm start
```

## Environment variables

| Variable | Description |
|----------|-------------|
| `PORT` | API port (default `4000`) |
| `NODE_ENV` | `development` / `production` |
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_SECRET` | Access token secret |
| `JWT_REFRESH_SECRET` | Refresh token secret |
| `JWT_ACCESS_EXPIRES_IN` | e.g. `15m` |
| `JWT_REFRESH_EXPIRES_IN` | e.g. `7d` |
| `CORS_ORIGIN` | Comma-separated allowed origins |
| `REDIS_URL` / `REDIS_ENABLED` | BullMQ email queue (`true` in Docker Compose) |
| `RESEND_API_KEY` | Free Resend API key ([resend.com](https://resend.com)); empty = console log fallback |
| `EMAIL_FROM` | From address (use Resend onboarding domain until you verify yours) |
| `PUBLIC_SITE_URL` | Frontend base URL for confirm/unsubscribe links |
| `PUBLIC_API_URL` | API base URL for email confirmation redirects |
| `ADMIN_EMAIL` / `ADMIN_PASSWORD` | Seed admin user |
| `RATE_LIMIT_WINDOW_MS` / `RATE_LIMIT_MAX` | Rate limit config |

## Public API

- `GET /api/articles` — paginated published articles
- `GET /api/articles/:slug` — article by EN or RU slug
- `GET /api/articles/search?q=` — PostgreSQL full-text search
- `GET /api/banners?position=` — active banners (random for position)
- `POST /api/banners/:id/click` — track banner click
- `POST /api/analytics/view` — unique views (IP + article, 24h)
- `POST /api/analytics/share` — share by platform
- `GET /api/categories` — categories
- `POST /api/newsletter/subscribe` — double opt-in subscribe (queues confirm email)
- `GET /api/newsletter/confirm?token=` — confirm subscription (redirects to site)
- `POST /api/newsletter/unsubscribe` — unsubscribe by token or email
- `GET /api/newsletter/unsubscribe?token=` — unsubscribe via link

## Default admin credentials

Created by seed (override via `ADMIN_EMAIL` / `ADMIN_PASSWORD`):

| Field | Value |
|-------|-------|
| Email | `admin@example.com` |
| Password | `ChangeMe123!` |

## Admin API (JWT)

- `POST /api/admin/auth/login`
- `POST /api/admin/auth/refresh`
- Articles / banners / categories CRUD
- `GET /api/admin/subscribers` — newsletter subscribers
- `GET /api/admin/analytics/stats|views|shares`
