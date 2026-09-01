# Data sources

`answerbank` has no upstream dataset. It manages text its users write. That makes this document shorter than its equivalent in the sibling repositories, and it changes what belongs in it.

What answerbank actually depends on is a set of *conventions*: which questions get asked, how funder application forms are structured, and how word and character limits are counted. Those conventions are the substrate the template gallery and the export counters are built on, so they are documented here with the same care a dataset would get.

**Verification status.** Everything in this file was compiled on 2026-08-30. Items marked **VERIFY** are drawn from practitioner convention or from sources that change without notice, and must be re-checked against the current form or notice of funding opportunity before they appear in public-facing copy on `answers.opengrants.io`. Federal forms in particular are revised on an expiration cycle, and a form's OMB control number and expiration date are printed on the form itself.

---

## 1. The standard question taxonomy

The claim behind this tool is that grant applications reuse a small, stable set of questions. This is the working list. It is the set that ships as `templates/` and that becomes the indexable pages on the hosted gallery.

The list is not derived from a single published source. It is the intersection of what appears across federal notices of funding opportunity, private foundation applications, community foundation letters of inquiry, and the common grant application formats that regional associations of grantmakers have maintained in various forms since the 1990s. Where a question maps onto a named section of a published form, the mapping is noted in section 2.

### Core thirty

| # | id | Title | Typical `verify_every` | Notes |
|---|---|---|---|---|
| 1 | `mission` | Mission statement | `2y` | Also appears verbatim on Form 990 Part III Line 1. Keep them consistent; reviewers check. |
| 2 | `vision` | Vision statement | `2y` | Often optional. Some funders treat mission and vision as one question. |
| 3 | `organizational-history` | Organizational history and founding story | `1y` | The most commonly padded answer in the field. Dates and founding circumstances only. |
| 4 | `need-statement` | Statement of need / problem statement | `1y` | The one answer that must carry current external data. Cite year and source inline. |
| 5 | `target-population` | Population served | `1y` | Demographics change. Numbers here should trace to your own service data. |
| 6 | `service-area` | Geographic service area | `1y` | Name counties and census tracts, not "the region." |
| 7 | `program-description` | Core program description | `1y` | One file per major program. Use ids like `program-riverwatch`. |
| 8 | `goals-objectives` | Goals and measurable objectives | `1y` | Objectives must be numeric and time-bound or reviewers discount them. |
| 9 | `logic-model` | Logic model / theory of change | `1y` | Keep the prose version here; the diagram is an attachment. |
| 10 | `evaluation-methodology` | Evaluation methodology and data collection | `1y` | Name the instruments and who administers them. |
| 11 | `outcomes-impact` | Outcomes and impact to date | `180d` | Tied to your reporting cycle. Goes stale fast and reviewers notice. |
| 12 | `organizational-capacity` | Organizational capacity and qualifications | `1y` | Why this organization rather than another one. |
| 13 | `key-staff` | Key staff and qualifications | `180d` | Bios plus credentials. Full CVs are attachments, not narrative. |
| 14 | `staffing-structure` | Staffing structure and organizational chart | `90d` | The fastest-moving answer most organizations have. |
| 15 | `board-composition` | Board composition and governance | `180d` | The single most commonly stale answer in grant work. |
| 16 | `volunteer-program` | Volunteer program | `1y` | Hours and counts date quickly if you cite them. |
| 17 | `dei-statement` | Diversity, equity, inclusion, and accessibility | `1y` | Expected language in this area has moved repeatedly. Re-read before reuse. |
| 18 | `community-engagement` | Community engagement and constituent voice | `1y` | Increasingly scored explicitly in federal environmental and health programs. |
| 19 | `partnerships` | Partnerships and collaborations | `180d` | Partner lists rot. Confirm each partner still says yes before submission. |
| 20 | `sustainability-plan` | Sustainability plan | `1y` | About the program after the grant, not the organization in general. |
| 21 | `budget-narrative` | Budget narrative | `180d` | Per-project, but the boilerplate framing is reusable. |
| 22 | `other-funding` | Other funding sources and diversification | `90d` | Funders check this against Form 990 Schedule B and against each other. |
| 23 | `indirect-cost-rate` | Indirect cost rate | `1y` | State whether you hold a negotiated rate, use the de minimis rate, or neither. |
| 24 | `financial-management` | Financial management and internal controls | `1y` | Segregation of duties, approval thresholds, who reconciles. |
| 25 | `audit-compliance` | Audit and compliance history | `180d` | Includes single audit status. See the threshold note in section 4. |
| 26 | `fiscal-sponsorship` | Fiscal sponsorship arrangement | `1y` | Only if applicable. Name the sponsor and the model (typically Model A or Model C). |
| 27 | `risk-management` | Risk management and contingency | `1y` | Insurance, safeguarding, what happens if a key person leaves. |
| 28 | `dissemination` | Dissemination and knowledge sharing | `1y` | Federal research and demonstration programs almost always ask. |
| 29 | `timeline-workplan` | Timeline and work plan | `1y` | Structure is reusable even when dates are not. |
| 30 | `cultural-competency` | Cultural competency and language access | `1y` | Health and human services funders ask directly. |

