# Build prompt: answerbank core (library, CLI, MCP server, exporters)

You are building `answerbank`, a local-first narrative library for grant writers, from an empty repository to a working, tested, publishable Python package. Work through this document top to bottom. It is the specification, not a suggestion.

---

## 1. Mission

Grant applications ask the same thirty questions over and over. Mission statement, organizational history, board composition, evaluation methodology, DEI statement, sustainability plan, logic model, budget narrative. A consultant with twelve nonprofit clients keeps those answers in twelve Google Docs folders, retypes them constantly to fit different word limits, and periodically ships something wrong: last year's staff count, a board member who resigned in June, a budget figure from the prior fiscal year.

`answerbank` fixes that with plain files. One directory per organization, one Markdown file per answer, word-count variants inside the file, a verification date on every answer, and one command that reports what has gone stale.

Three things determine whether this succeeds:

1. **`answerbank stale` is the headline feature.** It is the command a consultant runs every Monday, and it is what prevents the embarrassing error. Treat it as the product, not as a utility. It gets the best output formatting and the most careful date math in the codebase.
2. **The file format is the product.** A human must be able to edit an answer in any text editor with no tooling, and a `git diff` on an answer file must be readable by somebody who does not know git. Every format decision serves those two properties.
3. **The MCP server is the real unlock.** Pointed at a consultant's library, Claude can draft a complete application for a specific funding opportunity using only that organization's verified, current language. That is a categorically different artifact from generic AI grant writing, and it is the sharp answer to the objection, now common among funders, that AI-written applications are obvious and unwelcome. Build the MCP server as a first-class surface, not as a wrapper written last.

---

## 2. Read these first, before writing any code

In this order:

1. `docs/program/CONVENTIONS.md` — binding program conventions. Repo layout, engineering standards, the dual CLI plus MCP requirement, the optional OpenGrants integration rules, attribution requirements, and documentation writing standards.
2. `README.md` in this repository — the user-facing contract. The commands, flags, and output shapes shown there are promises. If you find yourself wanting to deviate, that is a stop-and-ask, not a judgment call.
3. `docs/NON-GOALS.md` — the boundaries. Do not build anything on that list, even if it seems easy and useful while you are in the code.
4. `docs/research/data-sources.md` — section 1 has the thirty-question taxonomy you will implement as templates. Section 3 has the exact word- and character-counting specification you must satisfy.
5. `SECURITY.md` at the repository root — specifically the scope notes about credential handling and local data.
6. `.github/workflows/ci.yml` and `.gitignore` are already in place, generated from the shared program templates. Do not regenerate them.

---

## 3. Hard constraints

Violating any of these is a build failure, not a style disagreement.

### Privacy (the absolute one)

**Nothing leaves the user's machine unless the user explicitly asks.** This is a promise printed in the README and it is enforced here.

- Exactly one module in the package, `answerbank/opengrants.py`, is permitted to open a network socket. Every other module importing `socket`, `http`, `urllib`, `requests`, or `httpx` is a bug. Add a test that walks the source tree and asserts this.
- `answerbank/opengrants.py` is only reachable from the `pull` and `push` commands. It must not be imported at CLI startup, and it must not be reachable from the MCP server unless the server was started with `--allow-network` (default off).
- No telemetry. No analytics. No crash reporting. No update check. No "anonymous usage statistics." Not behind a flag, not opt-in, not at all.
- The `OPENGRANTS_API_KEY` value is never written to disk, never logged, never included in an exception message, and never echoed. If you build a debug output mode, redact it there too.
- The tool never writes outside the resolved library directory, except where the user gave an explicit output path via `--out`. Reject `--out` paths that would traverse upward through a symlink into somewhere unexpected only insofar as you resolve the path; do not build a sandbox, just do not write anywhere you were not told to.
- The MCP server is read-only by default. Write tools are registered only when `--allow-write` is passed.

### Architecture

- **Business logic never lives in a CLI command handler or an MCP tool handler.** Both are thin adapters over the library modules. If a capability is available through the CLI and not through MCP, that is a bug.
- **No database. No cache. No index file. No lockfile.** The directory of Markdown files is the complete state of the system. If `answerbank stale` can ever disagree with what is in the files, the design is wrong.
- **No model, no embeddings, no vector store, no generation.** Search is deterministic keyword ranking computed at query time.

### Dependencies

Runtime dependencies are exactly:

```
click
pyyaml
python-docx
mcp
```

Optional extras:

```
pdf  -> reportlab
clip -> pyperclip
```

Development: `pytest`, `ruff`.

Use `urllib.request` from the standard library for the two OpenGrants calls. Use `tomllib` from the standard library for config. Do not add `rich`, `httpx`, `requests`, `pydantic`, `typer`, `jinja2`, `python-dateutil`, or anything requiring a compiler or a system library. Adding a dependency is a stop-and-ask.

### Behaviour

- Python 3.11 minimum. CI runs 3.11, 3.12, 3.13. Do not use syntax newer than 3.11.
- `uvx answerbank --help` must work with no configuration, no library, and no environment variables.
- Every command supports `--json` producing machine-readable output on stdout with nothing else mixed in. Human-readable text is the default.
- Human-readable output goes to stdout; warnings, notes, and diagnostics go to stderr. This matters because people will pipe `get` into `pbcopy`.
- Colour only when stdout is a TTY and `NO_COLOR` is unset.
- Never modify a user's answer text. The single exception is `verify`, which appends a dated line to the `## notes` section.
- Front-matter round-trips byte-identically except for the field being deliberately changed. Do not reflow YAML, do not reorder keys, do not restyle quotes.

