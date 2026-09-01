# Non-goals

`answerbank` does one job: it keeps an organization's reusable narrative in plain files on your disk, and it tells you when that narrative has gone stale.

Everything below has been asked for, or will be, and the answer is no. This document exists so that the answer is on record before the request, rather than negotiated one feature at a time until the tool is unusable.

A tool that does one thing gets opened every day. A platform that does nine things gets evaluated once, and then not adopted.

---

## This is not a CRM

No contacts. No funder records. No relationship history, no "last touched" dates, no notes about which program officer prefers a phone call. No pipeline, no stages, no probability-weighted forecast.

Those are real needs. They are not this tool's needs. A narrative library that also tracks relationships becomes a database with a schema, which becomes a sync problem, which becomes a service, which becomes a login. The distance from "let me add a funders table" to "please create an account" is shorter than it looks.

If you want pipeline tracking, `grantdesk` in this same program is where that work belongs, and it can read this library.

## This is not a proposal editor

`answerbank` does not open a document for you to write in. It has no rich text editor, no side-by-side view, no track changes, no comment threads, no collaborative cursors.

Your editor is your editor. The files are Markdown. Use VS Code, use Obsidian, use TextEdit, use vim, use whatever you already have open. The tool's job ends at handing you the right text at the right length.

This also means no web UI, no desktop app, and no Electron.

## This is not a cloud service

There is no hosted version. There is no account, no login, no sync, no team workspace, no shared library, no permissions model, no seats.

Your library is a directory. If you want it on two machines, use Dropbox, iCloud, Syncthing, or a git remote, all of which do that job better than we would and none of which require us to hold your clients' unpublished narratives on our servers.

Multi-user editing with conflict resolution is a genuinely hard problem that git already solved for text files. We are not going to solve it worse.

`answers.opengrants.io` is a static website of public template content. It is not a place your library goes.

## This is not an AI writer

`answerbank` contains no model, calls no model, and generates no prose. `answerbank get` returns text a human wrote. `answerbank export` formats text a human wrote.

The MCP server exposes your verified language *to* a model that you chose to run. That is the opposite arrangement, and the distinction is the entire product thesis. Generic AI grant writing produces text that funders have learned to recognize and have started to reject. Grounding a model in an organization's own approved, dated language is the version of this that survives contact with a review panel.

So: no `answerbank write`. No `answerbank improve`. No `answerbank rewrite --tone warmer`. No embedded API key for any model provider. The tool never edits your answer text, with the single exception of `verify` appending a dated line to the `## notes` section, which is not answer text.

## This is not a submission tool

It does not log into Submittable, Foundant, Fluxx, SM Apply, Grants.gov, or anything else. It does not fill in web forms, does not upload attachments, and does not click submit.

Portal automation means storing your clients' portal credentials, which is a security posture we are not willing to take on and you should not want us to have. It also breaks every time a vendor ships a UI change, which is roughly monthly across the field.

Export a DOCX, or copy to the clipboard, and paste it yourself. The paste is thirty seconds. The credential store is forever.

## This is not a compliance or eligibility checker

`answerbank` never tells you whether an organization qualifies for anything. It does not check registration status, does not verify tax-exempt standing, does not read a NOFO and score your fit.

`grantcheck` in this program does the eligibility work, against public IRS and SAM.gov data, with the disclosures that work requires.

The one temporal claim this tool makes is that you said you would re-verify an answer by a certain date and you have not. That is a claim about your own calendar, not about the world.

## This is not a version control system

Answer files are text in a directory. `git init` gives you history, diffs, branches, blame, and remote backup, and it gives you all of that better than a bespoke implementation would.

So there is no `answerbank history`, no `answerbank revert`, no built-in snapshots, and no undo stack. The format is designed so that `git diff` on an answer file is readable by a human who does not know git, which is why variants are headings inside one file and why front matter keys have a fixed order.

## This is not a template marketplace

The starter gallery is a curated set of the standard questions with guidance, maintained in this repository and reviewed by maintainers. It is not a store, not user-generated at scale, not rated, not searchable by author, and nobody sells anything through it.

Curation is the value. A gallery of four thousand uploaded mission statements would be worth less than thirty good ones with an explanation of what makes them good.

## Not a database, ever

No SQLite, no DuckDB, no index file, no `.answerbank.cache`, no lockfile. The directory of Markdown files is the complete state of the system.

This is a hard architectural constraint, not a current limitation. A cache is a thing that goes wrong silently. If `answerbank stale` ever disagrees with what is in the files, the tool has failed at the only job it has. Reading a few hundred small Markdown files takes milliseconds, and a consultant with twelve clients has a few hundred files, not a few hundred thousand.

## Not a scheduler or a notifier

No daemon, no background process, no email, no Slack, no push notification, no calendar integration.

`answerbank stale` exits `1` when something is overdue. Every scheduler you already have (cron, launchd, Task Scheduler, a GitHub Action, a reminder in your own calendar) can act on that. Building a notification system means building a mail sender, which means an SMTP configuration, which means either a service or a support burden. The exit code is enough.

## Not a semantic search engine

`answerbank search` is keyword ranking over titles, tags, and body text. It runs locally, produces the same result every time, and needs no model, no embeddings, and no vector store.

Semantic search over three hundred documents that one person wrote is a solution to a problem nobody has. If you want a model to find the right answer, the MCP server hands the model the whole list and lets it decide, which works better and costs nothing.

## Not multilingual answer management

Answers are text and the tool is encoding-agnostic, so a Spanish-language answer file works fine today. What the tool will not do is manage translation pairs, track which language version is authoritative, flag when the English changed but the Spanish did not, or call a translation API.

If your organization maintains parallel narratives, use a tag (`lang:es`) and separate ids (`mission-es`). That covers it without a schema change.

---

## How to argue with this list

These are decisions, not laws. If you think one is wrong, open an issue that makes the case in terms of the two hard rules:

1. Does the quickstart stay one command, with no account, no key, and no database?
2. Does it stay one job?

An argument that clears both is worth having. An argument that starts "it would only be a small addition" is the one this document exists to answer.