### Frequently needed beyond the thirty

| id | Title | Notes |
|---|---|---|
| `executive-summary` | Executive summary / project abstract | Almost always the last thing written and the first thing read. |
| `nonprofit-status` | Legal status and determination letter | EIN, subsection code, ruling date, fiscal sponsor if applicable. |
| `eeo-policy` | Equal employment opportunity and non-discrimination | Frequently a required assurance rather than a narrative. |
| `data-security` | Data security and client confidentiality | Increasingly asked where services involve health or immigration status. |
| `environmental-considerations` | Environmental and climate considerations | Now common well outside environmental programs. |
| `lobbying-disclosure` | Lobbying and advocacy posture | Relates to SF-LLL. Get this one reviewed by counsel, not by a template. |
| `annual-budget` | Current annual operating budget | Numeric. `verify_every: 180d` at the longest. |
| `staff-count` | Current staff count, FTE and headcount | `verify_every: 90d`. This is the number that embarrasses people. |

**VERIFY** before publication: the frequency claims ("almost always," "increasingly") are practitioner judgment, not measured. Either soften them on the public site or back them with a counted sample of notices.

---

## 2. How funder applications are structured

### Federal: the SF-424 family

Nearly every discretionary federal grant application is assembled from standard forms in the SF-424 family plus program-specific attachments. The forms are U.S. Government works and are not copyrightable in the United States (17 U.S.C. sec. 105). Current versions and instructions live on Grants.gov under Forms, and each form carries an OMB control number and an expiration date printed on its face.

The pieces that matter for a narrative library:

- **SF-424, Application for Federal Assistance.** Cover data: applicant identity, UEI, DUNS successor identifiers, congressional districts, project title, dates, estimated funding. Almost all of this is reusable organizational fact, and most of it belongs in `_org.toml` rather than in narrative files.
- **SF-424A, Budget Information (Non-Construction).** The budget grid. Its companion narrative is normally a separate **Budget Narrative Attachment Form**.
- **SF-424B, Assurances (Non-Construction).** Assurances signed, not written. Nothing here is narrative, but the underlying policies (non-discrimination, drug-free workplace, environmental compliance) are things a narrative library should be able to describe when a program officer asks.
- **Project Abstract Summary form.** A one-page structured abstract. Length constraints appear in the form instructions and are commonly expressed in characters rather than words. **VERIFY** the specific limit against the current form before quoting a number publicly.
- **Project Narrative Attachment Form.** The actual proposal. The internal structure is dictated by the notice of funding opportunity, not by the form.
- **SF-LLL, Disclosure of Lobbying Activities.** Required when applicable.

Agency narrative structures worth knowing, because they determine which of the thirty questions you need and in what order. Each is **VERIFY** against the specific current notice, since agencies revise these:

- **HHS operating divisions (HRSA and similar).** Commonly: Introduction, Needs Assessment, Methodology, Work Plan, Resolution of Challenges, Evaluation and Technical Support Capacity, Organizational Information, Budget Justification. Maps onto questions 4, 7, 8, 29, 27, 10, 12, 21.
- **Department of Education discretionary programs.** Structured by scored selection criteria published in the notice, commonly: Need for Project, Quality of Project Design, Quality of Project Services, Quality of Project Personnel, Adequacy of Resources, Quality of the Management Plan, Quality of the Project Evaluation. Point values per criterion are stated in the notice and should drive how much of each answer you use.
- **NIH.** Specific Aims, then Research Strategy in three parts: Significance, Innovation, Approach. Page limits are set per activity code and are strict. This is the least reusable narrative in the field and the least well served by this tool.
- **NSF.** Project Summary with three required sub-parts (Overview, Intellectual Merit, Broader Impacts), then Project Description. The Project Summary sub-parts are enforced by the submission system.
- **EPA environmental justice programs.** Narrative weighted heavily toward community engagement, partnerships, and demonstrated community benefit, which is why questions 18 and 19 deserve full-length variants rather than stubs.

### Foundations: the letter of inquiry

Most private foundations that accept unsolicited requests begin with a letter of inquiry or letter of intent, typically one to three pages. The near-universal shape:

1. Opening: who you are, one or two sentences, and the specific amount requested.
2. Statement of need, tightly scoped to what this project addresses.
3. Project description: what you will do, for whom, over what period.
4. Expected outcomes, stated so they could be measured.
5. Organizational capacity: why you.
6. Budget summary: total project cost, amount requested, other committed support.
7. Close with a contact and an offer to send more.

Practical consequence for the format: an LOI mostly wants the shortest variant of each answer. A library with only 500-word and 1,000-word variants forces retyping at exactly the moment volume is highest. Keep a 100 to 150 word variant of every core answer. This is the strongest single argument for multiple variants living in the same file.

**Common grant application forms.** Regional associations of grantmakers historically published shared application formats so an applicant could write once for many local funders. Several are still in use and several have been retired in favor of portal-specific forms. **VERIFY** any specific association's current form and whether it is still maintained before citing it on the public site. The structural convention has outlived the forms and is what the taxonomy above encodes.