---

## 4. The on-disk format

This is the specification. Implement it exactly.

### Library layout

```
<library root>/
├── answerbank.toml
├── riverkeeper/
│   ├── _org.toml
│   ├── mission.md
│   ├── organizational-history.md
│   ├── board-composition.md
│   └── ...
├── casa-esperanza/
│   ├── _org.toml
│   └── ...
└── northside-youth/
    └── ...
```

Rules:

- An organization is any immediate subdirectory of the library root that contains `_org.toml`. Directories without it are ignored, so a user can keep `_attachments/` or `.git/` alongside without confusing the tool.
- The directory name is the organization slug. Lowercase letters, digits, and hyphens. It must equal the `org` field in every answer file inside it.
- Answer files are `*.md` at the top level of an organization directory. Subdirectories inside an organization directory are ignored, deliberately: flat is easier to browse and grep, and thirty to sixty files is not enough to need folders.
- Files beginning with `_` or `.` are never treated as answers.

### `answerbank.toml`

```toml
# Library configuration. Every key optional.

default_org = "riverkeeper"

[export]
default_format = "docx"
# Where exports land when --out is omitted. Relative paths resolve against
# the library root. Created on demand.
directory = "_exports"

[stale]
# How far ahead "due soon" looks, in days.
due_soon_days = 14
```

### `_org.toml`

```toml
name = "Lower Cosumnes Riverkeeper"
slug = "riverkeeper"
ein = "94-1156365"
fiscal_year_end = "06-30"
notes = "Fiscal sponsor relationship ended 2024. They file their own 990 now."

# Optional, set by `answerbank pull`
[opengrants]
profile_id = "..."
last_pulled = "2026-08-14"
```

Only `name` and `slug` are required. `slug` must equal the directory name.

### An answer file

```markdown
---
id: board-composition
title: Board composition and governance
org: riverkeeper
variants: [100, 300]
last_verified: 2026-08-14
verify_every: 180d
source: >
  Board roster maintained by the Governance Committee. Cross-checked
  against the FY2025 Form 990 Part VII and the minutes of the
  2026-07-09 annual meeting.
tags: [governance, core, required-everywhere]
status: verified
owner: Dana Whitfield
---

## 100 words

Lower Cosumnes Riverkeeper is governed by an eleven-member volunteer board...

## 300 words

Lower Cosumnes Riverkeeper is governed by an eleven-member volunteer board of
directors that meets six times annually...

## notes

2026-08-14 (verify): Confirmed with Dana. 11 seats, Vega seat still open.
Sonoran Joint Venture wants demographics as a table; see board-demographics.md.
```

### Front-matter schema

| Field | Required | Type | Validation |
|---|---|---|---|
| `id` | yes | string | `^[a-z0-9][a-z0-9-]*$`. Must equal the filename stem. Unique within the organization. |
| `title` | yes | string | Non-empty. |
| `org` | yes | string | Must equal the containing directory name. |
| `variants` | yes | list of int | Non-empty, positive, strictly ascending, no duplicates. Must correspond exactly to the `## N words` sections present in the body: same set, no extras on either side. |
| `last_verified` | yes | date | `YYYY-MM-DD`. Must not be in the future. |
| `verify_every` | yes | duration | `^\d+[dwmy]$`. See duration rules below. |
| `source` | yes | string or list of strings | Non-empty. |
| `tags` | no | list of strings | Lowercase, may contain `:` for namespacing (`lang:es`). |
| `status` | no | `draft` \| `verified` \| `retired` | Defaults to `draft`. |
| `owner` | no | string | Free text. |

**Unknown keys are an error.** A typo'd `last_verifed` that silently does nothing is exactly the failure this tool exists to prevent. `check` reports it and `load` raises.

### Duration rules for `verify_every`

- `Nd` adds N days.
- `Nw` adds N times seven days.
- `Nm` adds N calendar months, incrementing the month field and clamping the day to the last valid day of the target month. `2026-01-31` plus `1m` is `2026-02-28`.
- `Ny` adds N calendar years, clamping `02-29` to `02-28` in a non-leap year.

Implement this by hand in `staleness.py`. Do not add `python-dateutil`.

### Body rules

- The body begins after the closing `---` of the front matter.
- A variant section starts at a line matching `^##\s+(\d+)\s+words\s*$` and continues until the next `^##\s` line or end of file.
- A `^##\s+notes\s*$` section is a notes section. It is never exported, never returned by `get`, and never included in MCP output except by an explicit request from the `answerbank_get_answer` tool's `include_notes` argument.
- Content before the first `##` heading is preamble. It is preserved on write and ignored on read. Warn about it in `check`.
- Any other `##` heading is an error in `check` and a warning on load. This catches a stray `## 250 words` typed as `## 250 Words` or `## 250-word version`.
- Variant bodies are Markdown. Preserve them exactly on read and write, including trailing whitespace patterns, because a round-trip that reflows a user's text is a betrayal.

### Why variants live in one file

You will be tempted to split them. Do not. A 150-word and a 500-word mission statement are the same claim at two lengths and they go stale together. Two files means two places to update when the executive director changes, and one of them gets missed. One file gives one verification date governing every length, one readable git diff for the whole change, and one thing to open. The cost is longer files, which is not a cost.

