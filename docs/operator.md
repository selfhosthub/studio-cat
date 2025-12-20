# Operator Guide

Production deployment and self-hosting guide for Studio.

---

## Table of Contents

- [Deployment Overview](#deployment-overview)
- [RunPod Deployment](#runpod-deployment)
- [Kubernetes Deployment](#kubernetes-deployment)
- [Security Configuration](#security-configuration)
- [Custom Domains](#custom-domains)
- [OAuth & Cloud Storage](#oauth--cloud-storage)
- [Monitoring & Maintenance](#monitoring--maintenance)
- [Compliance](#compliance)
- [Platform Registration Settings](#platform-registration-settings)

---

## Deployment Overview

### Deployment Targets

| Target | Description | Best For |
|--------|-------------|----------|
| **Local Docker** | Docker Compose setup | Development, testing |
| **RunPod** | GPU-enabled single container | GPU workloads, cost-effective |
| **Kubernetes** | Multi-pod architecture | Production, scaling |

### Environment Modes

| Mode | Command | Features |
|------|---------|----------|
| **Production** | `make prod` | Secure, optimized, no debug |
| **Development** | `make dev` | Hot reload, debug, access logs |
| **E2E Testing** | `make e2e` | Isolated database, safe to reset |

### Architecture Components

```
┌─────────────────────────────────────────────────────────────┐
│                      Studio Stack                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│   │   Frontend  │    │   Backend   │    │   Workers   │    │
│   │   Next.js   │◄───┤   FastAPI   │◄───┤   Python    │    │
│   │   :3000     │    │   :8000     │    │             │    │
│   └─────────────┘    └──────┬──────┘    └──────┬──────┘    │
│                             │                   │           │
│                             ▼                   ▼           │
│   ┌─────────────┐    ┌─────────────┐                       │
│   │  PostgreSQL │    │    Redis    │                       │
│   │    :5432    │    │    :6379    │                       │
│   └─────────────┘    └─────────────┘                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## RunPod Deployment

### Overview

RunPod provides GPU-enabled instances with persistent network storage.

**Architecture:**
```
RunPod Pod
├── Container: studio-api (Dockerfile.runpod)
│   ├── FastAPI application (:8000)
│   ├── Embedded PostgreSQL (:5432)
│   └── Redis (:6379)
└── Network Volume: /workspace (persistent)
    └── data/postgresql_data/
```

### Step 1: Build Docker Image

```bash
docker build -f Dockerfile.runpod -t your-registry/studio-api:runpod .
docker push your-registry/studio-api:runpod
```

### Step 2: Create RunPod Secrets

In RunPod dashboard, create secrets:

| Secret Name | Generate With |
|-------------|---------------|
| `postgres_password` | `openssl rand -hex 32` |
| `jwt_secret_key` | `openssl rand -hex 32` |
| `worker_shared_secret` | `openssl rand -hex 32` |
| `runpod_api_key` | Your RunPod API key |

### Step 3: Configure Pod

**Pod Settings:**
- GPU Type: RTX A4000 or higher
- Container Disk: 50 GB
- Network Volume: 100 GB
- Expose Ports: `8000/http`

**Environment Variables:**
```bash
ENV=production
PORT=8000
HOST=0.0.0.0
DEBUG=false
CREATE_TABLES=false

# Using RunPod secrets
POSTGRES_PASSWORD={{ RUNPOD_SECRET_postgres_password }}
DATABASE_URL=postgresql://postgres:{{ RUNPOD_SECRET_postgres_password }}@localhost:5432/selfhost_studio
REDIS_URL=redis://localhost:6379/0
JWT_SECRET_KEY={{ RUNPOD_SECRET_jwt_secret_key }}
WORKER_SHARED_SECRET={{ RUNPOD_SECRET_worker_shared_secret }}
RUNPOD_API_KEY={{ RUNPOD_SECRET_runpod_api_key }}
```

### Step 4: First Boot

Default admin credentials created automatically:
- Email: `admin@localhost`
- Password: `admin`
- **Must change on first login**

### Data Persistence

Network volume persists across pod restarts:
- PostgreSQL data: `/workspace/data/postgresql_data/`
- Uploaded files: `/workspace/`

---

## Kubernetes Deployment

### Architecture

```yaml
Namespace: studio
├── Deployment: studio-api (replicas: 2)
├── Deployment: studio-worker (replicas: 3)
├── Deployment: studio-web (replicas: 2)
├── StatefulSet: postgresql (replicas: 1)
├── StatefulSet: redis (replicas: 1)
├── Service: studio-api (ClusterIP)
├── Service: studio-web (ClusterIP)
├── Ingress: studio (external)
├── ConfigMap: studio-config
├── Secret: studio-secrets
└── PVC: postgresql-data, workspace-data
```

### Prerequisites

- Kubernetes cluster 1.24+
- kubectl configured
- Helm 3 (optional)
- Ingress controller (nginx, traefik)
- Cert-manager (for TLS)

### Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: studio-secrets
  namespace: studio
type: Opaque
stringData:
  DATABASE_URL: postgresql://postgres:PASSWORD@postgresql:5432/selfhost_studio
  REDIS_URL: redis://redis:6379/0
  JWT_SECRET_KEY: <generated-secret>
  WORKER_SHARED_SECRET: <generated-secret>
```

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: studio-config
  namespace: studio
data:
  ENV: production
  DEBUG: "false"
  PORT: "8000"
  HOST: "0.0.0.0"
```

### API Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: studio-api
  namespace: studio
spec:
  replicas: 2
  selector:
    matchLabels:
      app: studio-api
  template:
    spec:
      containers:
      - name: api
        image: your-registry/studio-api:latest
        ports:
        - containerPort: 8000
        envFrom:
        - configMapRef:
            name: studio-config
        - secretRef:
            name: studio-secrets
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Horizontal Pod Autoscaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: studio-api-hpa
  namespace: studio
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: studio-api
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## Security Configuration

### Environment Secrets

**Required secrets (generate with `openssl rand -hex 32`):**

| Variable | Purpose |
|----------|---------|
| `JWT_SECRET_KEY` | JWT token signing |
| `WORKER_SHARED_SECRET` | Worker authentication |
| `POSTGRES_PASSWORD` | Database password |

### TLS/SSL

**Local development:** Not required

**Production:** Configure at ingress/load balancer level

```yaml
# Kubernetes ingress with cert-manager
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

### CORS Configuration

Default: Accepts requests from same origin

Production: Configure allowed origins:
```bash
CORS_ORIGINS=https://studio.yourdomain.com,https://admin.yourdomain.com
```

### Rate Limiting

Configure per-provider rate limits in provider.json:
```json
{
  "config": {
    "rate_limit": {
      "requests_per_minute": 60,
      "concurrent_requests": 5
    }
  }
}
```

### Security Headers

Add via reverse proxy (nginx example):
```nginx
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

---

## Custom Domains

### Overview

Enable branded landing pages via custom domains (e.g., `workflows.mycompany.com`).

### Architecture

```
User Request → Cloudflare Tunnel → Next.js Middleware → BrandingContext → API
```

### Cloudflare Tunnel Setup

1. **Create tunnel** in Cloudflare Zero Trust dashboard
2. **Get tunnel token**
3. **Configure environment:**
```bash
CLOUDFLARE_TUNNEL_TOKEN=<your-tunnel-token>
```

4. **Add DNS records** in Cloudflare:
   - CNAME: `workflows.mycompany.com` → `<tunnel-id>.cfargotunnel.com`

### Organization Configuration

1. In organization settings, set custom domain
2. Middleware detects domain from request headers
3. BrandingContext fetches org-specific branding
4. Landing page renders with custom branding

### Multiple Domains

Single tunnel handles all custom domains:
- Configure each domain in Cloudflare dashboard
- Each org sets their custom domain in settings
- Routing based on Host header

---

## OAuth & Cloud Storage

### Google Drive Integration

**Prerequisites:**
1. Google Cloud Console project
2. OAuth 2.0 credentials
3. Drive API enabled

**Environment variables:**
```bash
GOOGLE_CLIENT_ID=<your-client-id>
GOOGLE_CLIENT_SECRET=<your-client-secret>
```

**OAuth flow:**
1. User clicks "Connect Google Drive"
2. Redirects to Google consent screen
3. User grants access
4. Callback saves encrypted tokens
5. Workflows can access Drive

### GitHub Integration

```bash
GITHUB_CLIENT_ID=<your-client-id>
GITHUB_CLIENT_SECRET=<your-client-secret>
```

### OAuth Security

| Feature | Description |
|---------|-------------|
| CSRF protection | State tokens in Redis |
| Token encryption | Encrypted before storage |
| Organization scoped | Credentials per org |
| Auto-refresh | Tokens refreshed automatically |

---

## Monitoring & Maintenance

### Health Endpoints

| Endpoint | Description |
|----------|-------------|
| `/health` | Basic health check |
| `/api/v1/infrastructure/health` | Detailed health |
| `/api/v1/infrastructure/health/redis` | Redis status |
| `/api/v1/infrastructure/health/workers` | Worker status |
| `/api/v1/infrastructure/health/full` | Full report |

### Infrastructure Dashboard

Super Admin only (`/infrastructure`):
- Redis connection status
- Storage usage statistics
- Worker status (online, busy, last heartbeat)
- Database status

### Logging

**Production logging:**
```bash
DEBUG=false  # Reduces log verbosity
```

**View logs:**
```bash
# Docker
docker logs selfhost-studio-api

# Kubernetes
kubectl logs -f deployment/studio-api -n studio
```

### Backups

**Database backup:**
```bash
# Docker
docker exec selfhost-studio-postgres pg_dump -U postgres selfhost_studio > backup.sql

# Restore
cat backup.sql | docker exec -i selfhost-studio-postgres psql -U postgres selfhost_studio
```

**Make commands:**
```bash
make backup
make restore-database
```

### Scaling Workers

Workers scale horizontally:
```bash
# Docker Compose
docker-compose up -d --scale worker=3

# Kubernetes
kubectl scale deployment/studio-worker --replicas=5 -n studio
```

---

## Compliance

### Data Storage

| Data Type | Location | Encryption |
|-----------|----------|------------|
| Database | PostgreSQL | At rest (optional) |
| Files | Workspace directory | None (add at storage level) |
| Secrets | Database | AES encrypted |
| Credentials | Database | AES encrypted |

### Multi-Tenancy

| Feature | Description |
|---------|-------------|
| Organization isolation | Each org has separate data |
| No cross-org access | Strict boundary enforcement |
| Audit logging | Action tracking per org |

### Data Retention

- Configure backup retention policy
- Implement cleanup jobs for old instances
- Archive vs. delete workflows

### GDPR Considerations

- User data export capability
- Right to deletion (user deactivation)
- Consent tracking for OAuth

---

## Troubleshooting

### Common Issues

**Container won't start:**
```bash
docker logs selfhost-studio-api
# Check for missing environment variables
```

**Database connection failed:**
```bash
# Verify PostgreSQL is running
docker exec selfhost-studio-postgres pg_isready
```

**Redis connection failed:**
```bash
# Verify Redis is running
docker exec selfhost-studio-redis redis-cli ping
```

**Workers not processing:**
```bash
# Check worker status
curl http://localhost:8000/api/v1/infrastructure/health/workers
```

### Performance Tuning

**PostgreSQL:**
```bash
# Increase connections
max_connections = 200

# Increase shared buffers
shared_buffers = 256MB
```

**Redis:**
```bash
# Increase memory
maxmemory 512mb
maxmemory-policy allkeys-lru
```

**API:**
```bash
# Increase workers (Docker)
WORKERS=4 make prod
```

---

## Environment Variables Reference

### Required

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | PostgreSQL connection URL |
| `REDIS_URL` | Redis connection URL |
| `JWT_SECRET_KEY` | JWT signing key |
| `WORKER_SHARED_SECRET` | Worker authentication |

**Missing Required Variables:**

If any required environment variable is missing, the API will fail to start with an error like:

```
ERROR: Missing required environment variable: JWT_SECRET_KEY
```

For production, generate secure values with:
```bash
openssl rand -hex 32
```

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `ENV` | `development` | Environment identifier |
| `DEBUG` | `true` | Debug mode |
| `PORT` | `8000` | API server port |
| `HOST` | `0.0.0.0` | API server host |
| `WORKSPACE_PATH` | `~/workspace` | File storage path |
| `CREATE_TABLES` | `true` | Auto-create tables |
| `CORS_ORIGINS` | `*` | Allowed CORS origins |
| `CLOUDFLARE_TUNNEL_TOKEN` | - | Custom domain tunnel |

---

## Platform Registration Settings

### Organization Approval Workflow

Control how new organizations are handled at signup.

| Setting | Default | Description |
|---------|---------|-------------|
| `auto_activate_new_orgs` | `true` | If true, new orgs are active immediately. If false, they start as `pending_approval` |
| `pending_org_max_users` | `1` | Max users for pending orgs (creator only) |
| `pending_org_max_executions` | `0` | Max workflow executions for pending orgs |
| `pending_org_max_storage_mb` | `50` | Max storage in MB for pending orgs |

### Organization Status States

| Status | Description | Capabilities |
|--------|-------------|--------------|
| `pending_approval` | Awaiting admin approval | Build templates/workflows, limited storage, no executions |
| `active` | Full access | All features based on plan or default limits |
| `suspended` | Restricted access | Read-only, no new resources, no executions |

### Configuring Registration Settings

These settings are stored in the system organization's billing_settings and can be configured via the Super Admin dashboard or API:

```bash
# Via API (super_admin only)
curl -X PATCH "http://localhost:8000/api/v1/organizations/{system_org_id}/billing/settings" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "auto_activate_new_orgs": false,
    "pending_org_max_users": 1,
    "pending_org_max_executions": 0,
    "pending_org_max_storage_mb": 50
  }'
```

### Managing Organization Status

Super admins can manage organization status:

```bash
# Activate a pending organization
curl -X POST "http://localhost:8000/api/v1/organizations/{id}/activate" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"method": "manual"}'

# Suspend an organization
curl -X POST "http://localhost:8000/api/v1/organizations/{id}/suspend" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"reason": "Billing issue"}'

# Set back to pending
curl -X POST "http://localhost:8000/api/v1/organizations/{id}/set-pending" \
  -H "Authorization: Bearer $TOKEN"
```

---

## Next Steps

- **[Admin Guide](/docs/admin)** - Organization management, users, and credentials
- **[User Guide](/docs/user)** - Using workflows, templates, and instances
