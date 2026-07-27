---
name: redmine-ticket-writer
description: >-
  Write or edit Redmine tickets, issues, wiki pages, or status writeups so they render
  cleanly in Redmine and are ready to share. Use this whenever the user wants to produce,
  format, or clean up content destined for Redmine — a ticket, issue description, comment,
  wiki page, technical writeup, migration/handoff summary, or "documento" for the issue
  tracker — even when they just say "armá un ticket", "write this up for Redmine", "pasalo
  a formato redmine", "make a ticket so I can share it", or mention Redmine/Textile/CommonMark.
  Also triggers when someone calls the markup "markdown de redmine".
  DEFAULT is always Markdown (.md). Textile (.textile) ONLY when the user explicitly requests it
  or confirms their Redmine instance is in Textile mode. Also handles converting Textile → Markdown.
---

# Redmine ticket writer

**Default: Markdown. Always.**

Modern Redmine instances use Markdown (CommonMark/GFM). Write all content in Markdown unless
the user explicitly says their instance is in Textile mode or asks for Textile. When in doubt,
use Markdown and say so — it is almost certainly correct.

Textile is a secondary, fallback format. Only switch to it when explicitly requested. Do not
offer it as an alternative unprompted.

Output file: `.md` for Markdown (default), `.textile` only if Textile is requested.

Getting the format right matters: a `#` heading pasted into a Textile-mode Redmine shows up as
a literal `#`; an `h1.` in a Markdown-mode Redmine shows up as literal `h1.`.

## Workflow

1. **Get the real content first.** A ticket is only useful if it's accurate. Pull the actual
   facts from the conversation, the repo, git history, or by asking — versions, IDs, commands,
   file paths, ARNs, decisions, pending items. Do not pad with generic boilerplate; a precise,
   data-rich ticket is the whole point. If key facts are missing, ask rather than invent.

2. **Format: default to Markdown.** Only use Textile if the user says so. If you're unsure
   which mode an instance is in, default to Markdown. The tell-tale of a wrong guess is a
   previously pasted ticket that shows raw `#`/`h1.` markup.

3. **Choose a structure that fits the ticket type** (see "Structure patterns" below). Most
   technical tickets want: a one-line status/summary up top, a `{{>toc}}` if it's long, then
   sections. Short tickets (a bug, a single ask) don't need a TOC or many headings.

4. **Write it in the chosen syntax.** The sections below cover the 90% you'll reach for.
   For edge cases — alignment, spans, footnotes, attachments, cross-links, conversion gotchas —
   read the reference for your format:
   - Markdown → `references/redmine-markdown-syntax.md`
   - Textile → `references/textile-syntax.md`

5. **Save to a file** named after the ticket, e.g. `<slug>.md` (or `<slug>.textile`), in a
   location the user can reach. Tell them the path and that it's ready to copy-paste into Redmine.

6. **Offer to adjust** — structure, length, detail, or switching format if needed.

## Markdown syntax (primary — use this)

```markdown
# Title
## Section
### Subsection

**bold**   *italic*   ~~strikethrough~~   `inline code`

- bullet
  - nested bullet (2-space indent)
1. numbered

[link text](https://example.com)

> A blockquote — great for warnings/callout notes.

- [ ] task item (unchecked)
- [x] task item (checked)

{{>toc}}   Redmine macro — right-aligned TOC (works in Markdown and Textile)
```

**Tables** — separator row is mandatory:

```markdown
| Component | Version | Namespace |
|-----------|---------|-----------|
| mimir     | 5.8.0   | mimir     |
| loki      | 0.79.0  | loki      |
```

**Code blocks** — always add the language tag:

````markdown
```shell
kubectl -n mimir get pods
helmfile -e shared-monitoring -f 00-mimir/helmfile.yaml apply
```
````

Supported tags: `shell`, `bash`, `yaml`, `hcl`, `sql`, `python`, `ruby`, `json`, `diff`, `go`, `java`.