If a future contributor proposes splitting them, this paragraph is the reason to say no.

---

## 5. Module architecture

```
answerbank/
├── pyproject.toml
├── README.md                  (already written; do not rewrite it, make the code match it)
├── LICENSE                    (Apache 2.0 full text — already present)
├── NOTICE                     (already written)
├── CONTRIBUTING.md            (already written)
├── CODE_OF_CONDUCT.md         (already present)
├── SECURITY.md                (already present)
├── CHANGELOG.md               (create, start at 0.1.0)
├── .env.example               (already written)
├── .gitignore                 (already present)
├── .github/workflows/ci.yml   (already present)
├── docs/                      (already written)
├── prompts/                   (already written)
├── templates/                 (you create: 30+ template answer files)
│   └── CREDITS.md
├── src/answerbank/
│   ├── __init__.py            __version__
│   ├── __main__.py            python -m answerbank
│   ├── model.py
│   ├── frontmatter.py
│   ├── library.py
│   ├── counts.py
│   ├── staleness.py
│   ├── search.py
│   ├── templates.py
│   ├── errors.py
│   ├── export/
│   │   ├── __init__.py
│   │   ├── markdown.py
│   │   ├── text.py
│   │   ├── docx.py
│   │   ├── pdf.py
│   │   └── clipboard.py
│   ├── opengrants.py
│   ├── cli.py
│   └── mcp_server.py
└── tests/
    ├── fixtures/
    │   └── library/           a real 3-org library committed to the repo
    └── test_*.py
```

### `model.py`

Dataclasses only. No I/O, no filesystem, no clock. Frozen where practical.

```python
@dataclass(frozen=True)
class Variant:
    words: int          # declared target from front matter
    body: str           # raw Markdown, exactly as written
    actual_words: int   # computed by counts.count_words

@dataclass(frozen=True)
class Answer:
    id: str
    title: str
    org: str
    variants: tuple[Variant, ...]     # ascending by declared words
    last_verified: date
    verify_every: str
    source: tuple[str, ...]
    tags: tuple[str, ...]
    status: Literal["draft", "verified", "retired"]
    owner: str | None
    notes: str | None
    path: Path
    raw_frontmatter: str              # verbatim, for byte-identical round-trip
    preamble: str

@dataclass(frozen=True)
class Org:
    slug: str
    name: str
    path: Path
    ein: str | None
    fiscal_year_end: str | None
    notes: str | None

@dataclass(frozen=True)
class LibraryConfig:
    root: Path
    default_org: str | None
    export_format: str
    export_directory: str
    due_soon_days: int

@dataclass(frozen=True)
class StaleReport:
    answer: Answer
    due_on: date
    days_overdue: int    # negative means days remaining
    bucket: Literal["overdue", "due_soon", "ok"]
```

### `frontmatter.py`

- `parse(text: str) -> tuple[dict, str, str]` returning parsed front matter, the raw front-matter block, and the body.
- `split_body(body: str) -> tuple[str, dict[int, str], str | None]` returning preamble, variant sections keyed by declared word count, and notes.
- `serialize(raw_frontmatter: str, updates: dict) -> str` performing a **surgical** update: rewrite only the lines belonging to the named keys, leave every other byte alone. Do not round-trip through `yaml.dump`. This is the only way to guarantee the format promise.
- Use `yaml.safe_load` for reading. Never `yaml.load`.

### `library.py`

The core API. Everything else adapts it.

```python
class Library:
    @classmethod
    def discover(cls, explicit: Path | None = None) -> Library: ...
    def orgs(self) -> list[Org]: ...
    def resolve_org(self, slug: str | None) -> Org: ...
    def answers(self, org: str | None = None, *, include_retired: bool = False) -> list[Answer]: ...
    def get(self, answer_id: str, org: str | None = None) -> Answer: ...
    def select_variant(self, answer: Answer, words: int | None, *, exact: bool = False) -> tuple[Variant, bool]: ...
    def write(self, answer: Answer, *, frontmatter_updates: dict, notes_append: str | None = None) -> None: ...
    def create(self, org: str, answer_id: str, *, title: str, variants: list[int], verify_every: str, template: str | None = None) -> Answer: ...
    def check(self, org: str | None = None) -> list[Problem]: ...
```

Library root resolution, in order, first hit wins:

1. `--library PATH` on the command line.
2. `ANSWERBANK_HOME` in the environment.
3. `answerbank.toml` in the current directory, then each parent up to the filesystem root.
4. `~/answerbank` if it exists.
5. Otherwise: error with a message naming `answerbank init`.

Organization resolution, in order:

1. `--org SLUG`.
2. `ANSWERBANK_DEFAULT_ORG`.
3. `default_org` in `answerbank.toml`.
4. If the library contains exactly one organization, use it.
5. Otherwise: error listing the available slugs.

### `counts.py`

Implement exactly the specification in `docs/research/data-sources.md` section 3.

```python
def to_plain_text(markdown: str) -> str: ...
def count_words(markdown: str) -> int: ...
def count_chars(markdown: str, *, include_spaces: bool = True) -> int: ...

@dataclass(frozen=True)
class Counts:
    words: int
    characters: int
    characters_no_spaces: int

def counts(markdown: str) -> Counts: ...
def check_limits(c: Counts, *, limit_words: int | None, limit_chars: int | None) -> list[str]: ...
```

