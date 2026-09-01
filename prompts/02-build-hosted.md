# Build prompt: answers.opengrants.io (hosted template gallery)

You are building the hosted companion site for `answerbank`: a fully static website at `answers.opengrants.io` containing the thirty standard grant application questions with real guidance on what a strong answer contains, plus the tool's documentation.

This is not a marketing site. It is a reference work that happens to link to a tool.

---

## 1. Mission

Somebody types "what should a nonprofit sustainability plan say" into a search box or asks a model. Today they get thin content-marketing posts from grant writing vendors. This site should be the best answer to that question and to twenty-nine others like it, at a stable URL, free, with no signup wall.

Three audiences, in priority order:

1. **An emerging development professional** learning what a complete organizational narrative looks like. Working through the thirty questions in order is a serviceable curriculum, and the answer bank they build is portable between jobs. This is the career-development artifact, and it is why the guidance content has to be genuinely good rather than merely present.
2. **A search engine and an answer engine.** These pages are how the program earns category authority. The repositories will never rank; these pages can.
3. **A working consultant** who wants the skeleton for a question they are staring at right now, and does not want to install anything to get it.

The tool is a link at the bottom of a useful page. It is never a gate.

---

## 2. Read these first

1. `docs/program/HOSTING.md` — binding. Platform reasoning, the SEO and GEO requirements, DNS notes. Section 2 of this prompt restates the platform decision but the reasoning lives there.
2. `docs/hosted/architecture.md` in this repository — the full site design, URL structure, page anatomy, and content operations.
3. `docs/research/data-sources.md` — the thirty-question taxonomy is section 1. Application structures are section 2. The word and character limit conventions are section 3 and become a guide page.
4. `README.md` — the tool's contract. Documentation pages must match it exactly.
5. `docs/program/CONVENTIONS.md` — TypeScript engineering standards and documentation writing standards.
6. `docs/NON-GOALS.md` — the site inherits every one of these. No accounts, no uploads, no user content, no forms.

---

## 3. Hard constraints

- **Fully static.** `output: 'static'` in Astro. No server-side rendering, no adapter, no serverless functions, no edge middleware. The build produces a directory of files.
- **Platform-portable.** Nothing Netlify-specific. No Netlify Forms, Functions, Identity, or Edge Functions. Redirects in `_redirects` and headers in `_headers`, both of which Netlify and Cloudflare Pages read. Moving platforms must be a half-hour job.
- **No client-side JavaScript for primary content.** Every question page must be complete and readable with JavaScript disabled. Search may ship a small prebuilt index; nothing else ships JS.
- **No accounts, no forms, no uploads, no user-generated content, no newsletter modal, no chat widget, no cookie banner.** A consent banner on a site whose adjacent pitch is "we do not take your data" is a self-inflicted wound.
- **Template content is not duplicated.** The site reads `templates/*.md` from this repository as its content collection. There is exactly one copy of every guidance paragraph and it is the same file the Python package ships.
- **The build fails on invalid template front matter.** Same schema the Python tool enforces. Two independent validators over one specification.
- **No web font CDN.** Self-host one variable font subset, or use a system font stack.

---

## 4. Platform decision

**`answers.opengrants.io` deploys to Netlify, under the existing OpenGrants team. This is deliberate.**

The program hosting plan puts four of five companion sites on Cloudflare for two specific reasons: R2 has no egress fees, which matters when serving multi-gigabyte derived datasets, and Cloudflare Pages' file-count ceiling forces edge rendering for sites with a page per organization. Neither applies here. This site has no dataset and roughly a hundred pages.

With the technical tiebreakers gone, the operational one decides: OpenGrants already runs a Netlify team, and one deploy dashboard that a non-engineer can check is worth more than platform uniformity.

Record this in the site's own docs so a future engineer finding one site on a different platform knows it was a choice rather than drift. Revisit it if the site starts serving a dataset, if bandwidth becomes a visible line item, or if OpenGrants leaves Netlify.

**Both deploy paths must work.** Cloudflare Pages is documented and tested, and CI must not depend on either platform. See section 10.

---

## 5. Stack

