# DeployIt

A self-hosted deployment platform. Connect a GitHub repo, click deploy, get a live URL — running on your own infrastructure.

## Features

- **GitHub OAuth** — sign in and import repos directly
- **One-click deploys** — push a repo, DeployIt builds and runs it as a Docker container with a `*.your-domain` subdomain
- **Auto-sleep** — idle containers sleep automatically and wake on the next request
- **Neon database provisioning** — spin up a Postgres database from the dashboard
- **Built-in SQL editor** — query provisioned databases without leaving the UI
- **Env var management** — set and update environment variables per project
- **Real-time logs** — build and runtime logs streamed in the dashboard

## Stack

| Layer | Tech |
|---|---|
| Frontend | React + Vite + Tailwind |
| API | Node.js + Express + TypeScript |
| Worker | Node.js + TypeScript (builds & manages containers) |
| Database | PostgreSQL + Drizzle ORM |
| Proxy | Caddy (subdomain routing + automatic SSL) |
| Infra | AWS EC2 + Terraform |

## Monorepo structure

```
apps/
  api/        Express API server
  web/        React frontend
  worker/     Deployment worker (Docker builds, Caddy config)
packages/
  db/         Drizzle schema + migrations
infra/
  caddy/      Caddyfile configs
  terraform/  AWS provisioning
```

## Local development

**Prerequisites:** Node 20+, pnpm, Docker

```bash
# Install dependencies
pnpm install

# Start Postgres + Caddy
docker compose up -d

# Run migrations
pnpm --filter @deployit/db run migrate

# Start all services in parallel
pnpm dev
```

- API: `http://localhost:4000`
- Web: `http://localhost:5173`

Copy `.env.example` to `.env` in `apps/api/` and fill in the values (GitHub OAuth app, session secret, etc.).

## Self-hosting on AWS (free tier)

See [DEPLOY.md](DEPLOY.md) for the full guide — provisions a t2.micro EC2 with Terraform, configures Cloudflare DNS, and boots the stack with Docker Compose. Runs for ~$0/month for the first 12 months.

## Environment variables

| Variable | Description |
|---|---|
| `GITHUB_CLIENT_ID` | GitHub OAuth app client ID |
| `GITHUB_CLIENT_SECRET` | GitHub OAuth app client secret |
| `SESSION_SECRET` | Random 32-byte hex string |
| `ENCRYPTION_KEY` | Random 32-byte hex string (encrypts env vars at rest) |
| `DATABASE_URL` | Postgres connection string |
| `NEON_API_KEY` | Neon API key (for database provisioning) |
| `DOMAIN` | Your base domain (e.g. `deployit.yourdomain.com`) |

Generate secrets with `openssl rand -hex 32`.
