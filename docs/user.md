# User Guide

Complete guide for using Studio - workflows, templates, instances, and the visual editor.

---

## Table of Contents

- [Core Concepts](#core-concepts)
- [Dashboard](#dashboard)
- [Workflows](#workflows)
- [Visual Flow Editor](#visual-flow-editor)
- [Templates](#templates)
- [Instances & Execution](#instances--execution)
- [Files](#files)
- [Marketplace](#marketplace)
- [Notifications](#notifications)
- [Keyboard Shortcuts](#keyboard-shortcuts)

---

## Core Concepts

Studio follows the **Template → Workflow → Instance** pattern:

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   TEMPLATE   │ ──►  │   WORKFLOW   │ ──►  │   INSTANCE   │
│              │      │              │      │              │
│ Reusable     │      │ Configured   │      │ Single       │
│ blueprint    │      │ for your use │      │ execution    │
└──────────────┘      └──────────────┘      └──────────────┘
```

- **Template**: A reusable workflow blueprint (shared, versioned)
- **Workflow**: Your configured copy of a template (organization-specific)
- **Instance**: One execution run of a workflow (has status, results)

### User Roles

| Role | Capabilities |
|------|--------------|
| **User** | Create/execute workflows, view own data |
| **Admin** | Manage org resources, members, credentials |
| **Super Admin** | Full system access, infrastructure monitoring |

---

## Dashboard

The dashboard (`/dashboard`) provides an overview of your organization.

### Stats Cards

- **Active Workflows** - Count of active workflows
- **Team Members** - Organization member count
- **Templates** - Available templates

### Quick Actions

- View all workflows
- Manage team
- View templates

### Infrastructure Section (Super Admin Only)

- Redis connection status
- Storage usage statistics
- Worker status

---

## Workflows

### Workflow List (`/workflows/list`)

| Feature | Description |
|---------|-------------|
| Search | Filter workflows by name |
| Pagination | 5, 10, 25, 50, 100 items per page |
| Status filter | Active, inactive, error |

### Creating a Workflow (`/workflows/create`)

Three creation methods:

1. **From Template** - Select from template library
2. **From Workflow** - Duplicate an existing workflow
3. **From Scratch** - Start with blank canvas

### Workflow Editor (`/workflows/[id]/edit`)

The main editing interface with:

- **Settings Panel** (collapsible)
  - Workflow name
  - Status (Draft, Active, Inactive, Archived)
  - Trigger type (Manual, Webhook, Schedule, Event, API)
  - Description

- **Webhook Configuration** (when webhook trigger selected)
  - Webhook URL with copy button
  - Authentication type
  - Secret management
  - Regenerate secret

- **Visual Flow Editor** - See dedicated section below

- **Action Buttons**
  - Save As (copy to new workflow)
  - Cancel
  - Save Workflow
  - Run

### Trigger Types

| Type | Description |
|------|-------------|
| **Manual** | User-initiated execution |
| **Webhook** | HTTP webhook trigger |
| **Schedule** | Cron or interval-based |
| **Event** | Event-based trigger |
| **API** | API endpoint trigger |

---

## Visual Flow Editor

The drag-and-drop workflow builder powered by React Flow.

### Canvas Features

| Feature | Description |
|---------|-------------|
| Drag nodes | Move steps by dragging |
| Resize nodes | Resize on canvas |
| Grid background | Toggle-able grid |
| Zoom | Zoom in/out controls |
| Pan | Pan/scroll canvas |
| Fit to view | Auto-fit all nodes |

### Toolbar Buttons

| Button | Description |
|--------|-------------|
| Add Step | Add new workflow step |
| Toggle Settings | Show/hide quick settings |
| Auto Arrange | Automatically arrange nodes |
| Undo | Revert last change (Ctrl+Z) |
| Redo | Reapply undone change (Ctrl+Shift+Z) |
| Fullscreen | Enter/exit fullscreen mode |

### Quick Settings Panel

| Setting | Options |
|---------|---------|
| Editor Width | Centered, Wide, Full |
| Node Width | Narrow, Normal, Wide |
| Node Spacing | Compact, Normal, Spacious |
| Edge Style | Bezier, Straight, Step |
| Backdrop Blur | None, Light, Heavy |
| Show Grid | On/Off |
| Editor Height | 400-1000px slider |

### Undo/Redo Support

| Feature | Description |
|---------|-------------|
| Undo | Ctrl+Z (Cmd+Z on Mac) - Reverts last change |
| Redo | Ctrl+Shift+Z (Cmd+Shift+Z on Mac) - Reapplies |
| History | Maintains up to 50 states |
| Scope | Tracks steps and connections |

### Unsaved Changes Protection

The editor warns you when attempting to close the browser tab/window with unsaved changes.

### Step Execution Modes

Each step can have an execution mode:

| Mode | Description | Visual |
|------|-------------|--------|
| **Enabled** | Normal execution (default) | Standard appearance |
| **Skip** | Skip step, data flows through | Gray, dashed line |
| **Stop** | Stop workflow at this step | Red border, stop sign |

Toggle modes by:
- Click the button in top-left corner of node
- Use dropdown in step configuration panel

### Step Configuration Panel

Click a step to configure:

| Field | Description |
|-------|-------------|
| Step name | Display name |
| Execution mode | Enabled, Skip, Stop |
| Service type | Category of service |
| Provider | Service provider |
| Service | Specific service |
| Parameters | Service-specific inputs |
| Input mappings | Map outputs from previous steps |
| Output fields | Define step outputs |

### Connection Features

- **Drag to connect** - Draw connections between steps
- **Multiple styles** - Bezier, Straight, Step edges
- **Delete connections** - Click connection, press Delete

---

## Templates

### Template List (`/templates/list`)

Browse and manage templates with:
- Search and filter
- Pagination
- Status badges (Draft, Published, Archived)

### Template Marketplace (`/templates/marketplace`)

| Tab | Content |
|-----|---------|
| My Templates | Your created templates |
| Organization | Shared within org |
| Public | Community templates |

### Template Categories

| Category | Description |
|----------|-------------|
| DATA_PROCESSING | Data transformation |
| AI_GENERATION | AI/ML workflows |
| AUTOMATION | General automation |
| NOTIFICATION | Notification workflows |
| CONTENT_CREATION | Content creation |
| INTEGRATION | System integration |

---

## Instances & Execution

### Instance List (`/instances/list`)

| Status Tab | Description |
|------------|-------------|
| All | All instances |
| Pending | Awaiting execution |
| Processing | Currently executing |
| Active | Running instances |
| Completed | Successfully finished |
| Failed | Execution errors |
| Cancelled | User cancelled |

### Instance Detail (`/instances/[id]`)

View execution progress with:

- **Timeline view** - Visual step-by-step progress
- **Job cards** - Individual step execution details
- **Status indicators** - Completed/failed/processing/pending
- **Duration** - Execution time per step
- **Resource outputs** - Step outputs and files
- **Error messages** - Failure details

### Job Actions

| Button | Description |
|--------|-------------|
| Retry job | Retry a failed job |
| Rerun job | Rerun completed job |
| Rerun only | Rerun without downstream |
| Rerun and continue | Rerun job + all dependents |
| Download resource | Download output files |

### Instance Statuses

| Status | Description |
|--------|-------------|
| PENDING | Created, awaiting execution |
| PROCESSING | Currently executing |
| WAITING_FOR_WEBHOOK | Paused for external callback |
| PAUSED | User paused |
| COMPLETED | Successfully finished |
| FAILED | Error occurred |
| CANCELLED | User cancelled |

---

## Files

The Files page (`/files`) provides a central location to browse, preview, and manage all files in your organization.

### File Sources

Files can come from three sources:

| Source | Description | Icon |
|--------|-------------|------|
| **Generated** | Created by workflow steps (e.g., AI-generated images) | Sparkles |
| **Downloaded** | Downloaded from external providers during workflow execution | Cloud |
| **Uploaded** | Manually uploaded by users | Upload |

### View Modes

Toggle between two view modes using the buttons in the top-right corner:

| Mode | Description |
|------|-------------|
| **Grid View** | Card-based layout with thumbnails, ideal for images |
| **Table View** | Compact list with sortable columns, ideal for many files |

Your view preference is saved and persisted across sessions.

### Filtering & Sorting

| Filter | Options |
|--------|---------|
| **Search** | Filter by filename |
| **File Type** | All, Images, Videos, Audio, Documents, Other |
| **Source** | All, Generated, Downloaded, Uploaded |
| **Sort By** | Name, Date, Size, Type, Source (with asc/desc) |

In table view, click column headers to sort. In grid view, use the Sort By buttons.

### Previewing Files

Click on a file card or table row to open the file viewer modal for supported types:

| File Type | Preview Support |
|-----------|-----------------|
| Images | Full preview with zoom |
| Videos | Video player |
| Audio | Audio player |
| PDFs | PDF viewer |

Use arrow keys or navigation buttons to browse between files in the viewer.

### Multi-Select & Delete

Select multiple files for batch operations:

**In Grid View:**
- Hover over a card to reveal the checkbox
- Click the checkbox to select/deselect
- Selected files show a blue border

**In Table View:**
- Click the checkbox in the first column
- Use the header checkbox to select/deselect all visible files
- Partial selection shows an indeterminate state

**Floating Action Bar:**

When files are selected, a floating bar appears at the bottom with:
- Selection count (e.g., "3 files selected")
- **Delete** button - Opens confirmation dialog
- **Cancel** button - Clears selection

**Delete Confirmation:**

Clicking Delete shows a confirmation modal warning that:
- Files will be permanently deleted
- Associated thumbnails will be removed
- This action cannot be undone

### File Information

Each file card/row displays:

| Field | Description |
|-------|-------------|
| **Thumbnail** | Preview image (for images with thumbnails) |
| **Filename** | Display name of the file |
| **Size** | File size (KB, MB, GB) |
| **Type** | MIME type (e.g., PNG, MP4, PDF) |
| **Date** | Creation date |
| **Source** | How the file was created |
| **Status** | Ready, Pending, Generating, Failed |

### Downloading Files

Click the Download button on any file card or use the download button in the file viewer modal.

---

## Marketplace

### Provider Marketplace (`/providers/marketplace`)

Browse and install service providers.

| Filter | Options |
|--------|---------|
| Tier | All, Basic, Advanced |
| Category | By service type |
| Status | All, Installed, Available |

### Package Tiers

| Tier | Access |
|------|--------|
| **Built-in** | Always available |
| **Basic** | Free add-ons |
| **Advanced** | Community members |

### Installing a Provider

1. Browse marketplace
2. Click "Install" on desired provider
3. Configure credentials if required
4. Provider services available in workflow editor

---

## Notifications

### Notifications Page (`/notifications`)

| Tab | Description |
|-----|-------------|
| All | All notifications |
| Unread | Unread only |
| Workflow | Workflow events |
| Job | Job execution events |
| System | System notifications |
| Account | Account events |
| Security | Security alerts |
| Billing | Billing notifications |

### Notification Actions

- Mark as read
- Clear all
- Filter by status

---

## Keyboard Shortcuts

### Flow Editor

| Shortcut | Action |
|----------|--------|
| `Ctrl+Z` / `Cmd+Z` | Undo |
| `Ctrl+Shift+Z` / `Cmd+Shift+Z` | Redo |
| `Ctrl+Y` / `Cmd+Y` | Redo (Windows style) |
| `Delete` / `Backspace` | Delete selected |
| `Escape` | Exit fullscreen |

### General

| Shortcut | Action |
|----------|--------|
| `Ctrl+S` / `Cmd+S` | Save (when in editor) |

---

## Tips

### Workflow Design

1. **Start simple** - Begin with minimal steps, add complexity later
2. **Name clearly** - Use descriptive step names
3. **Test incrementally** - Test after each major change
4. **Use Skip mode** - Temporarily disable steps for debugging

### Performance

1. **Limit history** - Undo/redo keeps 50 states max
2. **Save regularly** - Unsaved changes warning protects you
3. **Use templates** - Reuse proven workflow patterns

### Troubleshooting

1. **Check credentials** - Yellow warning banner indicates missing credentials
2. **View logs** - Instance detail shows step-by-step execution
3. **Retry failed** - Use retry button for transient failures

---

## Workflow Failure Troubleshooting

### Common Failure Causes

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| Step shows "No credentials" | Provider credentials not configured | Go to Secrets → Provider Credentials and add API key |
| Step times out | External API slow or unresponsive | Check provider status; increase timeout if available |
| "Invalid parameters" error | Required field missing or wrong type | Review step configuration; check field mappings |
| "Rate limit exceeded" | Too many requests to provider | Wait and retry; consider batch processing |
| Step stuck on "pending" | Worker not processing | Check worker status in Infrastructure dashboard |

### Debugging a Failed Instance

1. **Go to instance detail page** (`/instances/{id}`)
2. **Expand the failed job** - Click on the red status indicator
3. **Check error message** - Shows exact API error
4. **Review input parameters** - Verify mapped values are correct
5. **Check timing** - Look for timeout vs. immediate failures

### Using Skip Mode for Debugging

When a workflow fails midway:

1. **Edit the workflow** - Go to `/workflows/{id}/edit`
2. **Set failed step to "Skip"** - Click execution mode button on the step
3. **Add a temporary test step** - Debug with simpler parameters
4. **Run again** - Workflow skips problematic step
5. **Fix and restore** - Once working, set step back to "Enabled"

### Retry Strategies

| Button | What it does |
|--------|--------------|
| **Retry job** | Re-runs just this failed step |
| **Rerun only** | Runs step again with same inputs |
| **Rerun and continue** | Runs step + all downstream steps |

Use "Retry job" for transient failures (network issues). Use "Rerun and continue" when you've fixed the root cause.

---

## Next Steps

- **[Admin Guide](/docs/admin)** - Organization management, users, and credentials