- **Astro**, latest stable, static output.
- **Content collections** with a Zod schema mirroring the answerbank front-matter specification, reading from `../templates/`.
- **`pnpm`**, TypeScript strict mode, **`biome`** for lint and format, **`vitest`** for tests. Per program conventions for TypeScript.
- Site lives in `site/` in this repository. Build command `pnpm build`, output `site/dist`.
- Styling: plain CSS with custom properties. No Tailwind, no CSS framework, no component library. This is thirty text pages; a design system is overhead.

Astro is chosen because it emits zero JavaScript by default, which is what the answer-engine requirements demand, and because content collections give type-checked front matter for free.

---

## 6. Content model

### Source of truth

```
answerbank/
├── templates/                 <- the only copy of the guidance content
│   ├── mission.md
│   ├── need-statement.md
│   ├── sustainability-plan.md
│   ├── ...
│   └── CREDITS.md
└── site/
    ├── src/content/config.ts  <- collection pointing at ../../templates
    ├── src/content/guides/    <- site-only long-form explainers
    ├── src/content/docs/      <- tool documentation
    └── ...
```

A template file has front matter identical to any answer file, plus a `## guidance` section that the Python tool strips on copy and this site publishes.

### Collection schema

Validate, in the Zod schema, every rule the Python `check` command enforces:

- `id` matches `^[a-z0-9][a-z0-9-]*$` and equals the filename stem.
- `title` non-empty.
- `org` equals `_template`.
- `variants` non-empty, ascending, unique, and exactly matching the `## N words` sections in the body.
- `last_verified` is a valid date, not in the future. On a template this means "guidance last reviewed."
- `verify_every` matches `^\d+[dwmy]$`.
- `source` non-empty.
- No unknown keys. Set the schema to strict.

A build that succeeds on a malformed template is a bug. Add a vitest case that feeds a deliberately broken fixture through the schema and asserts it throws.

### Question ordering and grouping

Use the taxonomy order from `docs/research/data-sources.md` section 1, grouped for the index page:

- **Who you are** — mission, vision, organizational history, nonprofit status
- **What the problem is** — need statement, target population, service area
- **What you will do** — program description, goals and objectives, logic model, timeline and work plan
- **How you will know it worked** — evaluation methodology, outcomes and impact, dissemination
- **Who does the work** — organizational capacity, key staff, staffing structure, board composition, volunteer program
- **How you show up in community** — DEI, community engagement, partnerships, cultural competency
- **How the money works** — budget narrative, other funding, indirect cost rate, financial management, audit and compliance, fiscal sponsorship, sustainability plan
- **What could go wrong** — risk management, data security

Groups are for human navigation. The canonical URL never contains a group, so regrouping never breaks a link.

---

## 7. Page anatomy

### A question page, in this order

The order is the design. The first screen decides whether the page is useful and whether an answer engine can quote it.

1. **H1: the question phrased as a funder would ask it.** "What is your organization's plan to sustain this program after the grant period ends?" not "Sustainability."
2. **A two-sentence direct answer** to what the funder is actually asking. This paragraph is what gets extracted and quoted. Write it as a standalone claim that survives being lifted out of the page.
3. **What a strong answer contains** — numbered, each item self-contained.
4. **Common mistakes** — the highest-value content on the page, because nobody else publishes it.
5. **Typical length**, with concrete word ranges for federal narratives, foundation applications, and letters of inquiry.
6. **The skeleton answer** at two lengths, in a copy control, plus a link to the raw `.md` file that is byte-identical to the repository template.
7. **How often to re-verify, and why** — the recommended `verify_every` and the reason it is that number. This is where the tool's central idea gets taught rather than sold.
8. **Related questions**, cross-linked.
9. **Source and vintage**: which published form or notice structure this reflects, and the date the guidance was last reviewed, rendered from the template's `last_verified`.
10. **One quiet line about the tool.** A sentence and a link. No banner, no modal, no interstitial.

### Other pages

