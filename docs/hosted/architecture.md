# Hosted companion: answers.opengrants.io

## What the site is

A fully static website with two jobs:

1. **The template gallery.** The thirty standard grant application questions, each as its own page, each with guidance on what a strong answer contains, what reviewers actually look for, and what loses points. Readable and useful without installing anything.
2. **The documentation for `answerbank`.** Quickstart, the file format specification, the command reference, and MCP setup.

Job one is the reason the site exists. Job two is a subdirectory.

The gallery is simultaneously three things, which is why it deserves real effort:

- **The career development artifact.** An emerging development professional working through thirty questions in order learns what a complete organizational narrative is. That is not available anywhere else for free.
- **The search and answer-engine surface for the entire program.** "What should a nonprofit sustainability plan say" is a question thousands of people type every month. Nobody currently answers it well at a stable, citable URL.
- **The on-ramp for people who will not open a terminal.** Read the guidance, copy the skeleton, paste it into a Google Doc. The tool is an upsell, not a prerequisite.

## What the site is not

No dataset. No database. No API. No accounts. No forms. No user-submitted content. Nothing a visitor does on this site touches an answerbank library, and no library ever gets uploaded to it.

That last point is worth stating on the site itself, because a visitor arriving from a privacy-conscious README will wonder.

---

## Platform: Netlify, deliberately

The program's hosting plan puts four of the five companion sites on Cloudflare, for two specific reasons: R2 has no egress fees, which matters enormously when you are serving multi-gigabyte derived datasets, and Cloudflare Pages' file-count ceiling forces edge rendering for sites with a page per organization.

**Neither reason applies here.** This site has no dataset to serve and roughly a hundred pages to build. Bandwidth for a static docs site is negligible on any platform, and a hundred files sits far below every ceiling either platform has.

So the tiebreaker is operational, and it points at Netlify: **the OpenGrants team already exists on Netlify.** One deploy dashboard, one set of credentials, one place a non-engineer at OpenGrants can look to see whether the site built. Brand consistency and operational familiarity are real values when the alternative gains nothing.

**Decision: `answers.opengrants.io` deploys to Netlify under the existing OpenGrants team.**

This is a deliberate split, not drift. The reasoning is recorded here so that a future engineer finding one site on a different platform understands it was a choice. If any of the following becomes true, revisit it:

- The site starts serving a dataset, or any file larger than a few megabytes.
- Traffic reaches a level where Netlify's metered bandwidth is a visible line item.
- Managing two platforms costs more attention than one dashboard saves.
- OpenGrants leaves Netlify.

**Cloudflare Pages remains a fully supported path.** The build produces a plain directory of static files with no platform-specific adapter, no serverless functions, no edge middleware, and no platform SDK. Moving to Cloudflare Pages is a new project pointed at the same repository with the same build command and the same output directory, plus a DNS change. Both paths are documented in the build prompt, and CI must not depend on either.

The portability constraint is load-bearing: **do not use Netlify Forms, Netlify Functions, Netlify Identity, Netlify Edge Functions, or Netlify Redirects syntax that has no Cloudflare equivalent.** Redirects go in a `_redirects` file, which both platforms read. Headers go in `_headers`, likewise.

---

## Stack

- **Astro**, static output only (`output: 'static'`). No adapter, no server islands, no server-side rendering.
- **Content collections** reading Markdown from the repository's `templates/` directory.
- **No client-side JavaScript for primary content.** Search may ship a small prebuilt index; everything else is HTML and CSS.
- **`pnpm`**, TypeScript strict mode, `biome` for lint and format, per program conventions for TypeScript repositories.
- Build output: `dist/`. Build command: `pnpm build`.

Astro is chosen because it produces zero-JavaScript HTML by default, which is exactly what the SEO and answer-engine requirements demand, and because content collections give type-checked front matter that can validate against the same schema the Python tool enforces.

---

## Single source of truth for template content

This is the most important architectural decision on the site.

The gallery pages and the templates that `answerbank init` copies into a new library come from **the same files**: `templates/*.md` in this repository.

```
answerbank/
├── templates/
│   ├── mission.md
│   ├── need-statement.md
│   ├── sustainability-plan.md
│   └── ...                        <- one file per question
├── src/answerbank/                <- Python package; ships templates/ as package data
└── site/                          <- Astro site; reads ../templates/ as a content collection
```

Consequences, all of them good:

- A guidance improvement contributed by a grant professional appears on the website and in the tool from one pull request.
- The site cannot drift from the tool, because there is nothing to drift from.
- The site build validates the template front matter, so a malformed template fails the website build as well as the Python test suite. Two independent validators over one schema.
- A contributor never has to know which of the two consumers they are editing.

