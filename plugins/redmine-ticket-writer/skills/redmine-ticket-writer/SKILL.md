---
name: redmine-ticket-writer
description: >-
  Write or edit Redmine tickets, issues, wiki pages, or status writeups in Redmine's
  Textile formatting (h1. headers, @inline code@, <pre><code class="lang"> blocks, |_. tables,
  bq. callouts) so they render cleanly in Redmine and can be shared with colleagues. Use this
  whenever the user wants to produce, format, or clean up content destined for Redmine — a
  ticket, issue description, comment, wiki page, technical writeup, migration/handoff summary,
  or "documento" for the issue tracker — even when they just say "armá un ticket", "write this
  up for Redmine", "pasalo a formato redmine", "make a ticket so I can share it", or mention
  Redmine/Textile. Also triggers when someone calls Textile/Redmine markup "markdown de redmine"
  (it's actually Textile, not Markdown). Produces a .textile file ready to paste into Redmine.
---

# Redmine ticket writer (Textile)

Redmine's default wiki/issue markup is **Textile**, not Markdown — people often call it
"markdown de Redmine," but the syntax differs (`h1.` instead of `#`, `<pre><code>` instead of
fenced blocks, `|_.` table headers, `@code@` inline). Getting this right matters because a
ticket pasted in the wrong syntax renders as a wall of literal markup that colleagues can't read.

This skill produces a well-structured Textile document written to a `.textile` file, so the user
can copy-paste it straight into a Redmine ticket, comment, or wiki page.

## Workflow

1. **Get the real content first.** A ticket is only useful if it's accurate. Pull the actual
   facts from the conversation, the repo, git history, or by asking — versions, IDs, commands,
   file paths, ARNs, decisions, pending items. Do not pad with generic boilerplate; a precise,
   data-rich ticket is the whole point. If key facts are missing, ask rather than invent.

2. **Choose a structure that fits the ticket type** (see "Structure patterns" below). Most
   technical tickets want: a one-line status/summary up top, a `{{>toc}}` if it's long, then
   sections. Short tickets (a bug, a single ask) don't need a TOC or many headings.

3. **Write it in Textile.** Use the syntax in this file for the common cases; for anything less
   common (alignment, spans, footnotes, attachments, cross-links) consult
   `references/textile-syntax.md`.

4. **Save to a file** named after the ticket, e.g. `<slug>.textile`, in a location the user can
   reach (the working directory, or wherever the related work lives). Tell them the path and
   that it's ready to copy-paste into Redmine.

5. **Offer to adjust** — structure, length, level of detail. Mention they can ask for Markdown
   instead if their Redmine is configured in Markdown mode (some are).

## Core Textile syntax (the 90% you'll use)

```textile
h1. Title
h2. Section
h3. Subsection

*bold*   _italic_   +underline+   -strike-   @inline code@

* bullet
** nested bullet
# numbered
## nested numbered

"link text":https://example.com

bq. A blockquote — great for warnings/notes that should stand out.

{{>toc}}   right-aligned table of contents (use {{toc}} for left)
```

**Tables** — header cells use `|_.`; a blank line must precede the table:

```textile
|_. Component |_. Version |_. Namespace |
| mimir | 5.8.0 | mimir |
| loki  | 0.79.0 | loki |
```

**Code / command blocks** — use `<pre><code class="lang">`; the class enables syntax
highlighting (langs: `shell`, `bash`, `yaml`, `hcl`, `sql`, `python`, `ruby`, `json`, `diff`,
`go`, `java`, etc.):

```textile
<pre><code class="shell">
kubectl --context $GREEN -n mimir get pods
helmfile -e shared-monitoring -f 00-mimir/helmfile.yaml -l name=mimir apply
</code></pre>
```

> Note: inside `<pre>` blocks Textile is NOT processed, so `@`, `*`, `|` etc. are literal —
> exactly what you want for commands and config.

## Structure patterns

Pick the shape that matches the ticket. These are starting points, not rigid templates —
adapt to the content.

**Technical writeup / migration / handoff** (rich, multi-section):
```textile
h1. <Título>

{{>toc}}

p. *Estado:* <one-line status>

h2. Contexto
|_. Item |_. Antes |_. Después |
| ... | ... | ... |

h1. 1. <Área>
... prose, tablas de inventario, bloques <pre><code> con comandos/config ...

h1. Pendientes / cleanup
# item
# item

h1. Fuera de alcance
* item (de quién es la responsabilidad)
```

**Bug report:**
```textile
h2. Resumen
h2. Pasos para reproducir
# ...
h2. Esperado vs. actual
h2. Entorno
|_. Campo |_. Valor |
h2. Logs
<pre><code class="shell">...</code></pre>
```

**Task / change request:**
```textile
h2. Qué hay que hacer
h2. Por qué
h2. Criterios de aceptación
* [ ] ...
h2. Notas / riesgos
bq. ...
```

## Gotchas that break rendering

- **Tables:** there must be a blank line *before* the table, and the header marker is `|_.`
  (not `|*`). A pipe `|` inside a cell needs escaping or it splits the cell.
- **Headers need the trailing dot and a space:** `h1. Title`, not `h1.Title` or `# Title`.
- **Code blocks:** prefer `<pre><code class="lang">…</code></pre>`. A bare `<pre>` works but
  loses highlighting. Don't use triple-backtick fences — that's Markdown, and renders literally
  in Textile mode.
- **Lists:** `*`/`#` must be at the start of the line; nesting is `**`/`##`, not indentation.
  Note `#` at line-start is an *ordered list item* in Textile (not a heading) — that's correct.
- **Inline code is `@…@`**, not backticks.
- **Literal `@`, `#`, `|`, `*` appearing as content in prose** (e.g. the special characters being
  reported in a bug, a shell pipe in a sentence) get interpreted by Textile and mangle the output.
  Wrap them in `<notextile>…</notextile>` or `==…==`, or put them inside an `@code@` / `<pre>` span.
  Don't stack markers like `@@@` hoping they cancel out — that renders unpredictably.

When in doubt about a less-common construct, read `references/textile-syntax.md` before guessing —
a wrong guess renders as visible markup in the colleague's face.

## Future extension: push directly to Redmine (API + token)

Currently this skill only emits a `.textile` file for manual copy-paste — deliberately, so it
needs no credentials. A natural next step (left for whoever extends this) is to create/update
tickets directly via the Redmine REST API:

- `POST /issues.json` to create, `PUT /issues/:id.json` to update; auth via the
  `X-Redmine-API-Key` header (the user's Redmine API token).
- The Textile content goes in the `issue.description` field (or `notes` for a comment).
- Config needed: `REDMINE_URL` + API key (env vars or a small config file — never hardcode the
  token, and don't commit it).

If you add this, keep the file-output path as the default/offline mode and make the API push an
explicit opt-in, so the skill still works with no setup.
