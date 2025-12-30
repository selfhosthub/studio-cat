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

**Note:** Use the official Self-Host Studio template on RunPod. Pre-built images are provided — no need to build your own.

Detailed deployment guide coming soon.

---

## Kubernetes deployment

Kubernetes is the path for production scaling with separate API, workers, web, database, and redis.

**Note:** Detailed Kubernetes deployment guide coming soon.

Key concepts:

* Ingress controller for routing
* TLS termination (cert-manager or load balancer)
* Managed or in-cluster Postgres + Redis
* Persistent volumes for database and workspace files
* Secrets for sensitive configuration

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

### Worker logging configuration

Workers support structured logging with environment variables:

| Variable | Values | Default | Notes |
|----------|--------|---------|-------|
| `LOG_LEVEL` | DEBUG, INFO, WARNING, ERROR | INFO | Controls verbosity |
| `LOG_FORMAT` | pretty, json | pretty | Use `json` for ELK/Prometheus |
| `LOG_PREFIX` | any string | auto-detect | Override container name in logs |
| `LOG_COLORS` | true, false | true | Disable for non-TTY output |

**Pretty format** (development):
```
shs-worker-comfyui (172.17.0.2) | 15:33:19.086 | INFO | Job completed
```

**JSON format** (production/ELK):
```json
{"timestamp":"2024-12-30T15:30:45.123Z","level":"INFO","service":"shs-worker-comfyui","host":"172.17.0.2","message":"Job completed","job_id":"abc123"}
```

For production with log aggregation (ELK, Grafana Loki, etc.):
```bash
LOG_FORMAT=json LOG_LEVEL=INFO
```

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

### Worker-specific variables

| Variable     | Typical default | Notes                                    |
| ------------ | --------------: | ---------------------------------------- |
| `LOG_LEVEL`  |          `INFO` | DEBUG, INFO, WARNING, ERROR              |
| `LOG_FORMAT` |        `pretty` | `json` for production/ELK                |
| `LOG_PREFIX` |     auto-detect | Override container name in log prefix    |
| `LOG_COLORS` |          `true` | Disable for non-TTY environments         |

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
