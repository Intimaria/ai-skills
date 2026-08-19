# Redmine REST API — creating, updating, and logging time

How to find, post, and update ticket content and time entries in Redmine programmatically. Validated
on `proyectos.mikroways.net`. **Reads** (searching and reading issues) are safe and need no approval.
**Writes** (create, update, note, time entry, note edit) are always an **explicit, user-approved
step** — never do them automatically. Draft to a file first, show it, and only call the write API
once the user confirms (see **Protocols → Delivery** in `SKILL.md`).

**Never delete.** This skill only reads, creates, and updates — it never issues `DELETE` against any
Redmine resource (issue, journal note, time entry, attachment). If a user asks to delete something,
tell them to do it from the Redmine UI; do not call the delete API.

## Table of contents
- [Auth & config](#auth--config)
- [Find & read issues](#find--read-issues)
- [Create an issue](#create-an-issue)
- [Update an issue / add a note](#update-an-issue--add-a-note)
- [Edit an existing note (journal)](#edit-an-existing-note-journal)
- [Log time entries](#log-time-entries)
- [Instance-specific IDs](#instance-specific-ids)
- [JSON encoding](#json-encoding)

## Auth & config
- Base URL and token come from env vars: `REDMINE_URL` + the API key. **Never hardcode the token,
  never commit it.** Recommended: keep them in a sops-encrypted file and inject with `sops exec-env`
  — see "Getting your API token" below.
- Auth header on every request: `X-Redmine-API-Key: <token>`.
- Content type for writes: `Content-Type: application/json`.
- The ticket body is **Markdown** (the instance renders it), placed in `issue.description` (for the
  description) or `issue.notes` (for a journal note / comment).

### Getting your API token
Each user has a personal API key tied to their own permissions — use your own, don't share one.

1. Log in to Redmine (e.g. `https://proyectos.mikroways.net`).
2. Go to **My account** (top-right menu, or `{REDMINE_URL}/my/account`).
3. In the right-hand sidebar, under **API access key**, click **Show** to reveal the key (or
   **Reset** to generate a fresh one — resetting invalidates the old key).
4. Store it **encrypted at rest** (recommended), or at minimum in an env var — never in the repo.

**Recommended — sops + age (works across shells, harnesses, and direnv'd repos).** Keep the token in
a sops-encrypted file and decrypt it only into the process that makes the call — no plaintext on
disk, nothing left resident in your environment. One-time setup (the personal age key at the default
path is auto-discovered by sops from any directory):
   ```shell
   age-keygen -o ~/.config/sops/age/keys.txt && chmod 600 ~/.config/sops/age/keys.txt
   # note the "Public key:" it prints — that is <YOUR-AGE-PUBKEY> below
   mkdir -p ~/.config/redmine
   printf 'creation_rules:\n  - path_regex: redmine\\.sops\\.yaml$\n    age: <YOUR-AGE-PUBKEY>\n' \
     > ~/.config/redmine/.sops.yaml
   printf 'REDMINE_URL: https://proyectos.mikroways.net\nREDMINE_API_KEY: <your-key>\n' \
     > ~/.config/redmine/redmine.sops.yaml
   sops encrypt --in-place ~/.config/redmine/redmine.sops.yaml
   ```
   Edit the token later with `sops ~/.config/redmine/redmine.sops.yaml`. Make any API call with the
   secrets injected only into that one process:
   ```shell
   sops exec-env ~/.config/redmine/redmine.sops.yaml \
     'curl -sf -H "X-Redmine-API-Key: $REDMINE_API_KEY" "$REDMINE_URL/users/current.json"'
   ```
   sops unions its key sources, so this still works inside a repo whose direnv sets `SOPS_AGE_KEY_FILE`
   to a different project key — your personal key at the default path is found regardless.

**Fallback — plain env vars** (portable, but the token sits in your shell in cleartext):
   ```shell
   export REDMINE_URL="https://proyectos.mikroways.net"
   export REDMINE_API_KEY="<your-key>"
   ```

If the **API access key** section isn't there, the REST API is disabled instance-wide: an admin
must enable it under **Administration → Settings → API → "Enable REST web service"**. Verify your
token works with a harmless read:
```shell
curl -sf -H "X-Redmine-API-Key: $REDMINE_API_KEY" "$REDMINE_URL/users/current.json"
```
A `401` means the key is wrong or the REST API is off; a JSON user object means you're good.

## Find & read issues
Reads are safe (no approval needed), but **never guess a ticket number** — search, show the matches,
and let the user confirm which one before you edit anything.

**Search by subject** (`~` means "contains"):
```shell
curl -sf -H "X-Redmine-API-Key: $REDMINE_API_KEY" \
  "$REDMINE_URL/issues.json?subject=~migración+monitoring&limit=25"
```
Combine filters to narrow it down:
- `project_id=<id-or-identifier>` — restrict to one project
- `assigned_to_id=me` — only issues assigned to the token's owner (`me` is a valid literal)
- `status_id=open` (default) / `closed` / `*` (any)
- `sort=updated_on:desc`, `limit=<n>` — order and cap the results

**List my open tickets:**
```shell
curl -sf -H "X-Redmine-API-Key: $REDMINE_API_KEY" \
  "$REDMINE_URL/issues.json?assigned_to_id=me&status_id=open&sort=updated_on:desc&limit=50"
```
Each result carries `id`, `subject`, `status.name`, `project.name` — present those so the user can
pick. If nothing matches, say so; don't invent a ticket.

**Read one issue back, with its notes:**
```shell
curl -sf -H "X-Redmine-API-Key: $REDMINE_API_KEY" \
  "$REDMINE_URL/issues/14971.json?include=journals"
```
`include=journals` returns the comment history; each journal entry has its own `id` — you need that
id to edit an existing note (see below). This is also the read-back used to confirm a write stuck.

## Create an issue
`POST {REDMINE_URL}/issues.json`

**Required fields** — a create fails without these:
- `project_id` — which project the issue belongs to (numeric id or identifier)
- `subject` — the issue title

**Commonly needed:**
- `tracker_id` — issue type (Bug, Feature, Support…); often required by the project's workflow
- `description` — the Markdown body
- `status_id`, `priority_id`, `assigned_to_id`, `done_ratio` — optional

```json
{
  "issue": {
    "project_id": 42,
    "subject": "Migración de monitoring — Mimir y Loki",
    "tracker_id": 1,
    "description": "## Resumen para gestión\n\nSe actualizó el stack de monitoring...\n\n## Detalle técnico\n..."
  }
}
```

Creating a new issue is the most visible action against a client's Redmine — **always confirm with
the user before creating.**

## Update an issue / add a note
`PUT {REDMINE_URL}/issues/{id}.json`

Updates are lower-risk than creates, but still require approval. **No field is strictly required**;
send only what you're changing. To add a comment/journal entry, send `notes`:

```json
{
  "issue": {
    "notes": "## Resumen para gestión\n\nEn este período se avanzó...\n\n## Detalle técnico\n...",
    "status_id": 2,
    "assigned_to_id": 57
  }
}
```

- `notes` — Markdown journal note (a comment on the issue)
- `status_id` — workflow state; **default `2` ("En curso")** — see "Status, assignee & progress" below
- `assigned_to_id` — who owns the issue; set it to the **author (you)** — see below
- `done_ratio` — percent complete (0–100); often **derived from the status**, see below
- `private_notes: true` — make the note internal (not client-visible), when the instance supports it

A successful `PUT` returns `204 No Content` (empty body).

## Edit an existing note (journal)
`PUT {REDMINE_URL}/journals/{journal_id}.json`

Use this to **fix a note you already posted** — correcting a typo, a wrong version, a broken table.
It rewrites that comment in place. Do **not** add a fresh note via the issue `PUT` when the user
asked to correct an existing one (that stacks a second comment instead of fixing the first).

1. Find the journal id: read the issue with `include=journals` (see "Find & read issues") and pick
   the entry to fix — its `id` is the journal id (not the issue id).
2. Send the corrected Markdown:

```json
{
  "journal": {
    "notes": "## Resumen para gestión\n\n(texto corregido)\n..."
  }
}
```

A successful `PUT` returns `204 No Content`. Editing a note is a write — confirm the corrected text
with the user first, same as any other write.

## Log time entries
`POST {REDMINE_URL}/time_entries.json`

Hours are worked out with the author and approved conversationally, then logged **per ticket, per
day**. They never appear in the ticket body or the management summary — the PM plans from the logged
time (see **Protocols → Hours**).

**Required:** `issue_id` (or `project_id`), `hours`, and `activity_id`. `spent_on` defaults to today
if omitted — set it explicitly when back-dating a day's work.

```json
{
  "time_entry": {
    "issue_id": 14971,
    "spent_on": "2026-07-29",
    "hours": 2.5,
    "activity_id": 9,
    "comments": "Deploy y verificación de Mimir 5.8.0"
  }
}
```

Post one entry per (ticket, task). For a task worked across several iterations, post one entry per task. You may have more than one entry per ticket per day.

## Instance-specific IDs
The numeric IDs below are what `proyectos.mikroways.net` uses. **They are per-instance** — on any
other Redmine they mean something different (or don't exist). If you're working against a different
instance, look up the real IDs first with these read endpoints (same auth header):

| What | Discover with | Notes |
|------|---------------|-------|
| `status_id` | `GET /issue_statuses.json` | On `proyectos.mikroways.net`: `2` = "En curso", `10` = "Validar por el cliente" |
| `activity_id` (time entries) | `GET /enumerations/time_entry_activities.json` | e.g. Development, Support |
| `tracker_id` | `GET /trackers.json` | Bug / Feature / Support / … |
| `project_id` | `GET /projects.json` | numeric id or string identifier |
| `priority_id` | `GET /enumerations/issue_priorities.json` | |

> Footnote: if this skill is ever used against a Redmine other than `proyectos.mikroways.net`,
> re-check every id above — status/activity/tracker/priority ids are not portable between instances.

## Status, assignee & progress (and confirming they stuck)

On a real post these three fields silently failed to apply (the issue stayed "Nueva", unassigned,
0% done), so set them deliberately and verify:

- **Status.** Default to `status_id: 2` ("En curso") when logging work that's in progress. Use
  `status_id: 10` ("Validar por el cliente") **only when the work is actually complete** and ready
  for client review. "Resuelta" is set by management, not by this skill. Status is **never written
  in the ticket body** — always via the API.
  *Caveat:* Redmine restricts status changes by **role + tracker workflow**. If the API user's role
  isn't allowed the transition (e.g. `Nueva → En curso`), the change is silently ignored — this is
  the likely cause when a status won't stick. Check the allowed transitions with
  `GET /issues/{id}.json?include=allowed_statuses` (Redmine 5.1+).
- **Assignee.** Assign the issue to the **author (you)** — it is not automatic. Fetch your id once
  from `GET /users/current.json` and send `assigned_to_id: <that id>`.
- **Progress (`done_ratio`).** Send `done_ratio` explicitly (agree the value with the user, like hours). 
- **Verify the write.** After every create/update, **read the issue back** (`GET /issues/{id}.json`)
  and confirm `status`, `assigned_to`, and `done_ratio` match what you intended. If they don't,
  tell the user (usually a workflow-permission or done-ratio-mode issue) — never assume the write
  succeeded.

## JSON encoding
Ticket content is full of characters that break naive shell quoting — backticks, quotes, `$`, `|`,
emojis, newlines. Always build the request body as **proper JSON** rather than string-concatenating it.
Encoding the body from a file or heredoc through `python3` avoids the escaping traps:

```shell
python3 - "$REDMINE_URL" "$REDMINE_API_KEY" <<'PY'
import json, os, sys, urllib.request
url, key = sys.argv[1], sys.argv[2]
body = {"issue": {"notes": open("ticket.md").read(), "status_id": 2}}
req = urllib.request.Request(
    f"{url}/issues/14971.json",
    data=json.dumps(body).encode(),
    headers={"X-Redmine-API-Key": key, "Content-Type": "application/json"},
    method="PUT",
)
with urllib.request.urlopen(req) as r:
    print(r.status)
PY
```

Read the Markdown body from the drafted `.md` file (don't retype it inline) so what gets posted is
exactly what the user approved.
