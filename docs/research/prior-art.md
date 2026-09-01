# Prior art

Nobody is waiting for permission to solve this problem. Every working grant professional has already built something, and most of those somethings are worse than they deserve. This document names what people actually use, credits what deserves credit, and says specifically why each approach fails at the thing answerbank is for.

The honest summary: the enterprise sales world solved this exact problem fifteen years ago, priced the solution for enterprise sales teams, and nobody ported it down to a market where the median customer is one person with twelve clients and no software budget.

Verified 2026-08-30. Pricing figures are marked with their source and their verification status. Re-check any figure on the vendor's own page before it appears in public copy.

---

## 1. The Google Docs boilerplate folder

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

## 2. Foundant GrantHub and the funder-side platforms

**What it is.** Foundant Technologies sells Grant Lifecycle Manager to funders and GrantHub to grantseekers; GrantHub came to Foundant by acquisition. GrantHub includes a document and boilerplate store alongside deadline tracking and task assignment, and it is the closest commercial analog to answerbank in the nonprofit market. Blackbaud Grantmaking, Fluxx, Submittable, and SM Apply occupy the funder side of the same workflow.

**What it gets right.**
- Boilerplate storage genuinely is a feature, not an afterthought, and the vendor understands why it matters.
- Integrated with deadlines and tasks, which is where a development director actually lives.
- Multi-user, with roles, which small development shops need.

**Why it fails for the target user.**
- **Priced and shaped for a development department, not a solo consultant.** Pricing is quote-based rather than published. **VERIFY** any number before publishing one; the practitioner-reported range for grantseeker-side tooling in this class is roughly $1,000 to $2,500 per year, and that figure is unconfirmed.
- **Your text lives in their database.** Ending the relationship means an export, and the export is not a format you can keep working in.
- **No length variants and no verification interval** in the boilerplate store, which are the two features that actually change the work.
- **Client handoff is a problem, not a feature.** A consultant who wants to leave a client with their own narrative library cannot hand over a seat in the consultant's account.
- **No agent interface.** There is no way to point a model at the library.

**What answerbank borrows.** The confirmation that the need is real enough to be a shipped feature in a paid product. That is validation, not competition, and it deserves saying plainly.

---

## 3. Proposal and RFP response software from enterprise sales

**What it is.** Loopio, Responsive (formerly RFPIO), and Qvidian are answer-library products for enterprise sales teams responding to requests for proposal. They store approved answers, tag them by topic, assign each an owner and a review date, prompt the owner when the review date passes, and let a salesperson assemble a response from the library. Increasingly they also do AI-assisted assembly grounded in the library.

This is, structurally, exactly what answerbank does. It is worth being explicit that the idea is not original; the market it is aimed at is.

**What it gets right, and what answerbank takes directly.**
- **Content ownership per answer.** Every answer has a human who is responsible for its accuracy. This is where the `owner` front-matter field comes from.
- **Review cycles per answer, not per library.** Security questionnaire answers age in months; company history ages in years. This is exactly the `verify_every` design, and the RFP-response industry got there first.
- **Grounded assembly.** Draft from the approved library, not from a blank page or from a general-purpose model. The industry's whole value proposition is that the answer was already approved by somebody who knows.

**Why it does not fit.**
- **Price.** These are enterprise seats sold to revenue teams. Public pricing is generally unavailable and requires a sales conversation. **VERIFY**; do not publish a figure without a current source. Whatever the number, it is not a number a solo grant consultant reaches for.
- **Multi-tenancy is per company, not per client.** These products assume one organization's answers. A consultant needs twelve separate libraries with no bleed between them, and a hard guarantee that Casa Esperanza's program data never appears in a Riverkeeper draft.
- **Cloud-only.** Client narratives on a vendor's servers is a posture a consultant may not be able to agree to on a client's behalf.
- **Vocabulary and templates are sales, not philanthropy.** Nothing in a security questionnaire library helps you write a logic model.

**Credit where due.** If answerbank has an intellectual ancestor, it is this category. The design of per-answer review intervals is lifted from it deliberately.

---

## 4. Snippet managers and text expanders

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

## 5. Note systems: Obsidian, Notion, Logseq, plain folders

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

## 6. AI grant writing tools

**What it is.** A crowded and fast-moving category: Grantable, Grantboost, GrantAssistant, Fundwriter, and a steady stream of new entrants, plus general-purpose assistants used directly. Several of them do include an "answer library" or "knowledge base" feature, which is a sign the market has independently found the same insight.

**Why they are the wrong shape.**
- **The output is the product.** Generated prose is what you buy, and generated prose is what funders have learned to spot. Program officers increasingly say out loud that obviously-generated applications are declined on sight, and several funders have added disclosure requirements. **VERIFY** any specific funder policy before citing it publicly; this is moving quickly.
- **The library is a feed for the generator, not an asset you own.** You cannot take it with you and you cannot edit it in your own tools.
- **The verification problem is untouched.** A model given a stale board list generates a fluent, confident, wrong paragraph. Generation without a staleness signal amplifies the exact error this tool exists to prevent.
- **Cloud-only, with client narratives as training-adjacent input.** The terms vary and the questions are real.

**How answerbank differs, stated plainly.** It generates nothing. It exposes verified, dated, human-written language to a model the user already chose to run, through MCP, on their own machine. The output is grounded in an organization's own approved sentences, and anything past its verification date is flagged to the model before it drafts. That is a categorically different artifact from generic AI grant writing, and it is the defensible answer to the AI-slop objection rather than a denial of it.

---

## 7. Open-source and adjacent projects worth crediting

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