**Portals.** Applications are increasingly submitted through grants management platforms rather than documents: Foundant Grant Lifecycle Manager, Submittable, SM Apply, Fluxx, Blackbaud Grantmaking, and agency-specific federal systems. This matters to answerbank for one reason above all others: portals impose character and word limits on textarea fields and typically enforce them by truncating or refusing, which is why export must report counts. See section 4.

---

## 3. Word and character counting: the rules this tool uses

This is the most operationally important section in the document, because different tools disagree and the disagreement costs people submissions.

### The problem

- **Microsoft Word** counts a hyphenated compound (`community-based`) as one word. It counts a number as one word. It counts text in text boxes and footnotes depending on settings.
- **Google Docs** counts similarly to Word but differs on some punctuation-adjacent cases.
- **Web portal textareas** usually count characters, not words, and usually include spaces. Some include newline characters, some strip them. Some count the invisible markup a rich text editor inserted when you pasted from Word.
- **Pasting from Word into a portal** can introduce curly quotes, en and em dashes, and non-breaking spaces. These are single characters in a character count but can be rejected, mangled, or re-encoded by a portal.

### What answerbank does

Exports and `get` report three numbers, always:

- `words`
- `characters` (including spaces)
- `characters_no_spaces`

The canonical word counter is defined as follows, and this definition is the specification the implementation must satisfy:

1. Render the Markdown body of the requested variant to plain text: strip heading markers, emphasis markers, list markers, and link syntax, keeping link text and dropping URLs.
2. Normalize all runs of whitespace, including newlines, to a single space.
3. Split on spaces.
4. Count tokens containing at least one alphanumeric character. A bare `-` or `&mdash;` standing alone is not a word.
5. A hyphenated compound is one word. An em-dash-joined pair with no spaces is one word. `9,500` is one word. `9,500-square-foot` is one word.

Character counts are taken over the same plain-text rendering, after whitespace normalization, and are counted in Unicode code points rather than bytes, because that is what portals count.

`--count-style word` is provided as an alias for the default, reserved so that a future `--count-style raw-markdown` can be added without breaking anyone. **Do not add a second counting mode without a real reported case that needs it.**

### Practical guidance the gallery should carry

- Target 90 to 95 percent of a stated limit. Portals that count differently than you do will otherwise eat your last sentence.
- When a limit is stated in characters, assume spaces are included unless the form says otherwise. That assumption fails safe.
- When a limit is stated in pages, ask the notice for the font, size, margin, and line spacing before estimating. A page of 12-point Times New Roman, single spaced, one-inch margins, is roughly 500 to 550 words. **VERIFY** this rule of thumb against the specific notice; agencies state their own typography requirements and some specify a minimum font size that changes the arithmetic.
- Write in plain text, not in Word, when the destination is a portal textarea. Fewer characters get mangled that way, which is a side benefit of keeping a library in Markdown.

---

## 4. Facts from the program research dossier that answers touch

These are verified in the shared `RESEARCH.md` as of 2026-08-30 and are relevant because several standard answers depend on them.

- **Single audit threshold.** The 2024 Uniform Guidance revision raised the threshold from $750,000 to $1,000,000 in annual federal expenditures, effective for fiscal years beginning on or after 2024-10-01. Any `audit-compliance` answer written before that change may state the old number. Template guidance should say so explicitly.
- **Form 990 Part III Line 1** carries the organization's mission statement as filed. A `mission` answer that differs materially from the filed 990 is a reviewer's easy question. The template guidance should tell people to check.
- **Form 990 Part VII** lists officers, directors, trustees, and key employees. It is the cross-check for `board-composition` and `key-staff`.
- **SAM.gov registration status and expiration** gate every federal application and are the most common avoidable disqualification. This belongs in `_org.toml` as a date, not in narrative, and `grantcheck` is the tool that checks it.

## 5. Optional OpenGrants enrichment

Used only by `answerbank pull` and `answerbank push`, only with a key the user set.

- Base URL: `https://qnoicxojartltrownmal.supabase.co/functions/v1/`
- Auth: `Authorization: Bearer <key>`
- `POST /match-grants-api` accepts an organization profile and returns matched opportunities. It requires the Pro or Developer tier. `answerbank pull` uses the profile it holds to pre-seed a library; `answerbank push` sends a completed narrative set back as a richer profile.
- Rate limit headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`.
- Docs: https://ops.opengrants.io/api-docs

Both calls must degrade silently. A network failure, an expired key, or a rate limit must leave every other command working exactly as it did.

---

## 6. What this document is not

There is no refresh cadence here because there is no dataset to refresh. The conventions above move on the timescale of years, with two exceptions worth watching:

1. **Federal form revisions.** Forms expire. Check the OMB expiration date on any form referenced in template guidance at least annually.
2. **Expected DEI and community engagement language.** This has changed repeatedly and quickly. Template guidance in these areas should carry a visible "last reviewed" date on the public gallery, and the gallery's own `verify_every` discipline should apply to itself.