- `/` — what this is, the thirty questions as a scannable list, one paragraph on the tool, one paragraph on why the answers have dates on them. Nothing else. No hero video, no testimonials, no logo wall.
- `/questions/` — grouped index with a one-line description per question, ordered per section 6.
- `/collections/federal-application/`, `/collections/foundation-loi/`, `/collections/first-thirty-days/` — curated subsets with an introductory paragraph explaining who each is for. `first-thirty-days` is the career-development on-ramp: the eight answers a new development hire should build first, in order.
- `/guides/word-and-character-limits/` — from `docs/research/data-sources.md` section 3. This will draw search traffic on its own.
- `/guides/how-often-to-re-verify/` — the staleness argument, with the interval table from the README.
- `/guides/grounding-ai-in-verified-language/` — the MCP argument, aimed at a consultant who has been told AI grant writing is unwelcome and wants to know what is defensible. This is the highest-value page on the site for the program's positioning. Write it carefully and without hype.
- `/docs/` and children — quickstart, file format, CLI reference, MCP setup. Must match the README exactly. Where they would diverge, the README wins and the docs get fixed.

---

## 8. SEO and GEO requirements

These come from `docs/program/HOSTING.md` and apply in full. Implement all of them; each is checkable.

### Structural

- [ ] **Real content in the initial HTML response.** No client-side fetching for primary content. Satisfied by construction with a static build, and it must stay that way.
- [ ] **One canonical URL per question**, `/questions/<id>/`, with `<link rel="canonical">` on every page.
- [ ] **Slug variants 301 to canonical** via `_redirects`. At minimum: `/questions/mission-statement/` to `/questions/mission/`, `/questions/dei/` to `/questions/dei-statement/`, and every plausible alternate phrasing. Ship this from day one; you cannot add redirects retroactively for links you never had.
- [ ] **Trailing slashes consistent everywhere**, enforced in the Astro config and asserted in a build test.
- [ ] **Sitemap index chunked at 50,000 URLs per file.** One chunk for a long time. Build the index anyway so the convention matches the sibling sites.
- [ ] **`robots.txt`** allowing everything, pointing at the sitemap index.
- [ ] Per-page `<title>` and `<meta name="description">`, written per page, never templated from the H1.
- [ ] OpenGraph and Twitter card images generated at build time from the question title. One template, thirty renders.

### Structured data

This site has no organization entities, so the vocabulary differs from the sibling sites.

- [ ] Question pages: `TechArticle` or `HowTo`, plus a `FAQPage` whose single `Question` is the funder question and whose `acceptedAnswer` is the two-sentence direct answer. The nesting is what makes the page machine-quotable.
- [ ] `/questions/`: `ItemList` in canonical order.
- [ ] Docs and guides: `TechArticle`.
- [ ] Site-wide: `SoftwareApplication` for answerbank with `applicationCategory`, `operatingSystem`, `offers` at price `0`, `license` pointing at Apache 2.0, and `downloadUrl` pointing at the repository.
- [ ] Footer graph: `Organization` for Egeria Corporation with `funder` naming OpenGrants.
- [ ] Validate every emitted block against schema.org in a test. Hand-written JSON-LD drifts.

### `llms.txt`

At the root. Describe what the gallery is, that content is Apache 2.0, how to cite it, and where the canonical source files are. **Link the per-question `.md` files directly** so a crawler can fetch clean source instead of scraping HTML. That single decision does more for extraction quality than any amount of markup.

### Source and vintage, inline

Every page states its source and vintage in the body text, not only in a footer: "Guidance last reviewed 2026-08-30. Section ordering reflects the HRSA program narrative structure." Pages that show their work get cited. Pages that assert bare claims do not.

### Portfolio cross-linking

Each question page links to the sibling sites where relevant:

- `nonprofit-status`, `audit-compliance` to `check.opengrants.io`
- `other-funding`, `partnerships` to `funders.opengrants.io`
- `organizational-capacity`, `outcomes-impact` to `awards.opengrants.io`
- Every page footer to the program index.

Five sites that reference each other read as one authoritative body of work. Five that do not read as five orphans.

### Answer-engine specifics

Optimizing for a model asked a question is not the same as optimizing for a search results page.

- Lead with the direct answer. Models extract the first substantive claim under a heading.
- Use the funder's phrasing in the H1, because that is closer to how the question actually gets typed.
- One claim per sentence. Self-contained list items. Fragments that depend on the previous bullet do not survive extraction.
- Date any assertion inline, in the same sentence, never only in a page footer.
- Never put a load-bearing fact only in a table cell. Tables extract badly. State it in prose as well.

---

## 9. Performance and accessibility

