# Redmine Textile — full syntax reference

Consult this when SKILL.md's core syntax doesn't cover the construct you need. Source:
Redmine's official `RedmineTextFormattingTextile` / `wiki_syntax_detailed_textile` docs.

## Table of contents
- [Headings & anchors](#headings--anchors)
- [Inline styling](#inline-styling)
- [Paragraphs & alignment](#paragraphs--alignment)
- [Blockquotes](#blockquotes)
- [Lists](#lists)
- [Links & cross-references](#links--cross-references)
- [Tables (advanced)](#tables-advanced)
- [Code & syntax highlighting](#code--syntax-highlighting)
- [Images & attachments](#images--attachments)
- [Table of contents macro](#table-of-contents-macro)
- [Misc macros & escaping](#misc-macros--escaping)

## Headings & anchors
```textile
h1. Heading
h2. Subheading
h3. Sub-subheading
```
Redmine auto-assigns an anchor to each heading, so you can link to it: `"jump":#Heading`,
`[[Page#Heading]]`. Anchors are derived from the heading text.

## Inline styling
| Markup | Renders |
|--------|---------|
| `*bold*` | bold |
| `_italic_` | italic |
| `_*bold italic*_` | bold italic |
| `+underline+` | underline |
| `-strikethrough-` | strikethrough |
| `^superscript^` | superscript |
| `~subscript~` | subscript |
| `@inline code@` | monospace inline |
| `%{color:#f00}red%` | CSS-styled span |

## Paragraphs & alignment
- `p. text` — explicit paragraph
- `p>. text` — right-aligned
- `p=. text` — centered
- `p<>. text` — justified

## Blockquotes
Start a paragraph with `bq.`:
```textile
bq. This is quoted. Good for warnings, caveats, "landmine" notes that must stand out.
```

## Lists
```textile
* bullet
** nested bullet
*** deeper

# numbered
## nested numbered

# you can also mix:
#* a bullet under a numbered item
```
Markers must be at the start of the line. Nesting is by repeating the marker, NOT by indentation.

Checkbox-style lists (render as text, not interactive in description fields):
```textile
* [ ] todo
* [x] done
```

## Links & cross-references
```textile
"label":https://example.com           external link
"label":mailto:user@example.com       email
#1234                                  link to issue 1234
[[WikiPage]]                           wiki page
[[WikiPage|custom text]]               wiki page, custom label
[[WikiPage#Section]]                   anchor within a wiki page
[[project:WikiPage]]                   page in another project's wiki
commit:abc123                          link to a changeset
source:path/to/file                    link to a repo file
```

## Tables (advanced)
Basic:
```textile
|_. Header A |_. Header B |
| cell | cell |
```
Rules:
- A blank line MUST precede the `|...|` table block; nothing between the table keyword and rows.
- `|_.` marks a header cell.
- Cell alignment / style controls go right after the leading `|`, before the cell text, and
  order is fixed:
```textile
|=. centered | <. left | >. right | ^. top | ~. bottom |
|{background:#eee}. styled cell |
```
- Set table width before the table: `table{width:100%}.` on its own line, then the rows.
- A literal `|` inside a cell must be escaped (e.g. use `&#124;`) or it ends the cell.

## Code & syntax highlighting
Inline: `@code@`.

Block with highlighting (Rouge highlighter). The language name/alias is case-insensitive:
```textile
<pre><code class="ruby">
def hello
  puts "hi"
end
</code></pre>
```
Supported languages include: c, cpp (c++), csharp (c#, cs), css, diff, go (golang), groovy,
html, java, javascript (js), kotlin, objective_c (objc), perl (pl), php, python (py), r, ruby
(rb), sass, scala, shell (bash, zsh, ksh, sh), sql, swift, xml, yaml (yml), hcl/terraform.

A bare `<pre>...</pre>` (no `<code class>`) renders monospace without highlighting. Inside
`<pre>`, Textile markup is NOT interpreted — ideal for commands/config containing `@ * | _`.

## Images & attachments
```textile
!https://example.com/img.png!          external image
!>https://example.com/img.png!         right-floated
!attached_image.png!                   image attached to this page/issue, inline
!{width:300px}attached_image.png!       sized
```
Images can also be pasted from clipboard (Ctrl/Cmd-V) or dragged into the textarea to upload.

## Table of contents macro
```textile
{{toc}}     left-aligned TOC
{{>toc}}    right-aligned TOC
```
Generated from the headings on the page.

## Misc macros & escaping
- `{{macro(args)}}` — Redmine macros (e.g. `{{collapse}}`, `{{include(Page)}}`,
  `{{child_pages}}`), availability depends on the instance.
- To show literal Textile that would otherwise be interpreted, wrap it in `<notextile>...
  </notextile>` or use `==literal==`.
- A horizontal rule is `---` on its own line.
- Line breaks: a single newline becomes a `<br>` within a paragraph; a blank line starts a new
  paragraph.
