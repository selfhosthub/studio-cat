# Admin Guide

Organization administration guide - managing users, credentials, branding, and billing.

---

## Table of Contents

- [Organization Management](#organization-management)
- [User Management](#user-management)
- [Secrets & Credentials](#secrets--credentials)
- [Audit Logs](#audit-logs)
- [Branding](#branding)
- [Billing & Subscription](#billing--subscription)
- [Limits & Usage](#limits--usage)

---

## Organization Management

### Organization Dashboard (`/organization/manage`)

| Tab | Description |
|-----|-------------|
| Overview | Organization summary |
| Details | Name, slug, website, logo |
| Users | Member management |
| Settings | Configuration options |
| Billing | Subscription and invoices |
| Branding | Custom branding |
| Limits | Usage limits display |

### Organization Status

Organizations have a status that determines their access level:

| Status | Description | Capabilities |
|--------|-------------|--------------|
| **pending_approval** | Awaiting admin approval | Build templates/workflows, limited storage |
| **active** | Full access | All features based on plan limits |
| **suspended** | Restricted access | Read-only, no executions |

#### Status Transitions

- **New Registration** → `pending_approval` (if auto-activate disabled) or `active` (if auto-activate enabled)
- **Admin Approval** → `pending_approval` → `active`
- **Billing Issue** → `active` → `suspended`
- **Policy Violation** → `active` → `suspended`
- **Issue Resolved** → `suspended` → `active`

#### Pending Organization Limits

Organizations in `pending_approval` status have restricted limits:

| Limit | Default |
|-------|---------|
| Max Users | 1 (creator only) |
| Max Executions | 0 (cannot run workflows) |
| Max Storage | 50 MB |

These defaults are configured in platform registration settings.

### Organization Details

| Field | Description |
|-------|-------------|
| Organization name | Display name |
| Organization slug | URL-safe identifier |
| Website | Organization website URL |
| Logo | Organization logo upload |
| Status | Current organization status |
| Activated at | When the organization was activated |
| Activated by | Who activated the organization |

### Organization Settings (`/organization/manage/settings`)

Configure organization-wide preferences.

#### Images & Thumbnails

| Setting | Description |
|---------|-------------|
| **Show image thumbnails** | Display thumbnail previews for image files in the Files page and instance views. Disable to reduce bandwidth on slower connections. |

#### Display Settings

| Setting | Options | Description |
|---------|---------|-------------|
| **Resource Card Size** | Small, Medium, Large | Controls the size of file cards in grid view throughout the application |

These settings affect all users in the organization.

---

## User Management

### Managing Users (`/organization/manage/users`)

| Column | Description |
|--------|-------------|
| Username | User's username |
| Email | User's email |
| First/Last Name | Full name |
| Role | User role (dropdown for admins) |
| Status | Active/Inactive toggle |
| Actions | Edit, Remove buttons |

### Inviting Users

Click "Invite User" to add new members:

| Field | Description |
|-------|-------------|
| Email | User's email address |
| Username | Unique username |
| Password | Initial password |
| Role | user, admin, super_admin |

### User Roles

| Role | Permissions |
|------|-------------|
| **User** | Create/execute workflows, view own data, use services |
| **Admin** | All User permissions + manage org resources, members, credentials |
| **Super Admin** | All Admin permissions + system-wide access, infrastructure |

### Editing Users

- Update name, email, role
- Toggle active status
- Reset password

---

## Secrets & Credentials

### Secrets Page (`/secrets`)

Two types of secrets:

1. **Organization Secrets** - General-purpose secrets
2. **Provider Credentials** - API keys for service providers

### Organization Secrets

| Field | Description |
|-------|-------------|
| Name | Secret identifier |
| Type | Classification |
| Description | Help text |
| Value | The secret (encrypted) |
| Expiration | Optional expiry date |
| Status | Active/Inactive |

### Creating Secrets

1. Click "New Secret"
2. Enter name and type
3. Add secret value (masked)
4. Optional: Set expiration date
5. Save

### Provider Credentials

Provider credentials authenticate workflows with external services.

| Field | Description |
|-------|-------------|
| Provider | Service provider |
| Credential type | API_KEY, OAUTH2, etc. |
| Created | Creation date |
| Expiration | Expiry date |

### Credential Types

| Type | Description |
|------|-------------|
| API_KEY | Simple API key |
| ACCESS_KEY | Cloud provider keys |
| OAUTH2 | OAuth 2.0 flows |
| JWT | JSON Web Token |
| BASIC | HTTP Basic Auth |
| CUSTOM | Custom authentication |

### Security Features

| Feature | Description |
|---------|-------------|
| AES encryption | Encrypted at rest |
| Organization scoped | No sharing between orgs |
| Expiration tracking | Monitor credential expiry |
| Never exposed | Hidden in API responses |
| Audit trail | Last used tracking |

---

## Audit Logs

### Audit Logs Page (`/audit`)

Monitor all security-related and administrative events within your organization.

| Column | Description |
|--------|-------------|
| Event | Action performed (Created, Updated, Deleted, etc.) |
| Resource | Type and name of affected resource |
| Actor | Who performed the action |
| Severity | info, warning, or critical |
| Category | security, configuration, access, or audit |
| Timestamp | When the event occurred |

### Event Categories

| Category | Events Tracked |
|----------|----------------|
| **Security** | Secret reveals, credential changes, login attempts |
| **Configuration** | User role changes, organization updates |
| **Access** | Resource views and downloads |
| **Audit** | Viewing the audit log itself |

### Severity Levels

| Severity | Description | Examples |
|----------|-------------|----------|
| **Info** | Normal operations | Secret created, user invited |
| **Warning** | Attention needed | Failed login attempt |
| **Critical** | Security concerns | Secret value revealed, multiple failed logins |

### Filtering Events

Filter the audit log by:
- **Category** - Security, Configuration, Access, Audit
- **Severity** - Info, Warning, Critical
- **Resource Type** - Secrets, Credentials, Users, Workflows, etc.
- **Search** - Free text search across event details

### Event Details

Expand any event to view:
- Full change details (what was modified)
- Metadata (IP address, user agent, etc.)
- Resource and organization IDs

### Visibility Rules

| Role | Can View |
|------|----------|
| **User** | No access to audit logs |
| **Admin** | Organization's events only |
| **Super Admin** | All events (all orgs + system events) |

### Security Best Practices

1. **Regular review** - Check audit logs weekly for anomalies
2. **Monitor critical events** - Set up alerts for security events
3. **Track secret access** - Know who reveals secret values
4. **Review failed logins** - Investigate repeated failures

### Audit API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/audit/` | GET | List audit events with filters |
| `/api/v1/audit/system` | GET | System events (super_admin only) |
| `/api/v1/audit/resource/{type}/{id}` | GET | Resource history |
| `/api/v1/audit/actor/{id}` | GET | Actor history |

---

## Branding

### Branding Page (`/organization/manage/branding`)

Customize your organization's appearance.

| Setting | Description |
|---------|-------------|
| Logo | Organization logo |
| Primary color | Theme accent color |
| Secondary color | Supporting color |
| Tagline | Marketing tagline |
| Hero gradient | Landing page gradient |
| Header styling | Header customization |

### Custom Domain Landing Pages

Organizations can have branded landing pages via custom domains (e.g., `workflows.mycompany.com`).

**Setup:**
1. Configure custom domain in organization settings
2. Set up Cloudflare tunnel (see Operators Guide)
3. Branding applies automatically to custom domain

### Public Branding API

Unauthenticated endpoint for custom domain branding:
```
GET /api/v1/public/branding
```

Returns branding config based on request domain.

---

## Billing & Subscription

### Billing Page (`/organization/manage/billing`)

| Tab | Content |
|-----|---------|
| Current Plan | Subscription details |
| Invoices | Invoice history |
| Payments | Payment history |

### Invoices

| Column | Description |
|--------|-------------|
| Invoice number | Unique identifier |
| Amount | Invoice amount |
| Status | paid/pending/overdue |
| Date | Invoice date |
| Download | PDF download |

### Payments

| Column | Description |
|--------|-------------|
| Payment ID | Transaction ID |
| Amount | Payment amount |
| Status | completed/pending/failed |
| Date | Payment date |

### Subscription Management

- View current plan
- Change plan
- Cancel subscription
- View usage against limits

### Payment Integrations

Studio supports:
- **Stripe** - Credit card payments
- **LemonSqueezy** - Alternative payment processor

Configure in organization billing settings.

---

## Limits & Usage

### Limits Page (`/organization/manage/limits`)

| Limit Type | Description |
|------------|-------------|
| Workflows | Maximum workflow count |
| Users | Maximum team members |
| Storage | Maximum storage space |
| API calls | Rate limits |

### Usage Indicators

Progress bars show current usage vs. limits:
- Green: Below 70%
- Yellow: 70-90%
- Red: Above 90%

### Limit Types

| Type | Behavior |
|------|----------|
| Soft limits | Warning only |
| Hard limits | Enforcement at boundary |
| Metered limits | Tracked and reported |

### Grace Period

Organizations exceeding limits receive a grace period before enforcement:
- Automatic grace period granted
- Monitoring in super admin dashboard
- Blocking after grace expires

---

## Webhooks

### Webhooks Page (`/webhooks`)

Manage incoming webhook endpoints.

| Field | Description |
|-------|-------------|
| Name | Webhook identifier |
| URL slug | Unique path |
| Workflow | Associated workflow |
| Status | Active/Inactive |
| Auth type | Authentication method |
| Created | Creation date |

### Webhook URL Pattern

```
/webhooks/incoming/{org_slug}/{webhook_slug}
```

**Understanding the URL:**
- `{org_slug}` - Your organization's URL-safe identifier (found in Organization → Details). Example: `acme-corp`
- `{webhook_slug}` - Auto-generated from webhook name, or manually set. Example: `process-orders`

Full example URL: `https://studio.example.com/webhooks/incoming/acme-corp/process-orders`

### Webhook Authentication

| Type | Description |
|------|-------------|
| NONE | No authentication |
| HMAC_SHA256 | HMAC signature validation |
| BEARER_TOKEN | Bearer token auth |
| API_KEY | API key header auth |

### Webhook Details

Expandable details include:
- Full webhook URL (with copy button)
- Secret (with show/hide toggle)
- Regenerate secret button
- Delete webhook button

---

## Best Practices

### Security

1. **Rotate credentials regularly** - Use expiration dates
2. **Use least privilege** - Assign minimum necessary role
3. **Monitor usage** - Review audit logs
4. **Enable 2FA** - For admin accounts

### Organization

1. **Clear naming** - Use descriptive workflow/template names
2. **Document credentials** - Add descriptions to secrets
3. **Regular cleanup** - Archive unused workflows
4. **Monitor limits** - Stay within usage limits

### Team Management

1. **Onboarding** - Create users with User role first
2. **Gradual access** - Promote to Admin when needed
3. **Off-boarding** - Deactivate (don't delete) departing members
4. **Audit roles** - Review role assignments periodically

---

## Admin API Endpoints

### Organization

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/organizations/{id}` | PATCH | Update organization |
| `/api/v1/organizations/{id}/members` | GET | List members |
| `/api/v1/organizations/{id}/members` | POST | Add member |
| `/api/v1/organizations/{id}/activate` | POST | Activate organization (super_admin) |
| `/api/v1/organizations/{id}/suspend` | POST | Suspend organization (super_admin) |
| `/api/v1/organizations/{id}/set-pending` | POST | Set to pending_approval (super_admin) |

### Credentials

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/providers/{id}/credentials` | POST | Create credential |
| `/api/v1/providers/{id}/credentials` | GET | List credentials |
| `/api/v1/providers/credentials/{id}` | PATCH | Update credential |

### Secrets

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/organizations/secrets` | POST | Create secret |
| `/api/v1/organizations/secrets/{id}` | PATCH | Update secret |
| `/api/v1/organizations/secrets/{id}` | DELETE | Delete secret |

### Webhooks

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/webhooks` | POST | Create webhook |
| `/api/v1/webhooks/{id}` | PUT | Update webhook |
| `/api/v1/webhooks/{id}/regenerate-secret` | POST | Regenerate secret |

### Files

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/files` | GET | List organization files |
| `/api/v1/files/{id}` | GET | Get file details |
| `/api/v1/files/{id}` | DELETE | Permanently delete file (admin only) |
| `/api/v1/files/{id}/download` | GET | Download file content |
| `/api/v1/files/{id}/preview` | GET | Get thumbnail preview |

**Note:** Deleting a file permanently removes both the file and its thumbnail from storage. This action cannot be undone.

---

## Next Steps

- **[User Guide](/docs/user)** - Using workflows, templates, and instances