- [ ] Lighthouse 95 or better on performance, accessibility, best practices, and SEO for `/` and a representative question page. Enforce in CI with Lighthouse CI; fail the build below threshold.
- [ ] Question page weight under 150 KB excluding images.
- [ ] No web font CDN. One self-hosted variable font subset, or a system stack.
- [ ] WCAG 2.1 AA: correct heading hierarchy with exactly one H1, 4.5:1 contrast, visible focus rings, a skip link, alt text on every image, and a `prefers-reduced-motion` respecting stylesheet.
- [ ] Fully usable with JavaScript disabled. Test this explicitly; it is also the fastest proxy for whether crawlers see the content.
- [ ] Copy buttons degrade to a visible code block plus a raw `.md` link when JavaScript is off.

---

## 10. Deploy

### Netlify (the chosen path)

```toml
# netlify.toml at the repository root
[build]
  base = "site"
  command = "pnpm install --frozen-lockfile && pnpm build"
  publish = "site/dist"

[build.environment]
  NODE_VERSION = "22"
  PNPM_VERSION = "9"
```

- Site created under the existing OpenGrants Netlify team.
- Production branch `main`. Deploy previews on every pull request.
- Custom domain `answers.opengrants.io` with Netlify's managed certificate.
- `_redirects` and `_headers` live in `site/public/` so they land at the root of the published directory.

`_headers` should set, at minimum:

```
/*
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Content-Security-Policy: default-src 'self'; img-src 'self' data:; style-src 'self' 'unsafe-inline'; script-src 'self'; frame-ancestors 'none'
```

A static site with no third-party anything can afford a strict policy. Take it.

### Cloudflare Pages (the documented alternative)

- New Pages project on the same repository.
- Root directory `site`, build command `pnpm build`, output directory `dist`.
- `PAGES_WRANGLER_MAJOR_VERSION=4` if the file count ever approaches the free-plan ceiling of 20,000. It will not, but document it.
- `_redirects` and `_headers` are read the same way, which is why the build must not use Netlify-specific syntax.
- Custom domain added on the Pages project, needing the Cloudflare validation record.

Document both paths in `site/README.md` with the exact settings, so switching is a lookup rather than an investigation.

### DNS

`opengrants.io` DNS is managed externally rather than at the registrar's default. Per `docs/program/HOSTING.md` this is the step most likely to sit blocked for a day, so **identify the zone holder before the launch date, not on it.**

- Netlify: `CNAME answers -> <site-name>.netlify.app`, plus any verification record Netlify requests.
- Cloudflare Pages: `CNAME answers -> <project>.pages.dev`, plus the Cloudflare validation record.

Scope is exactly `answers.opengrants.io`. Do not touch the apex or any other subdomain.

### CI

Add `.github/workflows/site.yml`, separate from the Python workflow:

- `pnpm install --frozen-lockfile`
- `pnpm biome ci .`
- `pnpm vitest run`
- `pnpm build`
- Post-build assertions (section 11)
- Lighthouse CI against the built output

CI must not require Netlify or Cloudflare credentials. A pull request from a fork has to pass.

---

## 11. Post-build assertions

Run these against `site/dist` in CI and fail the build on any of them. Every one has caught a real regression on a site like this.

- [ ] Every question in the taxonomy has a page at its canonical URL.
- [ ] No page has zero or more than one H1.
- [ ] Every page has a canonical link, a title under 60 characters, and a description between 70 and 160 characters.
- [ ] Every question page contains a `FAQPage` JSON-LD block with a non-empty `acceptedAnswer`.
- [ ] Every JSON-LD block parses as JSON and carries `@context` and `@type`.
- [ ] No internal link 404s. Crawl `dist/` and resolve every `href`.
- [ ] Every redirect in `_redirects` has a live target.
- [ ] `llms.txt` exists, is non-empty, and every URL in it resolves.
- [ ] The sitemap index exists, every child sitemap resolves, and the union of their URLs equals the set of built pages.
- [ ] No page references `localhost`, a `.netlify.app` host, or a `.pages.dev` host.
- [ ] No page contains `Lorem ipsum`, `TODO`, `TBD`, or `[placeholder]`.
- [ ] Every question page's skeleton section matches the corresponding `templates/*.md` body byte for byte, so the site and the tool cannot drift.

---

## 12. Milestones