`to_plain_text` strips heading markers, emphasis markers, list markers, and link syntax keeping link text and dropping URLs, then normalises all whitespace runs including newlines to a single space. A token counts as a word if it contains at least one alphanumeric character. Hyphenated compounds are one word. `9,500` is one word. Character counts are Unicode code points over the same plain text.

Do not add a Markdown parsing dependency. A careful set of regular expressions is correct for this, and the fixtures will prove it.

### `staleness.py`

```python
def parse_duration(s: str) -> tuple[int, str]: ...
def add_duration(d: date, duration: str) -> date: ...
def due_date(answer: Answer) -> date: ...
def evaluate(answer: Answer, *, today: date, due_soon_days: int) -> StaleReport: ...
def report(answers: list[Answer], *, today: date, due_soon_days: int) -> list[StaleReport]: ...
```

`today` is always injected. No function in this module calls `date.today()`. That is what makes the tests deterministic across leap years and month ends.

### `search.py`

Deterministic keyword ranking. No network, no model, no persisted index.

Score each answer against the query tokens:

- Title match: weight 5.
- Tag match: weight 3.
- `id` match: weight 4.
- Body match in any variant: weight 1, counted once per distinct token, not per occurrence, so a long variant does not dominate.
- Normalise by the square root of the token count so 1,000-word variants do not swamp 150-word ones.
- Case-insensitive, punctuation stripped, simple suffix trimming (`s`, `es`, `ing`) and nothing cleverer.

Ties break by `id` ascending, so results are stable.

### `templates.py`

Ships the thirty-plus starter templates as package data from `templates/`. Provides `list_templates()`, `get_template(id)`, and `render_for_org(id, org)` which strips the `## guidance` section, rewrites `org`, sets `last_verified` to today, sets `status: draft`, and replaces `source` with the placeholder string.

You are writing these template files. Use the taxonomy in `docs/research/data-sources.md` section 1. Each template needs a `## guidance` section (what the funder is actually asking, what a strong answer contains as a numbered list, common mistakes, typical lengths in the field) and skeleton variants with bracketed slots rather than finished prose. Guidance quality matters more than volume; write them as a person who has read grant applications, not as a person filling a table.

### `export/`

`export/__init__.py` dispatches on format and returns:

```python
@dataclass(frozen=True)
class ExportResult:
    path: Path | None       # None for clipboard
    format: str
    counts: Counts
    warnings: tuple[str, ...]
```

### `opengrants.py`

The only module allowed to open a socket. `urllib.request` only.

- Base URL `https://qnoicxojartltrownmal.supabase.co/functions/v1/`, override with `OPENGRANTS_API_BASE`.
- Header `Authorization: Bearer <key>` from `OPENGRANTS_API_KEY`.
- Descriptive `User-Agent`: `answerbank/<version> (+https://github.com/egeria-corporation/answerbank)`.
- 10-second timeout. One retry on a connection error, none on an HTTP error status.
- **Every call is wrapped so it degrades silently.** No key, network failure, timeout, expired key, rate limit, malformed JSON: return `None` and let the caller print a single plain line. Never raise into the CLI.
- Surface `X-RateLimit-Remaining` in `--json` output when present.
- Anything sourced from the API is marked `— live from OpenGrants` in human output.

### `cli.py` and `mcp_server.py`

Thin. If either exceeds roughly 400 lines you have put logic in the wrong place.

---

## 6. Command surface

Exact signatures. These appear in the README and are a contract.

Global options, available on every command: `--library PATH`, `--json`, `--no-color`, `--version`, `--help`.

