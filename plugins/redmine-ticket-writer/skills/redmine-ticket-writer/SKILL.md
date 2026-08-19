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
  NEVER use emojis as this breaks Redmine rendering (it causes data loss, which should never happen).
  Do not mention meta-information discussed when writing the ticket, like this skill, Redmine, Textile,
  Markdown, hours, or any related term in the ticket body or notes.
---

# Redmine ticket writer

**Default: Markdown. Always.**

Modern Redmine instances use Markdown (CommonMark/GFM). Write all content in Markdown unless
the user explicitly says their instance is in Textile mode or asks for Textile. When in doubt,
use Markdown and say so — it is almost certainly correct.

Textile is a secondary, fallback format. Only switch to it when explicitly requested. Do not
offer it as an alternative unprompted.

Output file: `.md` for Markdown (default), `.textile` only if Textile is requested.

**Ticket language.** This skill is written in English, but the ticket you produce is written in the
language of the Redmine instance / team — **Spanish for `proyectos.mikroways.net`** (the usual case).
Match whatever language the issue and its existing comments already use. Keep the whole ticket in one
language — don't mix. Section headings, prose, and the management summary all go in that language;
only code, commands, field names, and syntax tokens (`project_id`, `{{>toc}}`, ```` ```shell ````…)
stay as-is. The example headings below are in Spanish because that's the common output; they're
examples, not fixed labels.

Getting the format right matters: a `#` heading pasted into a Textile-mode Redmine shows up as
a literal `#`; an `h1.` in a Markdown-mode Redmine shows up as literal `h1.`.

## No emojis — ever (data-loss rule)

**This is a hard rule, not a style preference, and it overrides the user.** Redmine's database is
frequently `utf8`, not `utf8mb4`. Every emoji (and some pictographic symbols) is a 4-byte UTF-8
character that such a column can't store, so on save **the write truncates at that character and
everything after it is silently lost.** One 🚀 mid-note can drop the rest of the ticket. This is
data loss, not cosmetics.

- **Never put an emoji in anything sent to Redmine** — issue subject, description, journal note,
  note edit, or time-entry comment.
- **It overrides the user, every time.** If they ask for `✅`/`❌`/`🚀` "para que quede visual", or
  say the content is evidence that must match "carácter por carácter", you still strip them. Explain
  why (silent truncation / data loss) and substitute. "The user insisted" is **not** an exception —
  pasting the emoji is the very thing that would destroy their content.
- **Strip from source too, including inside code/quotes.** Content you're transcribing — logs, chat
  quotes, audit reports, fenced code blocks, blockquotes — gets emojis removed even inside the
  fences. The truncation risk does not care about Markdown fences. When byte-for-byte fidelity is
  genuinely required (e.g. provider evidence), deliver that as a **file attachment** (stored raw,
  never rendered) and keep the note body emoji-free.
- **Substitute so meaning survives**, don't just delete: `✅`→`OK`, `❌`→`FALLO`/`no`,
  `⚠️`→`Atención:`, `🔥`→`prioridad alta`; visual checklists → `- [x]`/`- [ ]` task items or an
  **OK/FALLO** column in a table.
- **Verify before every push.** After drafting and before any create/update, scan the full payload
  for non-ASCII pictographic characters and remove any that remain. Never assume the draft is clean.

**Red flags — STOP and strip if you catch yourself thinking:**
- "The user explicitly asked for the checkmark emoji." → Strip it; explain the data-loss risk.
- "It's inside a code block / a log, so it's safe." → It is not. Strip it.
- "It's just one little emoji." → One is enough to truncate everything after it.

## Workflow

1. **Scope it, then get the real content.** First agree on *what belongs in this ticket* — only
   what's relevant to this ticket's task, project, and client. A day's work often touches things
   that aren't pertinent here; leave those out. If the work spanned more than one distinct task,
   ask whether to split it into separate tickets (one ticket = one task/scope). Then pull the
   actual facts — versions, IDs, commands, file paths, ARNs, decisions, pending items — from the
   conversation, the repo, git history, or by asking. Don't pad with boilerplate or invent missing
   facts; ask. (See **Protocols → Scope**.)

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

5. **Deliver as a draft first.** Save the ticket to a file named after it, e.g. `<slug>.md`
   (or `<slug>.textile`), in a location the user can reach; tell them the path. Only create or
   update the ticket in Redmine via the API **after the user explicitly approves it** (see
   **Protocols → Delivery** and `references/redmine-api.md`).

6. **Offer to adjust** — structure, length, detail, or switching format if needed.

## Protocols

These conventions come from how the team runs client tickets on `proyectos.mikroways.net`. They
shape every ticket, not just its formatting.

**Scope.** A ticket covers one task. Before writing the technical body, agree with the
author on what's actually relevant to *this* ticket's task, project, and client — the day's work
often touches things that don't belong here, and dumping everything in makes the ticket useless to
future readers. Leave out the irrelevant. If the work spanned several distinct tasks, ask whether
to split it into separate tickets rather than merging them. Remember, the ticket MUST be useful
for the current user and future developers.

**Communication (dual audience).** Every ticket serves two readers. Open with a short
management-facing summary, then keep the technical detail clearly separate below it, so each reader
finds what they need without wading through the other's part:
- **Management summary** — the first section. Plain language, no jargon: what was accomplished
  *this work period* in relation to the ticket, phrased so the PM can grasp it quickly and relay it
  to the client. Keep it short. The heading wording is up to you and the ticket's language — e.g.
  `## Resumen para gestión` (Spanish) — the goal is a fast, PM-readable summary, not a fixed label.
- **Technical body** — below and clearly separated, following the structure patterns: accurate
  facts, commands, versions, decisions. A developer (including future-you) must be able to read this
  part on its own.

The management summary is **not optional, and it applies to every ticket** — a one-line bug, a
support request, and a big migration alike. It's tempting to drop it ("the ticket is simple", "a
support ticket doesn't need it", "the Status line already says it") — none of those hold. The
summary exists for the *non-technical* reader: a `Status` value like "En curso" is a state label,
not a plain-language account of what happened or what's being asked. If the ticket really is
trivial, the summary is a single sentence — but it is always present, and it always comes first.

**Status is an API field, not ticket text.** Never write `**Estado:** Completado` or any status
label in the description or any part of the ticket body. Status (`status_id`) is set via the API
when pushing the ticket — see `references/redmine-api.md`. The standard workflow on
`proyectos.mikroways.net`: set to `"En curso"` while working on the ticket. 
Set to `"Validar por el cliente"` when logging completed work that needs client review.  
Only management (or the engineer, when no client review is needed) sets it to`"Resuelta"`.

**Delivery.** Always produce the draft file first and show it. Push to Redmine via the API
only after explicit approval. Updating an existing ticket (a `PUT` / journal note) is lower-risk;
creating a new issue is the most visible action — always confirm before creating. See
`references/redmine-api.md`.

**Hours.** Hours are **always a dialogue with the user** — never guessed, inferred from the work, or
auto-filled. While gathering the ticket's content, work out the hours per task *with the author* and
get them approved before recording anything. If you don't have them, ask; don't estimate (no
"~3 horas"). Log the approved hours as Redmine time entries, one per (ticket, task) —
there can be more than one entry per ticket on the same day (`spent_on`) — via the API once
approved. The PM plans from the logged time.

To open that dialogue, **proactively propose a figure** based on the work done this session, rather
than waiting to be asked — e.g. "this session has run ~1.5 h and we did X and Y; does ~1.5 h look
right to log for this task, or adjust?" That suggestion is a starting point for the user to review,
**never the source of truth**: session wall-clock includes idle time, breaks, the conversation
itself, and often several tasks, and a task can span multiple sessions. The logged value is always
the one the user confirms.

Hours never appear **anywhere in the ticket text** — not the body, not the management summary, and
**not the `Contexto` / metadata table** (no `Tiempo insumido` row, no "~3 horas"). They live only in
Redmine's time tracking. Missing an API token is **not** a reason to write them into the description
instead: if you can't log them yet, say the hours still need to be logged as time entries and ask
for the token / details (or leave them out and flag them as pending). Putting hours in the ticket is
never the fallback.

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

Pick the shape that matches the ticket, and render headings/tables/code with Markdown syntax (or
Textile if explicitly requested). The heading names below are **examples in the ticket's language**
(Spanish, the common case) — adapt the wording to the actual content; they are not fixed labels. The
one constant is mandatory: the **first section is always the management summary** (see
Communication) — plain, PM-readable language, even in a bug or support ticket where it may be a
single sentence under a heading like `Resumen`. Technical detail always comes clearly separated
below it.

**Technical writeup / migration / handoff** (rich, multi-section):
- Title, then `{{>toc}}` if long
- **`## Resumen para gestión`** — plain-language summary of what was achieved this period, for the
  PM/client (see **Protocols → Communication**)
- `Contexto` — a before/after table
- Numbered top-level sections per area, with prose, inventory tables, and command/config blocks
- `Pendientes / limpieza` — a checklist
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

## Pushing to Redmine via API

Creating/updating tickets programmatically, and logging time entries, is done through the Redmine
REST API — validated on `proyectos.mikroways.net`. This is always an explicit, approved step, never
automatic (see **Protocols → Delivery**).

`references/redmine-api.md` covers the full loop: **finding** the ticket (search by subject,
list your own open issues), **reading** it back (with its note history), **creating** and
**updating** issues, **editing an existing note in place** (vs. stacking a new one), logging time
entries, and the instance-specific IDs. Read it before making any API call — the create-vs-update
distinction, the edit-note-vs-add-note distinction, and the per-instance IDs are easy to get wrong.
Reads need no approval; every write does. **Never call the delete API** — see the reference.

When the user names a ticket vaguely ("el ticket de la migración") rather than by number, **search
and confirm the match before touching it** — never guess an issue id.