**H0. Scaffold.** Astro project in `site/`, content collection pointed at `../templates`, strict Zod schema, biome and vitest configured, `pnpm build` green.

**H1. One excellent page.** Build `/questions/sustainability-plan/` completely, all ten sections from section 7, with real guidance content. Get the layout, typography, and structured data right on one page before generating thirty. Review it with somebody who writes grants for a living.

**H2. All thirty.** Every question page, the grouped index, the three collections. **Do not launch thin.** Thirty stubs is worse for the site's credibility than ten excellent pages plus an honest note that more are coming.

**H3. Guides and docs.** The three guide pages, and the documentation section matched line by line against the README.

**H4. SEO and GEO.** Everything in section 8. Post-build assertions from section 11 wired into CI.

**H5. Performance and accessibility.** Lighthouse thresholds met and enforced. JavaScript-disabled pass. Manual keyboard and screen reader pass on one question page.

**H6. Deploy.** Netlify site created, deploy previews working, Cloudflare path verified once by actually building it there, `site/README.md` documenting both.

**H7. DNS and launch.** Zone holder confirmed, CNAME added, certificate issued, sitemap submitted. Cross-links from sibling sites added as those come online.

---

## 13. Acceptance criteria

- [ ] Thirty question pages live, each with genuine guidance and a real "common mistakes" section. No stubs.
- [ ] A grant professional who has never seen the tool reads a question page and says the guidance is correct and useful. This is a human review gate; do not skip it.
- [ ] Every page useful with JavaScript disabled.
- [ ] Lighthouse 95-plus on all four categories for `/` and a question page, enforced in CI.
- [ ] All post-build assertions in section 11 pass.
- [ ] The site builds and deploys on both Netlify and Cloudflare Pages from the same commit with no code changes.
- [ ] `llms.txt` links the raw `.md` source for every question.
- [ ] Guidance content exists in exactly one place in the repository. A grep for a distinctive guidance sentence returns exactly one file.
- [ ] Documentation pages match the README. Any command, flag, or output shape that differs is a bug in the docs.
- [ ] No cookie banner, no modal, no newsletter capture, no chat widget, no third-party script beyond cookieless analytics.

---

## 14. Stop and ask the human

1. **Adding any interactive feature** requiring JavaScript for primary content, a backend, or user state.
2. **Adding a form of any kind**, including a newsletter signup, feedback widget, or contact form.
3. **Any analytics that sets a cookie** or would require a consent banner.
4. **Deviating from Netlify**, or adding anything platform-specific that would make the Cloudflare path fail.
5. **Publishing a specific claim about a funder's requirements** that cannot be cited to a published form or notice. "HRSA program narratives are commonly structured as..." is fine. "HRSA requires 1,500 words" needs a citation or it does not ship.
6. **Publishing a competitor's price.** `docs/research/competitive.md` marks several figures VERIFY. Re-check on the vendor's own pricing page and date-stamp it in the text, or leave it out.
7. **Changing the URL structure** after launch, for any reason.
8. **Any content presenting a template as a finished answer** rather than a skeleton. Templates that read as complete get pasted in unchanged, and identical proposals land on the same program officer's desk.
9. **Adding a gate of any kind** in front of the guidance content, including an email-for-download.
10. **Restructuring the taxonomy.** The thirty questions are shared with the Python package's templates and with the tool's gap detection. Changing them is a coordinated change across both prompts.

---

## 15. Writing standards

Same as the tool, with one addition: this content is read by people who are new to the field and by people who have done it for twenty years, often on the same page.

- Write for the newcomer in the structure and for the veteran in the specifics. A numbered list of what a strong answer contains serves the first. A sentence about what reviewers actually deduct points for serves the second.
- Expand every acronym on first use, on every page, because people arrive from search on a single page and never see the rest.
- Use realistic nonprofit examples throughout. Lower Cosumnes Riverkeeper, Casa Esperanza Family Services, Northside Youth Collective. Real funders and agencies where relevant: the Packard Foundation, the Bush Foundation, HRSA, EPA. Never `foo`, `Example Org`, or `Lorem ipsum`.
- No marketing voice. No "unlock," no "supercharge," no "game-changing." The content is the pitch.
- Never imply that using the tool improves anyone's odds of being funded. It saves time and prevents a specific class of error. Say that, and stop there.