The site build must fail loudly if `templates/` contains a file whose front matter does not validate, or whose `variants` list does not match the `## N words` sections in its body.

---

## URL structure

One canonical URL per question, keyed on the template `id`, which is the same `id` the tool uses:

```
/                                          landing
/questions/                                the thirty, grouped and ordered
/questions/mission/                        one question page
/questions/sustainability-plan/
/questions/board-composition/
...
/collections/federal-application/          curated subsets
/collections/foundation-loi/
/collections/first-thirty-days/
/guides/word-and-character-limits/         evergreen explainers
/guides/how-often-to-re-verify/
/guides/grounding-ai-in-verified-language/
/docs/                                     tool documentation
/docs/quickstart/
/docs/file-format/
/docs/cli/
/docs/mcp/
/llms.txt
/sitemap-index.xml
```

Rules:

- Trailing slash, consistently, everywhere. Pick it once and enforce it in the build.
- Slug variants redirect to canonical with a 301. `/questions/mission-statement/` redirects to `/questions/mission/`. Never let two URLs serve the same content.
- Every page carries a `<link rel="canonical">`.
- Question ids are stable forever. If a question is renamed, the old URL redirects; it does not disappear.

---

## What a question page contains

Order matters, because the first screen determines whether the page is useful and whether an answer engine can quote it.

1. **H1: the question as a funder would ask it.** "What is your organization's plan to sustain this program after the grant period ends?" rather than "Sustainability."
2. **A two-sentence direct answer** to what the funder is actually asking. This is the paragraph an answer engine will quote. Write it as a standalone claim that survives being extracted from the page.
3. **What a strong answer contains** — the numbered list from the template guidance.
4. **Common mistakes**, which is the highest-value content on the page and the part nobody else publishes.
5. **Typical length**, with the concrete word ranges seen in federal narratives, foundation applications, and letters of inquiry.
6. **A skeleton answer with bracketed slots**, at two lengths, in a copy button and as a downloadable `.md` file that is byte-identical to the template in the repository.
7. **How often to re-verify this answer, and why**, with the recommended `verify_every` value and the reason it is that number.
8. **Related questions**, cross-linked.
9. **A source and vintage line**: which published form or notice structure this reflects, and when the guidance was last reviewed.
10. **One quiet line about the tool**, with a link. Not a banner, not a modal, not a newsletter interstitial.

Every page must be complete without JavaScript and useful to somebody who will never install anything.

---

## SEO and GEO requirements

These follow the program hosting plan and apply here in full. This site is the one that has to rank, because the repositories will not.

- **Server-rendered HTML with real content in the initial response.** No client-side fetching of primary content. This is a static build, so it is satisfied by construction, and it must stay that way: no hydration for the guidance text, no lazy-loaded skeletons.
- **`schema.org` structured data on every page.** This site has no organization entities, so the entity vocabulary differs from the sibling sites:
  - Question pages: `TechArticle` or `HowTo`, plus a `FAQPage` block whose single `Question` is the funder question and whose `acceptedAnswer` is the two-sentence direct answer. Nesting `Question` inside `FAQPage` is what makes the page machine-quotable.
  - `/questions/` index: `ItemList` in canonical order.
  - Docs pages: `TechArticle`.
  - Site-wide: `SoftwareApplication` for answerbank with `applicationCategory`, `operatingSystem`, `offers` at price `0`, and `license` pointing at Apache 2.0.
  - `Organization` for Egeria Corporation in the footer graph, with `funder` naming OpenGrants.
- **One canonical URL per question.** Variants redirect. See URL structure above.
- **Sitemap index chunked at 50,000 URLs per file.** This site will have one chunk for a long time. Build the index anyway so the convention matches the sibling sites and so growth needs no rework.
- **`llms.txt` at the root** describing what the gallery is, that the content is Apache 2.0, how to cite it, and where the canonical source files live. Link the per-question `.md` files directly so a crawler can fetch clean source rather than scraping HTML.
- **Every page states its source and vintage inline.** "Guidance last reviewed 2026-08-30. Section ordering reflects the HRSA program narrative structure." Pages that show their work get cited. Pages that assert get ignored.
- **Cross-link the portfolio.** Every question page links to the sibling sites where relevant: `check.opengrants.io` from the questions about legal status and audit history, `funders.opengrants.io` from the questions about other funding sources, `awards.opengrants.io` from the questions about federal award history. Five sites that reference each other read as one authoritative body of work rather than five orphans.
- **Real page titles and meta descriptions**, written per page, never templated from the H1.
- **OpenGraph and Twitter card images**, generated at build time from the question title. One template, thirty renders, no manual design work.

