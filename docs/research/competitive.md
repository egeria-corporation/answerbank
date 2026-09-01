# What this replaces

> **Note on scope.** This file describes the capability gap `answerbank` fills. It deliberately
> names no vendor and quotes no price. Comparative analysis of commercial products is maintained
> outside this repository for now. Nothing in the tool, its help text, its command output, or any
> hosted page may name or price a commercial product — see `docs/program/CONVENTIONS.md`.

## The gap is packaging, not capability

A narrative library — a boilerplate store, an answer bank — exists as a secondary feature inside
grant management software sold to development departments, and inside proposal management software
sold to enterprise sales teams. Both categories are priced and shaped for an organization with a
team, not for a solo consultant carrying twelve clients.

**Nobody sells this as a standalone tool to solo and small-shop grant consultants.** That is the
gap, and it is a gap in packaging rather than in capability. Which is why the answer is a directory
of Markdown files and not a product.

## What the field does instead

Twelve Google Docs folders. Retyping the mission statement to fit a 250-word box when the 500-word
version is three folders away. A board list that was accurate in March. Last fiscal year's budget
figure in a proposal submitted in November.

That last category is the one that matters. **Shipping a stale fact is the failure mode**, and it is
the reason `answerbank stale` is the headline command rather than a utility.

## Why plain files win here

- **Portability.** A consultant's answer bank should outlive any tool, including this one. Markdown
  in a directory is readable in thirty years and in any editor.
- **Version history for free.** The library is git-friendly by design, so client handoff and
  "what changed since last year" are solved problems rather than features.
- **No lock-in to argue about.** There is no export feature because there is nothing to export from.
- **It composes with agents.** Pointed at the library, a model drafts an application in that
  organization's own verified, current language. That is a categorically different artifact from
  generic AI grant writing, and it is the sharp answer to the objection — now common among funders —
  that AI-written applications are obvious and unwelcome.

## The career-development angle

For an emerging development professional, a well-maintained answer bank *is* the craft. It teaches
what a complete organizational narrative looks like, and it is portable between jobs. No commercial
product has a reason to optimize for that, because a portable asset is the opposite of retention.

## What `answerbank` does not claim

- It does not write your proposal. It stores and serves what you already verified.
- It does not check your answers for accuracy. It tracks when you last confirmed them and tells you
  when that was too long ago.
- It has no model, no embeddings, and no generation. Search is deterministic keyword ranking.