**Redmine-specific links** (work in both modes): issue links `#1234`, wiki links `[[WikiPage]]` /
`[[Page|label]]`, changesets `commit:abc123`, repo files `source:path/to/file`.

## Textile syntax (secondary — only if explicitly requested)

```textile
h1. Title   h2. Section   h3. Subsection
*bold*   _italic_   @inline code@
* bullet   ** nested
# numbered   ## nested
"link text":https://example.com
bq. blockquote
{{>toc}}
```

**Tables** — header cells use `|_.`; blank line required before table:

```textile
|_. Component |_. Version |
| mimir | 5.8.0 |
| loki  | 0.79.0 |
```

**Code blocks** — use `<pre><code class="lang">`; Textile markup is NOT processed inside:

```textile
<pre><code class="shell">
kubectl -n mimir get pods
</code></pre>
```

## Structure patterns

Pick the shape that matches the ticket. Render headings/tables/code with Markdown syntax (or
Textile if explicitly requested).

**Technical writeup / migration / handoff** (rich, multi-section):
- Title, then `{{>toc}}` if long
- One-line **Estado:** summary
- `Contexto` — a before/after table
- Numbered top-level sections per area, with prose, inventory tables, and command/config blocks
- `Pendientes / cleanup` — a numbered list
- `Fuera de alcance` — bullets noting whose responsibility each is

**Bug report:** `Resumen` · `Pasos para reproducir` (numbered) · `Esperado vs. actual` ·
`Entorno` (field/value table) · `Logs` (code block).

**Task / change request:** `Qué hay que hacer` · `Por qué` · `Criterios de aceptación`
(task-list checkboxes) · `Notas / riesgos` (callout).

## Gotchas that break rendering

**Markdown (the ones that bite most):**
- **Tables:** the separator row (`|---|---|`) is mandatory — missing it renders as plain text.
- **Pipe `|` inside a cell** needs escaping as `\|`.
- **Code blocks:** always add the language tag for syntax highlighting.
- **Blank lines:** required before lists and code blocks to avoid continuation-text treatment.
- **Nested lists:** use 2-space indentation, not tabs.
- **`#1234` autolinks to a Redmine issue.** To show a literal `#1234`, escape it as `\#1234` or wrap in `` `#1234` ``.
- **Single newlines don't break lines** in CommonMark — soft-wrapped prose collapses into one paragraph. Separate paragraphs with a blank line.
- **Backticks inside a fenced code block:** a line starting with ` ``` ` closes the fence. Use a `~~~` tilde fence or 4-backtick fence if the content contains triple-backticks.

**Textile (if used):**
- Header needs trailing dot + space: `h1. Title`, not `h1.Title`.
- Inline code is `@…@`, not backticks. No triple-backtick fences.
- Table header marker is `|_.` (not `|*`); blank line required before table.
- Lists nest by repeating the marker (`**`/`##`), not by indentation.
- Literal `@`, `#`, `|`, `*` in prose get interpreted — wrap in `<notextile>…</notextile>` or `==…==`.

**Mode mismatch (the worst one):** Markdown pasted into a Textile-mode Redmine (or vice versa)
renders as literal markup. Confirm the instance's mode before writing; default to Markdown when unsure.

## Pushing directly to Redmine (API + token)

The Redmine REST API is the preferred way to post content programmatically — validated and working
on `proyectos.mikroways.net`.

- `POST /issues.json` → create issue; `PUT /issues/:id.json` → update / add note
- Auth header: `X-Redmine-API-Key: <token>`
- Markdown content goes in `issue.description` (or `notes` for a comment/journal entry)
- Status/percentage: `status_id` (10 = "Validar por el cliente", 2 = "En curso") and `done_ratio`
- Time entries: `POST /time_entries.json` with `issue_id`, `spent_on`, `hours`, `activity_id`, `comments`
- Config: `REDMINE_URL` + API key in env vars — never hardcode the token, never commit it.

Always encode the body with proper JSON (use `python3 -c "import json,sys; ..."` or equivalent)
to avoid shell escaping issues with special characters in ticket content.