```
answerbank init [PATH] --org SLUG [--name "Display Name"] [--no-templates]
    Creates the library root, answerbank.toml, the org directory, _org.toml,
    and copies every starter template in as a draft answer.
    PATH defaults to ~/answerbank. Idempotent: adding a second org to an
    existing library is the same command. Never overwrites an existing file.

answerbank orgs [--json]
    Slug, display name, answer count, overdue count, one line each.

answerbank list [--org SLUG] [--tag TAG]... [--status STATUS] [--json]
    Columns: id, title, variants, last_verified, due, status.
    --tag repeatable, AND semantics.

answerbank get ID [--org SLUG] [--words N] [--exact] [--copy] [--json]
    Prints the requested variant to stdout and nothing else, so it pipes.
    --words picks the nearest declared variant; the note about which one and
    its actual count goes to STDERR.
    --exact fails with exit 2 if no variant declares exactly N.
    --words omitted with one variant returns it; with several, errors and
    lists them.
    --copy also places it on the clipboard.
    --json gives {id, title, org, requested_words, returned_variant,
    actual_words, characters, characters_no_spaces, exact, last_verified,
    due_on, days_overdue, source, status, text}.

answerbank new ID --org SLUG [--title TEXT] [--words 150,500] [--verify-every 1y]
                 [--from-template TEMPLATE_ID] [--edit]
    Creates one answer file. Refuses to overwrite. --edit opens $EDITOR after.

answerbank edit ID [--org SLUG]
    Opens the file in $VISUAL, then $EDITOR, then a platform default.

answerbank stale [--org SLUG | --all-orgs] [--within DURATION] [--json]
    THE headline command. Groups OVERDUE, DUE SOON, and a summary count.
    --within overrides due_soon_days, accepting the same duration syntax.
    Exit 1 if anything is overdue, 0 otherwise. Exit code is the contract
    that makes this usable from cron; do not change it.

answerbank verify ID [--org SLUG] [--date YYYY-MM-DD] [--note TEXT]
    Sets last_verified (default today) and appends "YYYY-MM-DD (verify): TEXT"
    to the ## notes section, creating that section if absent.
    Touches nothing else in the file. Verify this with a byte-comparison test.

answerbank check [--org SLUG] [--strict] [--json]
    Validates every answer against the schema. Reports unknown front-matter
    keys, missing required fields, id/filename/org mismatches, variants
    declared but absent, sections present but undeclared, future
    last_verified dates, malformed durations, duplicate ids, and variants
    whose actual word count differs from the declared target by more than
    20 percent (a warning, not an error, since a 500-word variant at 512
    words is fine and one at 180 words is a mistake).
    Exit 1 on any error. --strict promotes warnings to errors.

answerbank search QUERY [--org SLUG] [--all-orgs] [--limit N] [--json]
    Ranked results with the matching line in context.

answerbank export ID... [--org SLUG] [--words N]
                        [--format docx|pdf|md|txt|clipboard]
                        [--out PATH] [--limit-words N] [--limit-chars N]
                        [--title-page] [--json]
    One or more ids in the order given. Always reports words, characters,
    and characters excluding spaces. Warns and exits 1 when a stated limit
    is exceeded, having still written the file.

answerbank templates [list | show TEMPLATE_ID] [--json]

answerbank pull --org SLUG [--profile-id ID] [--dry-run] [--json]
    Requires OPENGRANTS_API_KEY. Without it, prints one line and exits 0.

answerbank push --org SLUG [--dry-run] [--json]
    Requires OPENGRANTS_API_KEY and the Pro or Developer tier. --dry-run
    prints the exact payload, field by field, and sends nothing.

answerbank mcp [--library PATH] [--allow-write] [--allow-network]
    Runs the MCP server on stdio. No output on stdout except protocol
    traffic; log to stderr only.
```

### Exit codes

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Actionable finding: something overdue, a check failure, a limit exceeded |
| 2 | Usage error: unknown id, ambiguous org, no exact variant, bad flag |
| 3 | Library problem: not found, unreadable, malformed config |

### Output formatting for `stale`

Get this one right; it is what people see weekly.

```
OVERDUE
  casa-esperanza  staff-count            verified 2025-09-02   overdue by 63d   (verify every 90d)
  casa-esperanza  board-composition      verified 2025-11-12   overdue by 22d   (verify every 180d)
  riverkeeper     annual-budget          verified 2025-07-01   overdue by 156d  (verify every 180d)

DUE SOON (14 days)
  northside-youth program-outcomes       verified 2026-02-20   due in 6d        (verify every 180d)

42 answers across 4 organizations. 3 overdue, 1 due soon.
```

- Columns aligned by computed width, not hard-coded.
- Org column omitted entirely when scoped to one org.
- Sorted by days overdue descending, then org, then id.
- Red for overdue and yellow for due soon, only when stdout is a TTY and `NO_COLOR` is unset.
- Nothing at all printed for the `ok` bucket. The point is the short list.
- When everything is current: `All 42 answers across 4 organizations are current.` and exit 0.

---

## 7. Export implementation

### DOCX (`python-docx`)

- One answer per section. `title` as a Heading 1 when more than one id is exported or `--title-page` is passed; a single-answer export defaults to body text only, because people paste that straight into a portal.
- Convert Markdown minimally and honestly: paragraphs, bold, italic, ordered and unordered lists, and nothing else. A grant narrative is prose. Do not build a Markdown-to-DOCX engine.
- No headers, no footers, no page numbers, no watermark, no "generated by" line. Funders see these files.
- Default style: 11pt, single spaced, one-inch margins. Leave the theme font alone.
- `--title-page` adds a first page with the organization name, the answer titles, and the verification dates. Useful for a client review packet, never for a submission.

### PDF (`reportlab`, optional extra)

- Only imported inside the PDF exporter function, so a user without the extra pays nothing.
- Missing extra produces: `PDF export needs the pdf extra. Install with: uv tool install "answerbank[pdf]"` and exit 2. Do not attempt an install.
- Letter page size, one-inch margins, 11pt serif, ragged right. Word wrapping only; no hyphenation.
- Same content rules as DOCX. No page furniture unless `--title-page`.

### Markdown and text

- `md` writes the variant body verbatim. Front matter and notes excluded.
- `txt` writes `to_plain_text` output with paragraph breaks preserved as blank lines. This is the format that pastes cleanly into a portal textarea and it should be the one you recommend in the help text.

### Clipboard

- Try `pyperclip` if installed. Otherwise shell out to `pbcopy` (macOS), `wl-copy` then `xclip -selection clipboard` (Linux), `clip.exe` (Windows and WSL).
- If none work, write the text to stdout and print one line to stderr explaining that the clipboard was unavailable. **Never fail a command because the clipboard is missing.** People run this over SSH.

### Counts and limits

Every export prints the three counts. `--limit-words` or `--limit-chars` exceeded produces a warning naming the overage and exits 1, with the file still written. Under the limit and within 5 percent of it produces an informational note, because portals count differently than you do.

