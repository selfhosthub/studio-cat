# Operators Guide

Deploy and run Studio in production (or a serious self-host), with security, backups, monitoring, and upgrades.

If you just want something running: choose **Docker** for simplest, **Kubernetes** for scaling, **RunPod** for single-node GPU.

---

## Common tasks

- Pick a deployment style → [Deployment choices](#deployment-choices)
- Generate required secrets → [Generate secrets](#generate-secrets)
- Put Studio behind TLS → [TLS and reverse proxy](#tls-and-reverse-proxy)
- Set CORS correctly → [CORS](#cors)
- Back up and restore → [Backups](#backups)
- Monitor health → [Health checks](#health-checks)
- Scale workers → [Scaling workers](#scaling-workers)
- Set up custom domains (Cloudflare Tunnel) → [Custom domains](#custom-domains)
- Configure OAuth (Google/GitHub) → [OAuth integrations](#oauth-integrations)
- Troubleshoot common outages → [Troubleshooting](#troubleshooting)

---

## Deployment choices

Studio is a small stack:

- Web UI (Next.js)
- API (FastAPI)
- Workers
- PostgreSQL
- Redis
- Workspace / file storage

Pick the deployment that matches your reality:

### Docker Compose

Best for:
- single server/VPS
- homelab
- most “small team” installs

Fastest time-to-working.

### RunPod (single node + GPU)

Best for:
- GPU workloads
- cost-controlled single node
- minimal ops overhead

Trade-off: often “monolith container” (API + DB + Redis) on one machine. Fine for some installs.

### Kubernetes

Best for:
- production scaling
- HA API + worker pools
- managed databases/redis
- standard platform ops

Trade-off: more moving parts.

---

## Production checklist

If you do only one thing from this guide, do this:

1. Generate secrets (JWT, worker shared secret, DB password)
2. Put Studio behind TLS
3. Change default admin password (if seeded)
4. Configure backups (database + workspace files)
5. Add monitoring/alerts for health endpoints

Everything else is optimization.

---

## Generate secrets

Generate these with a real CSPRNG:

```bash
openssl rand -hex 32
````

Required secrets (typical):

* `JWT_SECRET_KEY` (signs auth tokens)
* `WORKER_SHARED_SECRET` (workers authenticate to API)
* `POSTGRES_PASSWORD` (if you manage Postgres yourself)

Store them in your secret manager:

* Kubernetes Secret
* Docker secrets / env management
* RunPod secrets
* Vault / SSM Parameter Store / etc.

---

## Docker deployment

Use Docker when you want the simplest stable deployment.

### Minimal steps

1. Clone repo / obtain deployment bundle
2. Create a production env file
3. Set required variables:

   * `DATABASE_URL`
   * `REDIS_URL`
   * `JWT_SECRET_KEY`
   * `WORKER_SHARED_SECRET`
4. Start the stack (`make prod` if provided, or `docker compose up -d`)

### What to verify

* Web UI loads
* You can log in
* Creating a workflow works
* Running a workflow creates an instance and completes
* Worker health shows active

---

## RunPod deployment

RunPod is useful when you want a single node with GPU and persistent storage.

### Recommended shape

* One container running API (and optionally embedded Postgres/Redis)
* A persistent network volume mounted at `/workspace`

### Build and push image

```bash
docker build -f Dockerfile.runpod -t your-registry/studio-api:runpod .
docker push your-registry/studio-api:runpod
```

### RunPod secrets

Create secrets in the RunPod dashboard (names are up to you). Typical:

* `postgres_password`
* `jwt_secret_key`
* `worker_shared_secret`
* `runpod_api_key` (only if you need it)

Generate secrets with:

```bash
openssl rand -hex 32
```

### Typical environment variables

```bash
ENV=production
DEBUG=false
HOST=0.0.0.0
PORT=8000

POSTGRES_PASSWORD={{ RUNPOD_SECRET_postgres_password }}
DATABASE_URL=postgresql://postgres:{{ RUNPOD_SECRET_postgres_password }}@localhost:5432/selfhost_studio
REDIS_URL=redis://localhost:6379/0
JWT_SECRET_KEY={{ RUNPOD_SECRET_jwt_secret_key }}
WORKER_SHARED_SECRET={{ RUNPOD_SECRET_worker_shared_secret }}
```

### First boot warning

If the deployment seeds default credentials (example: `admin@localhost / admin`), force a password change immediately.

### Data persistence

If you use a network volume, keep these paths on it:

* PostgreSQL data directory (if embedded)
* workspace files (uploads/artifacts)

---

## Kubernetes deployment

Kubernetes is the “real production” path: separate API, workers, web, DB, redis.

### What you actually need

* Ingress controller (nginx/traefik)
* TLS (cert-manager or external TLS termination)
* Postgres + Redis (managed or in-cluster)
* Persistent volumes for:

  * database (if in-cluster)
  * workspace files

### Secrets and config

Put secrets in a Secret, and non-sensitive config in a ConfigMap.

Secrets commonly include:

* `DATABASE_URL`
* `REDIS_URL`
* `JWT_SECRET_KEY`
* `WORKER_SHARED_SECRET`

### Health probes

Use `/health` for liveness/readiness probes on the API.

---

## TLS and reverse proxy

In production, terminate TLS at:

* Kubernetes ingress, or
* your load balancer, or
* a reverse proxy (nginx/caddy/traefik)

Make sure:

* HTTP → HTTPS redirect
* WebSocket support (Studio uses real-time events)

If live updates don’t work, it’s usually proxy WebSocket settings.

Example Kubernetes ingress TLS (cert-manager):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - studio.yourdomain.com
    secretName: studio-tls
```

---

## CORS

Set CORS to your real domains in production.

```bash
CORS_ORIGINS=https://studio.yourdomain.com,https://admin.yourdomain.com
```

---

## Custom domains

If you support branded landing pages per org, you need a routing layer that forwards requests by Host header.

A common approach is Cloudflare Tunnel.

### Cloudflare Tunnel quick setup

1. Create a tunnel in Cloudflare Zero Trust
2. Get the tunnel token
3. Run the connector (`cloudflared`) with that token
4. Create DNS records (CNAME) pointing your domain to the tunnel

Studio then reads the `Host` header to determine which org branding to load.

---

## OAuth integrations

If you support Google Drive, GitHub, etc., configure OAuth at the operator level.

### Google Drive

* Create a Google Cloud project
* Enable Drive API
* Create OAuth credentials
* Set redirect/callback URL to match your deployment

Environment variables (example):

```bash
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
```

### GitHub

```bash
GITHUB_CLIENT_ID=...
GITHUB_CLIENT_SECRET=...
```

### OAuth security basics

* Use state/CSRF protection
* Encrypt tokens at rest
* Scope tokens to organization
* Rotate client secrets if exposed

---

## Health checks

Useful endpoints (typical):

* `/health` (basic)
* `/api/v1/infrastructure/health` (detailed)
* `/api/v1/infrastructure/health/redis`
* `/api/v1/infrastructure/health/workers`
* `/api/v1/infrastructure/health/full`

Alert on:

* API not healthy
* Redis not reachable
* Worker heartbeats missing

---

## Monitoring and logs

### Logs

Docker:

```bash
docker logs <api-container-name> --tail 200
```

Kubernetes:

```bash
kubectl logs -n studio deployment/studio-api --tail=200
```

Set `DEBUG=false` in production to reduce noise.

### What to monitor

* request latency
* error rate
* queue depth / job backlog
* worker heartbeats
* Postgres connections
* Redis memory/evictions
* storage usage (workspace)

---

## Backups

Back up two things:

1. **PostgreSQL**
2. **Workspace files** (uploads, downloaded artifacts, generated files)

### Database backup

Example (Docker Postgres):

```bash
docker exec <postgres-container> pg_dump -U postgres selfhost_studio > backup.sql
```

Restore:

```bash
cat backup.sql | docker exec -i <postgres-container> psql -U postgres selfhost_studio
```

### Workspace backup

Back up the workspace path you configured (volume, PVC, or filesystem). Exact method depends on your storage (S3, NFS, block storage).

### Backup test

A backup you’ve never restored is not a backup.

Schedule a restore test:

* quarterly for small installs
* monthly for serious production

---

## Scaling workers

Workers scale horizontally.

Docker Compose:

```bash
docker compose up -d --scale worker=3
```

Kubernetes:

```bash
kubectl scale -n studio deployment/studio-worker --replicas=5
```

If instances are stuck pending, scale workers and confirm Redis connectivity.

---

## Upgrades

Treat upgrades like a small deployment:

1. Back up database + workspace
2. Deploy new version
3. Run a small smoke test:

   * login
   * run a simple workflow
   * verify workers process jobs
4. Roll back if needed

---

## Troubleshooting

### API won’t start

Most common causes:

* missing required env vars
* invalid `DATABASE_URL` / `REDIS_URL`
* secrets are empty or incorrectly quoted

Start by checking logs.

### Database connection failed

* confirm Postgres is reachable from the API container/pod
* verify credentials in `DATABASE_URL`
* check DNS/service name (Kubernetes)

### Redis connection failed

* verify Redis is running/reachable
* check `REDIS_URL`
* check firewall/network policies

### Instances stuck on pending

* workers not running
* worker can’t authenticate (wrong `WORKER_SHARED_SECRET`)
* redis queue not reachable

### WebSockets don’t update live UI

* reverse proxy not configured for WebSockets
* TLS termination or headers dropping upgrade requests

---

## Environment variables reference

### Required

| Variable               | What it does                 |
| ---------------------- | ---------------------------- |
| `DATABASE_URL`         | PostgreSQL connection URL    |
| `REDIS_URL`            | Redis connection URL         |
| `JWT_SECRET_KEY`       | Signs auth tokens            |
| `WORKER_SHARED_SECRET` | Auth between API and workers |

Generate secrets with:

```bash
openssl rand -hex 32
```

### Common optional variables

| Variable                  | Typical default | Notes                                           |
| ------------------------- | --------------: | ----------------------------------------------- |
| `ENV`                     |   `development` | Use `production` in prod                        |
| `DEBUG`                   |          `true` | Set `false` in prod                             |
| `PORT`                    |          `8000` | API port                                        |
| `HOST`                    |       `0.0.0.0` | Bind address                                    |
| `WORKSPACE_PATH`          |          varies | Where files live                                |
| `CREATE_TABLES`           |          varies | Disable in production if migrations are managed |
| `CORS_ORIGINS`            |             `*` | Restrict in production                          |
| `CLOUDFLARE_TUNNEL_TOKEN` |               - | For custom domains                              |

---

## Compliance and data handling

At minimum, be able to answer these:

* Where is user data stored?
* How are secrets encrypted?
* How do we delete user data?
* How do we export data for an org?

Typical storage:

* DB (PostgreSQL): orgs, users, workflows, instances, secrets (encrypted)
* Redis: queues, transient state
* Workspace/storage: files and artifacts

If you need GDPR-style behavior, document:

* deactivation/offboarding
* data export
* retention policy
* deletion policy

---

## Next steps

* **Getting Started**: initial setup
* **Admins Guide**: org setup, users, credentials, billing, limits
* **API Reference**: automation from external apps
