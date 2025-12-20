# Admins Guide

Manage your organization: team access, secrets/credentials, branding, billing, limits, and webhooks.

If you’re new: **invite a user**, **add provider credentials**, then have them run a simple workflow.

---

## Common tasks

- Invite a teammate → [Invite a user](#invite-a-user)
- Promote someone to admin → [Change a user’s role](#change-a-users-role)
- Remove access quickly → [Deactivate a user](#deactivate-a-user)
- Add an API key / OAuth connection → [Add provider credentials](#add-provider-credentials)
- Store a reusable secret (tokens, URLs) → [Create an organization secret](#create-an-organization-secret)
- Rotate keys safely → [Rotate a secret or credential](#rotate-a-secret-or-credential)
- Set logo + colors → [Branding](#branding)
- Find “who changed what” → [Audit logs](#audit-logs)
- Check limits / usage → [Limits and usage](#limits-and-usage)
- Create an inbound webhook URL → [Webhooks](#webhooks)

---

## Organization overview

Your org settings live at **Organization → Manage**.

- Dashboard: `/organization/manage`
- Settings: `/organization/manage/settings`
- Users: `/organization/manage/users`
- Branding: `/organization/manage/branding`
- Billing: `/organization/manage/billing`
- Limits: `/organization/manage/limits`

### Update organization details

Use this when you want a clean name/slug/logo for your team.

1. Open **Organization → Manage** (`/organization/manage`)
2. Go to **Details**
3. Update:
   - **Name** (display name)
   - **Slug** (URL-safe identifier)
   - **Website** (optional)
   - **Logo** (optional)
4. Save

**Tip:** The slug often appears in webhook URLs and links. Choose it early and avoid changing it later.

### Organization settings

Org settings affect everyone.

- **Show image thumbnails**  
  Turn off on slow networks or when you want to reduce bandwidth.

- **Resource card size**  
  Controls how big file cards appear (grid views across the app).

---

## Team management

### Invite a user

1. Go to **Users** (`/organization/manage/users`)
2. Click **Invite User**
3. Fill in:
   - Email
   - Username
   - Initial password
   - Role (start with **User** unless you have a reason not to)
4. Save

They can log in immediately and start building/running workflows.

### Change a user’s role

Use this when someone needs access to org-level settings or secrets.

1. Go to **Users** (`/organization/manage/users`)
2. Find the user
3. Change the **Role** dropdown
4. Save

Role guide:
- **User**: build and run workflows
- **Admin**: manage org resources, members, and credentials
- **Super Admin**: system-wide access (infrastructure + global settings)

### Deactivate a user

This is the fastest, safest offboarding option.

1. Go to **Users** (`/organization/manage/users`)
2. Toggle **Active** → off
3. Save

Deactivation is usually better than deletion because it preserves history (instances, audit events, ownership).

### Reset a password

1. Go to **Users** (`/organization/manage/users`)
2. Edit the user
3. Set a new password
4. Save

If you’re doing this due to suspected compromise, rotate relevant secrets/credentials too.

---

## Secrets and credentials

Studio stores secrets encrypted and scoped to your organization.

There are two buckets:

- **Organization secrets**: reusable values you reference across workflows (tokens, URLs, shared secrets).
- **Provider credentials**: logins/API keys for external services (OpenAI, Google, Leonardo, etc).

### Create an organization secret

Use this for values you want to reuse in many workflows.

1. Go to **Secrets** (`/secrets`)
2. Open **Organization Secrets**
3. Click **New Secret**
4. Set:
   - **Name** (example: `SLACK_WEBHOOK_URL`, `WEBHOOK_SHARED_SECRET`)
   - **Type** (optional classification)
   - **Description** (helpful later)
   - **Value**
   - **Expiration** (optional)
5. Save

Naming tip: use uppercase + underscores. It’s easier to spot in mapping UIs and configs.

### Add provider credentials

Use this for provider authentication (most common reason workflows fail early).

1. Go to **Secrets** (`/secrets`)
2. Open **Provider Credentials**
3. Choose a provider
4. Enter what it asks for (API key / OAuth / etc.)
5. Save

If a user sees **“No credentials configured”** in a step, this is the fix most of the time.

### Rotate a secret or credential

Do this any time:
- someone leaves the team
- you suspect a leak
- a provider key was shared in plaintext
- a credential is expiring

Safe rotation sequence:

1. Create a **new** key/token in the external provider
2. Update the secret/credential in Studio
3. Run a small test workflow that uses it
4. Revoke the old key in the external provider

### Expiration and hygiene

- Use expiration dates when you can.
- Put real descriptions on secrets (“Used by: invoice webhook workflow”).
- Prefer provider credentials for provider auth (better organization and UI support).

---

## Webhooks

Webhooks let outside systems trigger a workflow.

### Create a webhook

1. Go to **Webhooks** (`/webhooks`)
2. Click **New Webhook**
3. Choose the workflow to trigger
4. Pick an authentication type (see below)
5. Save

Studio will show:
- A full URL you can copy
- A secret or token (depending on auth type)
- Controls to regenerate or disable

### Webhook URL format

```

/webhooks/incoming/{org_slug}/{webhook_slug}

```

Example:

```

[https://studio.example.com/webhooks/incoming/acme-corp/process-orders](https://studio.example.com/webhooks/incoming/acme-corp/process-orders)

```

### Webhook authentication

Choose one:

- **HMAC_SHA256** (best): sender signs payload, Studio verifies signature
- **BEARER_TOKEN**: sender includes `Authorization: Bearer <token>`
- **API_KEY**: sender includes a known header value
- **NONE**: only for trusted internal networks

If you’re unsure, use **HMAC_SHA256**.

### Regenerate a webhook secret

Regenerating breaks existing senders until you update them.

1. Open the webhook
2. Click **Regenerate secret**
3. Update the sending system with the new secret
4. Send a test request

---

## Branding

Branding controls how your org appears (especially for landing pages and custom domains).

- Branding page: `/organization/manage/branding`

Typical settings:
- Logo
- Primary/secondary colors
- Tagline
- Hero gradient / header styling

### Custom domains

If your deployment supports custom domains, org branding usually applies automatically.

A typical setup:
1. Operator configures custom-domain routing (often via Cloudflare Tunnel / reverse proxy)
2. You set the org’s custom domain in org settings
3. Landing pages render with your org branding

### Public branding API

Some deployments expose a public branding endpoint used by landing pages:

```

GET /api/v1/public/branding

```

It usually returns branding based on the request domain (Host header).

---

## Billing and subscription

Billing lives at `/organization/manage/billing`.

Typical sections:
- Current plan
- Invoices
- Payment history

### Invoices

From the invoice list you can usually:
- view invoice number
- see amount/status/date
- download a PDF (if supported)

### Cancel or change plan

If plan changes are enabled:
- changing plan updates limits and access
- cancellation usually keeps access until the end of the current billing period

Exact behavior depends on operator configuration.

---

## Limits and usage

Limits live at `/organization/manage/limits`.

You’ll typically see:
- workflows
- users
- storage
- API calls / rate limits

### How to read the usage bars

- **Green**: comfortable
- **Yellow**: approaching a cap
- **Red**: at/over the limit

### Soft limits vs hard limits

- **Soft**: warning only
- **Hard**: enforcement (blocks actions when exceeded)

### Grace periods

Some deployments allow a grace period after you exceed a limit before enforcement starts.

If you’re using marketplace “advanced” packages with token gating, align enforcement here with your product rules (what stops working and when).

---

## Audit logs

Audit logs are your “who did what” feed.

If your deployment enables it, look for an audit page (often `/audit` or within org management).

### Find a specific event fast

Start with filters:
- time range
- actor (user)
- event category
- severity (info/warn/error)

Then open the event detail to see:
- what changed
- previous vs new values (when available)
- request metadata (when enabled)

### Security best practices

- Audit admin role changes
- Audit credential creation/updates
- Watch for repeated failed login attempts (if tracked)
- Export audit data periodically if you have compliance requirements

---

## Quick reference

### Routes

- Org dashboard: `/organization/manage`
- Org settings: `/organization/manage/settings`
- Users: `/organization/manage/users`
- Branding: `/organization/manage/branding`
- Billing: `/organization/manage/billing`
- Limits: `/organization/manage/limits`
- Secrets: `/secrets`
- Webhooks: `/webhooks`

### Admin API endpoints (if you use the API)

Organization / members:
- `PATCH /api/v1/organizations/{id}`
- `GET /api/v1/organizations/{id}/members`
- `POST /api/v1/organizations/{id}/members`

Secrets:
- `POST /api/v1/organizations/secrets`
- `PATCH /api/v1/organizations/secrets/{id}`
- `DELETE /api/v1/organizations/secrets/{id}`

Webhooks:
- `POST /api/v1/webhooks`
- `PUT /api/v1/webhooks/{id}`
- `POST /api/v1/webhooks/{id}/regenerate-secret`

Files (admin-only actions may apply):
- `GET /api/v1/files`
- `DELETE /api/v1/files/{id}`

---

## Next steps

- **Users Guide**: build workflows, run instances, debug failures
- **Operators Guide**: deploy, secure, monitor, backup, upgrade