---

## 8. MCP server

Transport: stdio. Server name `answerbank`. Log to stderr only; anything on stdout that is not protocol traffic breaks the client.

**Read-only by default.** Register write tools only when `--allow-write` was passed. Register nothing network-capable unless `--allow-network` was passed, and even then only `answerbank_pull_profile`.

Every tool returns JSON as text content. Include `last_verified`, `due_on`, and `is_stale` on every answer returned by every tool, without exception. A model that cannot see staleness cannot warn about it, and warning about it is the point.

### `answerbank_list_orgs`

```json
{"type": "object", "properties": {}, "additionalProperties": false}
```

Returns `[{slug, name, answer_count, overdue_count, ein, fiscal_year_end}]`.

### `answerbank_list_answers`

```json
{
  "type": "object",
  "properties": {
    "org": {"type": "string", "description": "Organization slug. Omit to use the library default."},
    "tag": {"type": "string", "description": "Filter to answers carrying this tag."},
    "include_retired": {"type": "boolean", "default": false}
  },
  "additionalProperties": false
}
```

Returns `[{id, title, tags, variants, last_verified, verify_every, due_on, days_overdue, is_stale, status, owner}]`. No answer text; this is the index a model reads first to decide what to fetch.

### `answerbank_get_answer`

```json
{
  "type": "object",
  "properties": {
    "id": {"type": "string"},
    "org": {"type": "string"},
    "words": {"type": "integer", "description": "Target length. Returns the nearest declared variant."},
    "exact": {"type": "boolean", "default": false},
    "include_notes": {"type": "boolean", "default": false, "description": "Include the private notes section. Off by default; notes often contain internal remarks not meant for a funder."}
  },
  "required": ["id"],
  "additionalProperties": false
}
```

Returns `{id, title, org, text, requested_words, returned_variant, actual_words, characters, characters_no_spaces, exact, last_verified, due_on, days_overdue, is_stale, source, status, owner, notes}`.

### `answerbank_search`

```json
{
  "type": "object",
  "properties": {
    "query": {"type": "string"},
    "org": {"type": "string"},
    "all_orgs": {"type": "boolean", "default": false},
    "limit": {"type": "integer", "default": 10, "maximum": 50}
  },
  "required": ["query"],
  "additionalProperties": false
}
```

`all_orgs` defaults false and, when a specific `org` is given, is ignored. Cross-client bleed is the worst failure this tool could have; make it require an explicit request.

### `answerbank_stale`

```json
{
  "type": "object",
  "properties": {
    "org": {"type": "string"},
    "all_orgs": {"type": "boolean", "default": false},
    "within_days": {"type": "integer", "default": 14}
  },
  "additionalProperties": false
}
```

Returns `{overdue: [...], due_soon: [...], summary: {total, overdue, due_soon, orgs}}`.

### `answerbank_draft_context`

The important one. Given a funder's question, assemble the grounding material.

```json
{
  "type": "object",
  "properties": {
    "org": {"type": "string"},
    "question": {"type": "string", "description": "The funder's question, in the funder's own words."},
    "words": {"type": "integer", "description": "The funder's word limit, if stated. Selects variants near that length."},
    "max_answers": {"type": "integer", "default": 5, "maximum": 12}
  },
  "required": ["org", "question"],
  "additionalProperties": false
}
```

Returns:

```json
{
  "org": {"slug": "riverkeeper", "name": "Lower Cosumnes Riverkeeper", "ein": "94-1156365"},
  "question": "...",
  "word_limit": 1500,
  "sources": [
    {
      "id": "community-engagement",
      "title": "Community engagement and constituent voice",
      "text": "...",
      "variant_words": 500,
      "actual_words": 512,
      "last_verified": "2026-05-02",
      "due_on": "2027-05-02",
      "is_stale": false,
      "source": "Program logs 2024-2026; community advisory board minutes 2026-03-11",
      "relevance": 0.82
    }
  ],
  "stale_sources": [
    {"id": "staff-count", "last_verified": "2025-11-01", "days_overdue": 210,
     "warning": "Past its verification date. Do not state this figure as current. Ask the user to confirm it."}
  ],
  "gaps": [
    "No answer exists for 'evaluation-methodology'. The question appears to require one."
  ],
  "usage_instructions": "Draft using only the text in `sources`. Do not introduce facts, figures, dates, names, or claims that do not appear there. Where the question requires something not present in `sources`, say so explicitly in your response rather than inventing it. Anything listed in `stale_sources` is past its verification date and must not be presented as current; if you use it, flag it to the user. Preserve the organization's own phrasing wherever it fits the question; this text was approved by the organization and generic substitutions make the application worse."
}
```

The `usage_instructions` string is not decoration. It is the mechanism that makes this different from generic AI grant writing, and it ships in the payload so it reaches the model regardless of what the user typed.

`gaps` is computed by comparing the question against the template taxonomy: if the question matches a standard question id that the organization has no answer for, or has only a `draft`, say so. Finding a gap two weeks before a deadline is worth more than the drafting.

### `answerbank_check_limits`

```json
{
  "type": "object",
  "properties": {
    "text": {"type": "string"},
    "limit_words": {"type": "integer"},
    "limit_chars": {"type": "integer"}
  },
  "required": ["text"],
  "additionalProperties": false
}
```

Returns `{words, characters, characters_no_spaces, within_limits, warnings}`. Lets a model check its own draft before handing it back.

