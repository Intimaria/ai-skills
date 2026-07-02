# Redmine Markdown — full syntax reference (CommonMark / GFM)

Consult this when SKILL.md's core table doesn't cover the construct you need, or when converting
existing Textile content to Markdown. Redmine's "Markdown" formatter follows the
[CommonMark spec](https://commonmark.org/help/) plus the GitHub-flavored extensions (tables,
strikethrough, task lists, autolinks), and layers Redmine's own wiki/issue linking on top.

## Table of contents
- [Headings](#headings)
- [Inline styling](#inline-styling)
- [Paragraphs & line breaks](#paragraphs--line-breaks)
- [Blockquotes](#blockquotes)
- [Lists & task lists](#lists--task-lists)
- [Links & Redmine cross-references](#links--redmine-cross-references)
- [Tables](#tables)
- [Code & syntax highlighting](#code--syntax-highlighting)
- [Images & attachments](#images--attachments)
- [Macros & TOC](#macros--toc)
- [Escaping & literal characters](#escaping--literal-characters)
- [Textile → Markdown conversion gotchas](#textile--markdown-conversion-gotchas)

## Headings
```markdown
# Heading 1
## Heading 2
### Heading 3
```
Use ATX (`#`) headings; leave a space after the `#`. Redmine derives an anchor from the heading text
so `{{>toc}}` and `[[Page#Heading]]` links resolve.

## Inline styling
| Markup | Renders |
|--------|---------|
| `**bold**` | bold |
| `*italic*` or `_italic_` | italic |
| `***bold italic***` | bold italic |
| `~~strikethrough~~` | strikethrough (GFM) |
| `` `inline code` `` | monospace inline |

CommonMark has **no underline** syntax. If you truly need it, use raw HTML `<u>text</u>` (Redmine
renders it) — but usually bold is the better choice.

## Paragraphs & line breaks
- Separate paragraphs with a **blank line**.
- A **single newline is NOT a line break** in CommonMark — soft-wrapped lines join into one
  paragraph. This is the opposite of Textile and of Redmine's older Redcarpet formatter, and is the
  most common surprise when moving prose across.
- Force a hard break with **two trailing spaces** at the end of the line, or a trailing backslash `\`.

## Blockquotes
```markdown
> A quoted line — good for warnings, caveats, notes that must stand out.
> Second line of the same quote.
```
Blank-line-separated `>` blocks are separate quotes; consecutive `>` lines are one quote.

## Lists & task lists
```markdown
- bullet
  - nested bullet (indent 2–4 spaces under the parent)
    - deeper

1. numbered
2. second
   1. nested numbered

- [ ] todo
- [x] done
```
Nesting is by **indentation** (unlike Textile's repeated markers). Leave a blank line before a list
if it follows a paragraph. Task lists (`- [ ]`/`- [x]`) are a GFM extension and render as checkboxes.

## Links & Redmine cross-references
```markdown
[label](https://example.com)          external link
[label](mailto:user@example.com)      email
<https://example.com>                  autolinked bare URL (GFM)
#1234                                  link to issue 1234 (Redmine, both modes)
[[WikiPage]]                           wiki page (Redmine)
[[WikiPage|custom text]]               wiki page, custom label
[[WikiPage#Section]]                   anchor within a wiki page
[[project:WikiPage]]                   page in another project's wiki
commit:abc123                          link to a changeset
source:path/to/file                    link to a repo file
```
The `#1234`, `[[…]]`, `commit:`, and `source:` forms are Redmine features that work regardless of
formatter. Note `#1234` **autolinks**: to show it literally, escape it (`\#1234`) or code-span it.

## Tables
GitHub-flavored pipe tables. A blank line must precede the table, and a separator row of dashes goes
directly under the header:
```markdown
| Component | Version | Namespace |
|-----------|---------|-----------|
| mimir     | 5.8.0   | mimir     |
| loki      | 0.79.0  | loki      |
```
- Alignment via colons in the separator row: `:---` (left), `:---:` (center), `---:` (right).
- A literal `|` inside a cell must be escaped as `\|`.
- Cells can contain inline markup (`**bold**`, `` `code` ``, links) but not block elements.

## Code & syntax highlighting
Inline: `` `code` ``. To show a backtick *inside* an inline span, wrap with more backticks:
`` `` `x` `` `` renders `` `x` ``.

Fenced block with highlighting (same Rouge languages as Textile):
````markdown
```shell
kubectl --context $GREEN -n mimir get pods
helmfile -e shared-monitoring -f 00-mimir/helmfile.yaml -l name=mimir apply
```
````
Supported languages include: c, cpp, csharp, css, diff, go, groovy, html, java, javascript,
kotlin, objective_c, perl, php, python, r, ruby, sass, scala, shell (bash/zsh/sh), sql, swift,
xml, yaml, hcl/terraform. Inside a fenced block, Markdown is not interpreted — text is literal.

**When the code itself contains backticks** (stacktraces like `` EventDefinition`2 ``, Oracle
strings like `` Trace`1 ``): a backtick in the *middle* of a line is harmless, but a line that
*begins* with three-or-more backticks would close a ``` ``` fence early. Two safe options:
- Use a **tilde fence**, which backticks can't close: `~~~shell … ~~~` (add a language after `~~~`
  for highlighting).
- Use a **longer backtick fence** than any run inside — 4 backticks ```` ```` ```` closes only on a
  line of ≥4 backticks.
- Or keep the original **`<pre>…</pre>` HTML**; Redmine renders raw HTML in Markdown mode and treats
  everything inside as literal (closest to Textile's `<pre>` behavior).

## Images & attachments
```markdown
![alt text](https://example.com/img.png)     external image
![](attached_image.png)                       image attached to this issue/page, inline
```
Redmine resolves a bare filename to an attachment on the same issue/wiki page. Sizing beyond
plain Markdown needs raw HTML (`<img src="attached_image.png" width="300">`). Images can also be
pasted from the clipboard or dragged into the textarea to upload.

## Macros & TOC
Redmine macros are formatter-independent and work in Markdown mode:
```markdown
{{>toc}}        right-aligned table of contents (from the headings)
{{toc}}         left-aligned
{{collapse(Show details)
...content...
}}
{{include(OtherWikiPage)}}
{{child_pages}}
```
Availability of specific macros depends on the instance.

## Escaping & literal characters
- Backslash-escape a Markdown metacharacter to show it literally: `\* \_ \# \` \[ \] \| \\`.
- To show a literal `#1234` without autolinking to an issue: `\#1234` or `` `#1234` ``.
- For a block of literal text, a fenced code block or inline code span is the most robust —
  everything inside is verbatim.
- Raw HTML is allowed and rendered by Redmine, which is handy for the few things Markdown lacks
  (`<u>`, `<sub>`, `<sup>`, sized `<img>`, or a fully-literal `<pre>`).

## Textile → Markdown conversion gotchas
These are the failure modes seen migrating real Redmine tickets from Textile to Markdown. A bulk
converter often mangles them; fix by hand.

| Textile source | Correct Markdown | Why it breaks |
|----------------|------------------|---------------|
| `" *@1.11.1@* ":https://x` (bold + inline code as link text) | `[**`1.11.1`**](https://x)` | Nested emphasis + code span inside link text is valid CommonMark, but converters drop or reorder the delimiters. Code span goes *inside* the emphasis, inside the `[…]`. |
| `<pre>…EventDefinition`2…</pre>` (code containing a backtick) | keep `<pre>…</pre>`, or `~~~ … ~~~`, or a 4-backtick fence | A naive convert to a ``` ``` fence can be closed early by backtick runs in the content. |
| `<pre>(DESCRIPTION = …) Trace`1</pre>` (Oracle connection string) | same as above — tilde fence or keep `<pre>` | Parentheses are fine; the stray backtick is the hazard. |
| A count/anchor written as `#4` | `\#4` or `` `#4` `` | In Markdown mode `#4` autolinks to issue 4. |
| Prose hard-wrapped every ~80 cols | reflow into blank-line-separated paragraphs | CommonMark joins single-newline lines; Textile didn't. |
| `bq.` multi-line quote | prefix every line with `> ` | Textile's `bq.` covers the whole paragraph; Markdown needs `>` per line. |
| `|_. Header |` table | header row + `|---|` separator row | Markdown has no header-cell marker; the separator row makes row 1 the header. |

When a snippet mixes several of these (a link with formatted text inside a table cell, say), build it
up from the inside out and, if it's fiddly, fall back to a small piece of raw HTML rather than
fighting the delimiters.
