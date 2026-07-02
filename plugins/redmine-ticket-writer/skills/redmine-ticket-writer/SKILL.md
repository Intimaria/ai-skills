---
name: redmine-ticket-writer
description: >-
  Write or edit Redmine tickets, issues, wiki pages, or status writeups in the format your
  Redmine actually renders — GitHub-flavored Markdown (# headers, ```lang fences, GFM tables,
  > callouts) when the instance is in Markdown/CommonMark mode (the common default now), or
  Textile (h1., @code@, |_. tables, bq.) when it's in the older Textile mode. Use this whenever
  the user wants to produce, format, or clean up content destined for Redmine — a ticket, issue
  description, comment, wiki page, technical writeup, migration/handoff summary, or "documento"
  for the issue tracker — even when they just say "armá un ticket", "write this up for Redmine",
  "pasalo a formato redmine", "make a ticket so I can share it", or mention Redmine/Textile/CommonMark.
  Also triggers when someone calls the markup "markdown de redmine". Defaults to Markdown (.md);
  produces Textile (.textile) on request or when the Redmine is in Textile mode. Also handles
  converting existing Textile content to Markdown. Produces a file ready to paste into Redmine.
---

# Redmine ticket writer (Markdown or Textile)

A Redmine instance renders wiki/issue text with **one** configured text formatter, and the markup
differs sharply between them:

- **Markdown mode** — GitHub-flavored Markdown built on the [CommonMark](https://commonmark.org/help/)
  standard (`#` headers, fenced ```lang``` code, `| GFM | tables |`, `>` quotes). This is the
  common default on modern Redmine and is almost certainly what you want now.
- **Textile mode** — the older default (`h1.` headers, `@code@`, `|_.` tables, `bq.`). Still live
  on many long-running instances.

Getting the format right matters because a ticket written for the wrong mode renders as a wall of
literal markup that colleagues can't read. **A `#` heading in a Textile-mode Redmine shows up as a
literal `#`; an `h1.` in a Markdown-mode Redmine shows up as literal `h1.`.**

This skill produces a well-structured document written to a file (`.md` by default, `.textile` on
request), so the user can copy-paste it straight into a Redmine ticket, comment, or wiki page.

## Workflow

1. **Get the real content first.** A ticket is only useful if it's accurate. Pull the actual
   facts from the conversation, the repo, git history, or by asking — versions, IDs, commands,
   file paths, ARNs, decisions, pending items. Do not pad with generic boilerplate; a precise,
   data-rich ticket is the whole point. If key facts are missing, ask rather than invent.

2. **Pick the format.** Default to **Markdown** (`.md`) — modern Redmine runs in Markdown/CommonMark
   mode. Use **Textile** (`.textile`) when the user says their Redmine is in Textile mode, asks for
   Textile, or is editing content that's already Textile. If you're genuinely unsure which mode an
   instance is in, say so and default to Markdown; the tell-tale of a wrong guess is a previously
   pasted ticket that shows raw `#`/`h1.` markup.

3. **Choose a structure that fits the ticket type** (see "Structure patterns" below). Most
   technical tickets want: a one-line status/summary up top, a `{{>toc}}` if it's long, then
   sections. Short tickets (a bug, a single ask) don't need a TOC or many headings.

4. **Write it in the chosen syntax.** The comparison table below covers the 90% you'll reach for.
   For anything less common — alignment, spans, footnotes, attachments, cross-links, and the
   edge cases that bite during Textile→Markdown conversion — read the reference for your format:
   - Markdown → `references/redmine-markdown-syntax.md`
   - Textile → `references/textile-syntax.md`

5. **Save to a file** named after the ticket, e.g. `<slug>.md` (or `<slug>.textile`), in a location
   the user can reach. Tell them the path and that it's ready to copy-paste into Redmine.

6. **Offer to adjust** — structure, length, detail, or switching format (e.g. "same ticket in
   Textile" / "convert this to Markdown").

## Core syntax — Markdown vs. Textile

The common constructs, side by side. Markdown is the default column.

| Need | Markdown (default) | Textile |
|------|--------------------|---------|
| Heading L1 / L2 | `# Title` / `## Section` | `h1. Title` / `h2. Section` |
| Bold / italic | `**bold**` / `*italic*` | `*bold*` / `_italic_` |
| Strikethrough | `~~struck~~` | `-struck-` |
| Inline code | `` `code` `` | `@code@` |
| Bullet / nested | `- item` / indent 2 spaces | `* item` / `**` |
| Numbered / nested | `1. item` / indent | `# item` / `##` |
| Link | `[label](https://x)` | `"label":https://x` |
| Blockquote / callout | `> note` | `bq. note` |
| Task list | `- [ ]` / `- [x]` | `* [ ]` / `* [x]` |
| Horizontal rule | `---` on its own line | `---` on its own line |

**Code / command blocks** — highlighting uses the same [Rouge](https://github.com/rouge-ruby/rouge)
languages in both modes (`shell`, `bash`, `yaml`, `hcl`, `sql`, `python`, `ruby`, `json`, `diff`,
`go`, `java`, …):

````text
Markdown:                      Textile:
```shell                       <pre><code class="shell">
kubectl -n mimir get pods      kubectl -n mimir get pods
```                            </code></pre>
````

**Tables** — Markdown needs a `|---|---|` separator row under the header; Textile marks header cells
with `|_.`. Both need a blank line before the table:

```text
Markdown:                          Textile:
| Component | Version |            |_. Component |_. Version |
|-----------|---------|            | mimir | 5.8.0 |
| mimir     | 5.8.0   |            | loki  | 0.79.0 |
| loki      | 0.79.0  |
```

**Redmine-specific markup that works in *both* modes** (formatter-independent): issue links
`#1234`, wiki links `[[WikiPage]]` / `[[Page|label]]`, changesets `commit:abc123`, repo files
`source:path/to/file`, and the TOC macro `{{>toc}}` (right) / `{{toc}}` (left). Images/attachments
*do* differ: Markdown `![alt](image.png)`, Textile `!image.png!`.

## Structure patterns

Pick the shape that matches the ticket; these are section skeletons, not rigid templates. Render
the headings/tables/code with your chosen format's syntax (see the table above).

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

**Cross-cutting — the one that bites hardest:** *mode mismatch.* Markdown pasted into a Textile-mode
Redmine (or vice versa) renders as literal markup. Confirm the instance's mode before writing; when
unsure, default to Markdown and say so.

**Markdown (CommonMark/GFM) — the ones that surface when converting Textile → Markdown:**
- **Backticks inside a code block** (e.g. a stacktrace `` EventDefinition`2 ``, an Oracle string
  `` Trace`1 ``): a stray backtick mid-line is fine inside a ``` ``` fence, but if the content has a
  line that *starts* with three-or-more backticks it closes the fence early. When code contains
  backticks, use a `~~~` tilde fence (or a 4-backtick fence), or keep the original `<pre>…</pre>`
  HTML — Redmine renders raw HTML in Markdown mode.
- **Nested inline formatting inside a link** (e.g. Textile `" *@1.11.1@* ":url` = bold + inline code
  as the link text): CommonMark *does* allow it — write `[**`1.11.1`**](url)` — but the delimiter
  order matters and naive converters mangle it. Put the code span inside the emphasis, inside the
  link brackets.
- **`#1234` autolinks to an issue.** To show a literal `#1234` (a count, an anchor) escape it `\#1234`
  or wrap it in `` `#1234` ``.
- **Single newlines do NOT break lines in CommonMark** — soft-wrapped prose collapses into one
  paragraph. Separate paragraphs with a blank line; force a hard break with two trailing spaces or a
  backslash. (This is the opposite of Textile.)

**Textile:**
- Tables need a blank line *before* them and `|_.` (not `|*`) for header cells.
- Headers need the trailing dot and a space: `h1. Title`, not `h1.Title`.
- Code blocks: prefer `<pre><code class="lang">…</code></pre>`; never use ``` ``` ``` fences (that's
  Markdown and renders literally). Inside `<pre>`, all Textile markup is literal.
- Lists nest by repeating the marker (`**`/`##`), not by indentation.
- Literal `@ # | *` in prose get interpreted — wrap in `<notextile>…</notextile>` or `==…==`.

When in doubt about a less-common construct, read the reference for your format before guessing —
a wrong guess renders as visible markup in the colleague's face.

## Future extension: push directly to Redmine (API + token)

Currently this skill only emits a file for manual copy-paste — deliberately, so it needs no
credentials. A natural next step (left for whoever extends this) is to create/update tickets
directly via the Redmine REST API:

- `POST /issues.json` to create, `PUT /issues/:id.json` to update; auth via the
  `X-Redmine-API-Key` header (the user's Redmine API token).
- The formatted content goes in the `issue.description` field (or `notes` for a comment) — the same
  Markdown/Textile you'd paste manually. The instance's formatter setting decides how it renders, so
  the format still has to match the instance's mode.
- Config needed: `REDMINE_URL` + API key (env vars or a small config file — never hardcode the
  token, and don't commit it).

If you add this, keep the file-output path as the default/offline mode and make the API push an
explicit opt-in, so the skill still works with no setup.