### Write tools (only with `--allow-write`)

- `answerbank_verify` — `{id, org, date?, note?}`. Same semantics as the CLI command.
- `answerbank_save_answer` — `{id, org, words, text, source, verify_every, title?}`. Creates or replaces one variant. **Never** silently replaces a `verified` answer: if `status` is `verified`, require `confirm_overwrite: true` and return an error naming the current text's verification date otherwise.

### Network tool (only with `--allow-network`)

- `answerbank_pull_profile` — `{org}`. Fetches the OpenGrants profile. Returns it, does not write it.

---

## 9. Milestones

Ship each one green before starting the next. Commit at each boundary.

**M0. Skeleton.** `pyproject.toml` with the console entry point, package layout, `.gitignore`, CI workflow, `LICENSE`, `CHANGELOG.md`. `uvx --from . answerbank --help` works. Ruff clean.

**M1. Format and model.** `model.py`, `frontmatter.py`, `counts.py`, `staleness.py`. Fixture library committed under `tests/fixtures/library/` with three organizations and at least twelve real answer files, including deliberately broken ones for `check` to catch. Tests for parsing, byte-identical round-trip, counting against hand-computed values, and duration math across month ends and leap years.

**M2. Library and read commands.** `library.py`, plus `orgs`, `list`, `get`, `check`, `search`. `--json` on all of them.

**M3. Staleness.** `stale` with the exact output format above, exit codes, `--within`, `--all-orgs`. Plus `verify`, with a byte-comparison test proving nothing but `last_verified` and the notes section changed.

**M4. Templates and creation.** Write the thirty-plus templates. `templates.py`, `init`, `new`, `edit`, `templates list|show`. A fresh `init` must produce a library that passes `check` with zero errors.

**M5. Export.** All five formats, counts, limit warnings. Tests open the generated DOCX with `python-docx` and assert on its paragraph text rather than only asserting the file exists.

**M6. MCP server.** Every tool above. Tests drive the server in-process through the MCP SDK's test harness, asserting on tool schemas and returned payloads. Verify manually with Claude Desktop and Claude Code using the README config snippets exactly as written.

**M7. OpenGrants.** `pull` and `push`. Tests use a local stub HTTP server, never the real API, and must include the no-key, timeout, 401, 429, and malformed-JSON paths, each asserting the command exits 0 and prints one line.

**M8. Release.** README examples verified against real output. `CHANGELOG.md` at 0.1.0. Publish to PyPI so `uvx answerbank` works for a stranger.

---

## 10. Acceptance criteria

Each of these is checkable. Do not report done until every one passes.

**Quickstart**

- [ ] `uvx answerbank init /tmp/ab --org riverkeeper` creates a library with thirty or more draft answers, on a machine with no configuration and no network, in under five seconds.
- [ ] `uvx answerbank --help` runs with no library present and exits 0.
- [ ] Elapsed time from reading the README's first line to seeing real output is under sixty seconds for a Python-literate stranger.

**Format**

- [ ] Reading and rewriting an answer with `verify` changes exactly the `last_verified` line and the notes section. `diff` shows nothing else. Asserted with a byte comparison in a test.
- [ ] An answer with an unknown front-matter key fails `check` with a message naming the key and suggesting the nearest valid one.
- [ ] A `variants: [150, 500]` file whose body has a `## 250 words` section fails `check`.
- [ ] A library is a working Obsidian vault: open `tests/fixtures/library/` in Obsidian and confirm front matter renders as properties and variants appear in the outline.

**Counting**

- [ ] `count_words` matches hand-computed values on all fixtures, including hyphenated compounds, numbers with commas, em dashes, bulleted lists, and inline links.
- [ ] The three counts appear on every export and every `get --json`.
- [ ] `--limit-words 500` on a 517-word variant writes the file, warns naming the 17-word overage, and exits 1.

**Staleness**

- [ ] `stale` output matches the format in section 6 exactly, including alignment and the summary line.
- [ ] Exit 1 when anything is overdue, 0 otherwise, verified in a test.
- [ ] `2026-01-31` plus `1m` is `2026-02-28`. `2024-02-29` plus `1y` is `2025-02-28`. Both asserted.
- [ ] No function in `staleness.py` calls `date.today()`.

**Multi-client**

- [ ] Every read path is scoped to one organization unless `--all-orgs` is explicit.
- [ ] A test asserts that `search` and `answerbank_search` never return an answer from a different organization without `all_orgs: true`.
- [ ] `orgs` lists all three fixture organizations with correct overdue counts.

**Privacy**

- [ ] A test walks the source tree and asserts that no module except `opengrants.py` imports `socket`, `http`, `urllib`, `requests`, or `httpx`.
- [ ] Running every command with the network disabled produces normal behaviour, except `pull` and `push`, which each print one line and exit 0.
- [ ] `grep -ri "telemetry\|analytics\|posthog\|sentry\|mixpanel" src/` returns nothing.
- [ ] `OPENGRANTS_API_KEY` never appears in any output, including `--json` and error paths. Asserted with a test that sets a sentinel key and greps all output.
- [ ] The MCP server started without `--allow-write` does not register any write tool. Asserted against the tool list.

**MCP**