### Answer-engine specifics

The reason this site exists is that people increasingly ask a model rather than a search engine. Optimizing for that is not the same as optimizing for search:

- Lead with the direct answer. Models extract the first substantive claim under the heading.
- Use the funder's phrasing in the H1, because that is closer to how the question gets typed.
- Keep each claim in its own sentence and each list item self-contained. Fragments that depend on the previous bullet do not survive extraction.
- State the date of any assertion inline, in the same sentence, not in a page footer.
- Never put a load-bearing fact only in a table cell. Tables extract badly. State it in prose too.

---

## Performance and accessibility

- Lighthouse 95 or better on performance, accessibility, best practices, and SEO for the landing page and a representative question page. Enforce in CI.
- No web fonts, or one variable font subset and self-hosted. No font CDN.
- Total page weight under 150 KB for a question page, images excluded.
- WCAG 2.1 AA: real heading hierarchy, 4.5:1 contrast, visible focus, skip link, alt text on every image.
- Works with JavaScript disabled. Verify this explicitly; it is also the fastest proxy for whether crawlers see the content.

---

## Deployment

### Netlify (the chosen path)

```toml
# netlify.toml
[build]
  command = "pnpm install --frozen-lockfile && pnpm build"
  publish = "site/dist"
  base = "site"

[build.environment]
  NODE_VERSION = "22"
  PNPM_VERSION = "9"
```

- Site lives under the existing OpenGrants Netlify team.
- Production branch `main`. Deploy previews on pull requests.
- Custom domain `answers.opengrants.io`, HTTPS via Netlify's managed certificate.
- No Netlify-specific runtime features. Redirects in `_redirects`, headers in `_headers`, both of which Cloudflare Pages also reads.

### Cloudflare Pages (the documented alternative)

- Build command `pnpm build`, output directory `site/dist`, root directory `site`.
- `PAGES_WRANGLER_MAJOR_VERSION=4` if the file count ever approaches the free-plan ceiling of 20,000. It will not.
- Custom domain added on the Pages project, which requires the validation record described below.

Switching platforms should be a half-hour job. If it ever is not, something platform-specific crept in and needs removing.

### DNS

`opengrants.io` DNS is managed externally rather than at the registrar's default, and per the program hosting plan this is the step most likely to sit blocked for a day. Confirm who holds the zone before the launch date, not on it.

- Netlify: `CNAME answers -> <site-name>.netlify.app`, plus whatever verification record Netlify requests for the custom domain.
- Cloudflare Pages: `CNAME answers -> <project>.pages.dev`, plus the Cloudflare validation record.

Do not put the apex or any other subdomain in scope. This site owns exactly `answers.opengrants.io`.

---

## Content operations

The guidance content is the asset, and it ages. Two disciplines keep it honest, and both mirror what the tool itself does:

1. **Every question page carries a `last_reviewed` date** from the template front matter, rendered visibly on the page. A page whose guidance has not been reviewed in more than a year renders a quiet note saying so. The gallery should be subject to its own staleness rule; anything else would be embarrassing.
2. **The DEI, community engagement, and lobbying pages get reviewed more often than the rest**, because expected language in those areas has moved repeatedly and quickly. Set their review interval to six months and put it in the repository's issue templates.

New guidance arrives through pull requests to `templates/`, following `CONTRIBUTING.md`. Contributors are credited in `templates/CREDITS.md` and on the page.

---

## Analytics

Whatever OpenGrants already runs across its properties, provided it is cookieless and requires no consent banner. A consent banner on a page whose entire pitch is "we do not take your data" is a self-inflicted wound.

Track two things and ignore the rest: which question pages get traffic, and how many people click through to the repository. The first tells you which guidance to expand. The second tells you whether the gallery is doing its job as an on-ramp.

No analytics of any kind ship in the Python tool. That is a separate promise and it is absolute.

---

## Launch sequence

1. Template front matter validates in both the Python test suite and the site build.
2. All thirty question pages render with real guidance content. Thirty stubs is worse than ten good pages; do not launch thin.
3. Redirects, canonicals, sitemap index, and `llms.txt` verified against the built output.
4. Lighthouse thresholds met and enforced in CI.
5. Deploy preview reviewed by somebody who writes grants for a living, not only by an engineer.
6. DNS cutover, with the zone holder identified and available.
7. Cross-links from the sibling sites added once those exist.
8. Submit the sitemap. Watch which questions draw traffic and write more depth where they do.
