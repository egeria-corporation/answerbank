# Prior art

What people already use to solve this problem, what each approach gets right, and where each one
fails a consultant maintaining current, verified language for a dozen organizations.

> **Note on scope.** Sections comparing named commercial products have been removed from this
> repository for now; that analysis is maintained internally. What remains is what the program's
> conventions actually require prior art to cover: the open source and community work this
> repository builds on, the incumbent practice it has to beat, and the upstream contribution
> posture. Nothing here names or prices a commercial product.

---

## The Google Docs boilerplate folder

**What it is.** A Drive folder per client containing `BOILERPLATE.docx`, or a single long document with headings, or a spreadsheet with a column of pasted paragraphs. This is what the overwhelming majority of the field uses. It should be treated with respect, because it works well enough that nothing has displaced it.

**What it gets right.**
- Zero setup, zero install, zero learning curve.
- Sharing with the client is one click, and the client already has an account.
- Revision history exists, sort of.
- Comments let a program director correct a number in place.

**Why it fails.**
- **No length variants.** The document has one mission statement, at whatever length it was last written. Every application at a different limit means retyping, and retyping is where drift enters.
- **No staleness signal.** Nothing in a Google Doc knows that the board list is nine months old. The failure is silent and it surfaces in front of a funder.
- **Revision history is unusable at the paragraph level.** "Which of these four paragraphs changed since we submitted to Packard in March" is not a question Drive answers.
- **Twelve clients means twelve folders and no cross-client view.** A consultant cannot ask "what is overdue anywhere."
- **Search is document-level.** Finding the 250-word evaluation paragraph means opening files and scrolling.
- **No structured export with counts.** Copying out of Docs and into a portal textarea drags formatting artifacts along.

**What answerbank borrows.** The insight that the artifact must be editable by a non-technical person in a tool they already have. Markdown files open in anything. Nobody has to learn a UI to fix a typo.

---

---

## Grant management and proposal software

**Removed from this repository for now.** The scope-relevant summary: a narrative library exists as
a secondary feature inside software sold to development departments and inside proposal management
software sold to enterprise sales teams. Both are priced and shaped for an organization with a team.
Nobody sells this as a standalone tool to a solo consultant, which is the gap. See `competitive.md`.

---

## AI drafting tools

**Vendor names removed for now.** The category argument still matters, because it shapes what
`answerbank` is:

Tools that generate proposal prose from a prompt produce text that is fluent, generic, and
increasingly recognizable to the people reading it. Funders have begun saying so out loud. The
output is the product, so there is no durable asset left behind — nothing that is *yours* next year.

`answerbank` inverts that. It stores what a human wrote and verified, tracks when it was last
confirmed, and serves the right variant. Pointed at the library, a model drafts in that
organization's own current language rather than in the average of the internet. That is a different
artifact, and it is the sharp answer to the AI-slop objection rather than an instance of it.

---

## Snippet managers and text expanders

**What it is.** TextExpander, Espanso (open source, GPL-3.0), Alfred snippets, Raycast snippets, and the snippet features built into editors.

**What they get right.**
- Instantaneous insertion anywhere, including into a browser textarea, which is where applications actually get submitted.
- Espanso in particular is open source, local-first, and cross-platform, and its philosophy is close to ours.

**Why they fail.**
- **Built for short strings.** A 1,000-word sustainability narrative is not a snippet, and snippet UIs get unusable at that length.
- **No metadata at all.** No source, no verification date, no owner. A snippet manager cannot tell you anything has gone stale, because it does not model the idea that text can be wrong.
- **No variants.** One trigger, one expansion.
- **No multi-client separation.** Snippets are global to the user, which is exactly wrong when a consultant serves twelve organizations.

**What answerbank borrows.** The `--copy` flag. Getting text onto the clipboard in one command is the interaction snippet tools proved people want, and it is often faster than any file-based workflow.

---

---

## Note systems: Obsidian, Notion, Logseq, plain folders

**What it is.** A serious minority of grant professionals keep boilerplate in Obsidian or Notion, often with a database of answers tagged by question type.

**What they get right.**
- Obsidian is Markdown files in a folder on your disk, which is exactly the storage model answerbank uses. A consultant already using Obsidian can point it at an answerbank library and it will simply work, which is a design goal rather than an accident.
- Notion databases give you properties, which is the closest thing to front matter most people encounter, and some practitioners really do have a `Last verified` column.

**Why they fail.**
- **All structure, no behavior.** A `Last verified` column does not compute anything. Nobody sorts by it on a Monday. The database has the data and does nothing with it, which is the gap between a filing system and a tool.
- **No word-count variants and no export with counts.**
- **Notion is cloud storage** with the ownership and portability questions that follow.
- **Setting it up correctly is a project.** Every practitioner builds a different schema, none of them are shareable, and there is no starter set encoding what a complete organizational narrative even is.

**What answerbank borrows.** Obsidian compatibility as a hard constraint on the format. Front matter is YAML because that is what Obsidian reads. Variants are H2 headings because those become outline entries. A library should be a usable Obsidian vault with no configuration.

---

---

## Open source and adjacent projects worth crediting

- **Espanso** (https://github.com/espanso/espanso, GPL-3.0). Local-first text expansion done well. Different problem, compatible philosophy.
- **Pandoc** (https://pandoc.org/, GPL-2.0-or-later). The obvious answer to document conversion, and deliberately not a dependency here because it requires a system install and the quickstart has to stay one command. Users who have it should absolutely use it; `answerbank export --format md` produces clean input for it.
- **python-docx** (MIT) and **ReportLab** (BSD 3-Clause) do the actual export work. Credited in `NOTICE`.
- **Obsidian's front-matter conventions.** Not a project we depend on, but a convention we conform to on purpose.
- **The Nonprofit Open Data Collective** (https://github.com/Nonprofit-Open-Data-Collective) and **GivingTuesday** (https://990data.givingtuesday.org/tool-repository/) do not intersect with this repository's code, since answerbank parses no 990 data. They are the backbone of the sibling repositories in this program and are credited there. Named here so that a reader arriving at answerbank first knows where the program's data work comes from.

---

## Contribution posture

Per the program conventions: fix upstream first, credit prominently, and never re-implement something a community project already does well just to own it.

Concretely for this repository:

- If we hit a `python-docx` bug while implementing export, the fix goes upstream before any local workaround, and the workaround is commented with a link to the upstream issue.
- If the front-matter conventions we settle on turn out to conflict with Obsidian's, we change ours rather than asking anyone to change theirs.
- The template gallery is the contribution this project makes back to the field. It is Apache 2.0, it is usable without installing anything, and it is meant to be copied. If a competing product wants to ship our thirty-question taxonomy, that is a good outcome for grant professionals and we should say so.

Anything shipped upstream from this repository gets recorded here with a link to the pull request.