- [ ] The Claude Desktop config snippet in the README works verbatim.
- [ ] The `claude mcp add` command in the README works verbatim.
- [ ] `answerbank_draft_context` returns `usage_instructions`, `stale_sources`, and `gaps` on a query where the fixture library has a stale answer and a missing one.
- [ ] Every tool payload carries `last_verified` and `is_stale` on every answer.
- [ ] Nothing is written to stdout except MCP protocol traffic.

**Quality**

- [ ] `uv run ruff check .` and `uv run ruff format --check .` clean.
- [ ] `uv run pytest -q` green on 3.11, 3.12, 3.13.
- [ ] Test coverage above 85 percent on `library.py`, `counts.py`, `staleness.py`, and `frontmatter.py`.
- [ ] No business logic in `cli.py` or `mcp_server.py`. Each under roughly 400 lines.
- [ ] CI badge green in the README.

---

## 11. Verification steps

Run these by hand before declaring done. Automated tests are necessary and not sufficient.

1. **The cold-start test.** On a clean machine or container with no configuration: `uvx answerbank init ~/ab --org riverkeeper`, then `list`, then `get mission --words 250`, then `stale`. Time it. Over sixty seconds means something is wrong.

2. **The Monday test.** Hand-edit three fixture answers to have past-due dates. Run `stale --all-orgs`. Read the output the way a consultant with eleven minutes would. Is it obvious what to do next? Is the list short enough to act on?

3. **The round-trip test.** Open an answer in a text editor with unusual but valid YAML: block scalars, single quotes, an inline comment, a blank line inside front matter. Run `verify`. Diff. Anything beyond the intended change is a failure.

4. **The Obsidian test.** Open the fixture library as an Obsidian vault. Front matter should render as properties, variants as outline entries.

5. **The portal test.** Export a 500-word variant as `txt`, paste it into a plain textarea, and count characters in the browser. The tool's character count should be within a handful of the browser's. If it is far off, the plain-text renderer is wrong.

6. **The Claude Desktop test.** Configure the server with the README snippet. Then, in a real conversation: "Using only Lower Cosumnes Riverkeeper's answer bank, draft a 1,500-word community engagement section for the EPA Environmental Justice Government-to-Government program. Flag anything stale and tell me what is missing." The response must cite verification dates, refuse to invent facts, and name the gaps. If it invents a program name that is not in the library, `usage_instructions` is not strong enough.

7. **The airplane test.** Disable networking entirely. Every command except `pull` and `push` must behave identically. Those two print one line and exit 0.

8. **The handoff test.** `cp -r ~/ab/riverkeeper /tmp/handoff`. Would a program director who has never seen this tool understand the files? If not, the format is too clever.

---

## 12. Stop and ask the human

Do not decide these alone. Stop, state the trade-off in two or three sentences, and wait.

1. **Adding any runtime dependency** beyond the four listed, or any dependency requiring a compiler or a system library.
2. **Any code path that sends data anywhere**, for any reason, including a documentation link fetch, a version check, or a "did you know" message. This is the constraint most likely to erode through good intentions.
3. **Changing the front-matter schema**: adding a field, removing one, changing a validation rule, or changing how `variants` relate to body sections. The format is the product and it is published in the README.
4. **Splitting variants into separate files**, or any other change to the one-file-per-answer model.
5. **Adding a cache, index, or database of any kind**, including a "just for performance" one.
6. **Renaming or removing a command or flag** documented in the README.
7. **Changing `stale`'s exit codes.** People will put this in cron.
8. **Adding a write capability to the MCP server that is not gated behind `--allow-write`**, or loosening the confirmation on overwriting a `verified` answer.
9. **Anything on the `docs/NON-GOALS.md` list**, even when a user has asked for it and it looks like an afternoon of work.
10. **A PDF engine that needs system libraries** (WeasyPrint, wkhtmltopdf, pandoc, a headless browser). ReportLab is the choice precisely because it is pure Python.
11. **Template content that makes a factual claim about a funder's requirements** that you cannot cite to a published form or notice. Guidance about what reviewers look for is fine and valuable. "HRSA requires 1,500 words" is a citation-needed claim.
12. **Anything that reads or writes outside the resolved library root**, other than an explicit `--out` path.
13. **Any output that mentions the OpenGrants API key** when one is not set. The README mentions it exactly once. Command output mentions it never.

---

## 13. Writing standards for anything user-facing

Help text, error messages, template guidance, and log lines are read by grant consultants, not developers.

- Expand every acronym on first use. NOFO is a notice of funding opportunity. LOI is a letter of inquiry. MCP is the Model Context Protocol.
- Error messages say what happened, what the tool expected, and what to type next. `Error: invalid front matter` is useless. `mission.md: unknown front-matter key 'last_verifed'. Did you mean 'last_verified'? Valid keys are: id, title, org, variants, last_verified, verify_every, source, tags, status, owner.` is a fix.
- Examples use realistic nonprofits and real foundations. Lower Cosumnes Riverkeeper, Casa Esperanza Family Services, Northside Youth Collective. The Packard Foundation, the Bush Foundation, HRSA, EPA. Never `foo`, `bar`, `Example Org`, or `Lorem ipsum`.
- No marketing voice anywhere in the tool. No exclamation marks, no emoji, no "Great job!", no celebration on success. A consultant running this at 11pm before a deadline wants information, not encouragement.
- Never nag. No upgrade prompts, no signup suggestions, no "consider setting OPENGRANTS_API_KEY."
