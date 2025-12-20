# Users Guide

Use Studio to build workflows, run them, and understand what happened when things go right (or wrong).

If you just want to *get something running*, start at **Your first workflow**.

---

## Common tasks

Most people come here to do one of these:

- **Build your first workflow** → [Your first workflow](#your-first-workflow)
- **Add an API key / credentials** → [Add provider credentials](#add-provider-credentials)
- **Trigger a workflow from the outside** → [Webhooks](#webhooks)
- **See why a run failed** → [Debug a failed run](#debug-a-failed-run)
- **Retry a failed step** → [Retry and rerun](#retry-and-rerun)
- **Find files created by a workflow** → [Files](#files)
- **Install a provider from the marketplace** → [Marketplace](#marketplace)

---

## Studio in one minute

Studio is built around a simple loop:

**Template → Workflow → Instance**

- **Template**: a reusable blueprint (often shared)
- **Workflow**: your configured version (steps + settings + credentials)
- **Instance**: one run of a workflow (results, logs, files, errors)

You don’t need to memorize this. You’ll feel it after your first run.

---

## Your first workflow

Goal: create something small, run it, and watch it finish.

### 1) Create a workflow

Go to **Workflows → Create New** (`/workflows/create`) and choose **From Scratch**.

Give it a name like **My First Workflow**.

### 2) Add one step

Open the editor (`/workflows/{id}/edit`) and click **+ Add Step**.

Pick:
- **Service type** (what kind of thing you want to do)
- **Provider** (who offers it)
- **Service** (the exact action)

**Tip:** start with the **Core** provider if you see it. Core steps are built-in and usually don’t require API keys.

### 3) Configure the step

Fill in the required fields (marked with `*`). Leave everything else alone for now.

### 4) Save and run

Click **Save Workflow**, then **Run**.

You’ll land on the instance view (`/instances/{id}`) where you can watch each step execute.

### What “good” looks like

- The instance moves through steps and finishes as **Completed**
- Each step shows a result payload (even if it’s small)

If you hit **No credentials configured**, that’s normal — it means you chose a provider that needs an API key. Next section.

---

## Add credentials and secrets

Studio has two related concepts:

- **Provider credentials**: “Log into this external service” (API keys, OAuth, etc.)
- **Secrets**: “Store this sensitive value and reuse it anywhere” (tokens, URLs, shared secrets)

Most workflow issues early on are just missing credentials.

### Add provider credentials

Use this when a step talks to a provider (OpenAI, Leonardo, Google Drive, etc.).

1. Go to **Secrets** (`/secrets`)
2. Open **Provider Credentials**
3. Choose the provider
4. Paste the required values (API key, client secret, etc.)
5. Save

Now go back to your workflow and run again.

**Common gotcha:** if you have multiple organizations, make sure you’re adding credentials in the right org.

### Create an organization secret

Use this when you want a reusable value (like `WEBHOOK_SECRET`, `SLACK_TOKEN`, or `MY_API_BASE_URL`).

1. Go to **Secrets** (`/secrets`)
2. Open **Organization Secrets**
3. Click **Create Secret**
4. Name it something clear (example: `OPENAI_API_KEY`, `SLACK_WEBHOOK_URL`)
5. Save

### Use a secret inside a step

In a step configuration panel, look for places where you can provide a value via mapping.

Common patterns:
- A field lets you select **Secret** / **From secrets**
- Or you use **Input Mappings** and choose a value source

If you can’t find a secret picker yet, your step UI might only support raw values in that field today. In that case, store the value as a provider credential (preferred for provider auth) or as a secret for workflow config values.

### Rotate a key safely

When you rotate keys (recommended):
1. Create the new key in the external provider
2. Update it in Studio (credentials or secret)
3. Re-run a small test workflow
4. Only then revoke the old key

---

## Workflows

Workflows are where you build the automation.

### Where workflows live

- List: `/workflows/list`
- Create: `/workflows/create`
- Edit: `/workflows/{id}/edit`

### Save vs Save As

- **Save** updates the workflow you’re editing
- **Save As** makes a copy (best way to experiment without breaking a working workflow)

### Step modes: Enabled, Skip, Stop

Use these when you’re debugging:

- **Enabled**: runs normally
- **Skip**: bypass the step (data flows through)
- **Stop**: run up to this point, then stop

If your workflow fails halfway through, setting the failing step to **Skip** is often the quickest way to validate the rest of the flow.

---

## Webhooks

Webhooks let something outside Studio start your workflow.

### Create or enable a webhook trigger

1. Open your workflow editor (`/workflows/{id}/edit`)
2. Set trigger type to **Webhook**
3. Copy the webhook URL
4. Configure the external service to POST to that URL

### Secure your webhook

If Studio shows a webhook **secret**, use it.

Typical security setups:
- **HMAC signature** (best)
- **Bearer token**
- **Secret query/header** (acceptable for internal use)

If you don’t secure the webhook, anyone who finds the URL can trigger it.

### When a webhook “does nothing”

Check:
- Is the workflow **Active**?
- Did the request reach Studio (you should see an instance created)?
- Is the webhook URL correct (no missing org slug)?
- Is authentication configured correctly (signature/token)?

---

## Mapping data between steps

This is how workflows become real automation: step outputs feed the next step.

### The idea

Step A produces output (like an image URL). Step B uses that output as input (upload the image).

### Where to map values

In the step configuration panel, find **Input Mappings** (or mapping controls next to fields).

Typical mapping sources:
- **Static value**: type it in
- **From previous step**: choose an output field
- **Form input**: ask the user at run time (if supported)

### Debug mapping problems

If a step fails with “invalid parameters”:
- Confirm required fields are mapped
- Confirm types match (string vs number vs object)
- Check the instance detail to see what the step actually received

---

## Templates

Templates are reusable workflow blueprints.

### Use a template

1. Go to Templates (`/templates/list` or `/templates/marketplace`)
2. Pick a template
3. Create a workflow from it
4. Add credentials as needed
5. Run it

### Publish your workflow as a template

If your org supports template publishing:
- Start from a working workflow
- Clean up step names and descriptions
- Replace hardcoded sensitive values with secrets/credentials
- Then publish

---

## Runs and results

Every time you click **Run**, Studio creates an **Instance**.

### Find your instances

- List: `/instances/list`
- Detail: `/instances/{id}`

### Read an instance like a debugger

When something fails:
1. Open the instance
2. Find the first red step/job
3. Open its error details
4. Check inputs (what the step received)
5. Check outputs from the previous step (what it produced)

### Debug a failed run

Most failures fall into a few buckets:

- **No credentials configured**
  - Add provider credentials under `/secrets`, then re-run

- **Invalid parameters**
  - A required field is missing or the value type is wrong
  - Fix step config or mappings

- **Timeout**
  - External service is slow or unavailable
  - Try again; if it repeats, reduce workload or check provider status

- **Rate limit exceeded**
  - You hit a provider’s API limits
  - Wait, retry, or reduce concurrency (if your org/operator configured it)

### Retry and rerun

You’ll typically see actions like:

- **Retry job**: reruns just the failed step
- **Rerun only**: reruns a step (even if completed)
- **Rerun and continue**: reruns a step and downstream steps

Use **Retry job** for flaky network/provider issues.
Use **Rerun and continue** after you change configuration or credentials.

---

## Files

Workflows can generate or download files, and Studio keeps them in one place.

- Files page: `/files`

### What you can do there

- Browse by type (images, video, documents, etc.)
- Preview supported formats (image/video/audio/PDF)
- Download outputs from runs
- Delete files (careful: this is usually permanent)

### If files aren’t showing up

Check:
- Did the workflow step actually create/download a file?
- Did the instance complete that step successfully?
- Are you in the right organization?

---

## Marketplace

Marketplace is where you install providers and templates (if enabled in your deployment).

- Providers: `/providers/marketplace`
- Templates: `/templates/marketplace`

### Install a provider

1. Open provider marketplace
2. Click **Install**
3. Add credentials under `/secrets` if required
4. Use the provider’s services in the workflow editor

If a provider is “Advanced” or otherwise gated, your access depends on how your instance/operator configured tiers.

---

## Notifications

Notifications help you keep track of what changed and what failed.

- Notifications: `/notifications`

Typical categories:
- workflow events
- job/step failures
- system events
- security/billing (if enabled)

Use notifications for “what happened” and instances for “why it happened.”

---

## Quick reference

### Workflow statuses

- **Draft**: still building (often can run manually, but triggers may be off)
- **Active**: ready to run; triggers enabled
- **Inactive**: paused; won’t respond to triggers
- **Archived**: hidden/retained for records

### Instance statuses

- **PENDING**: created, waiting to execute
- **PROCESSING**: running now
- **WAITING_FOR_WEBHOOK**: paused for external callback
- **PAUSED**: manually paused
- **COMPLETED**: finished successfully
- **FAILED**: error occurred
- **CANCELLED**: stopped by user/system

### Keyboard shortcuts

Flow editor:
- `Ctrl/Cmd + Z` undo
- `Ctrl/Cmd + Shift + Z` redo
- `Delete/Backspace` delete selection
- `Escape` exit fullscreen

General:
- `Ctrl/Cmd + S` save (in editor)

---

## Next steps

- **Admins Guide**: org setup, users, credentials, branding, limits
- **Getting Started**: installation and setup
- **API Reference**: if you want to trigger workflows from your own code
