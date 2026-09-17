# dop-t.com — the product site, direction "Panel"

**Status:** approved by the owner on 2026-09-16 (direction A, "Painel");
**amended 2026-09-17** — the visual system in §5 is superseded by §9, "the
site wears the product's chrome". §1–§4 (audience, information architecture,
content rules) stand.
**Replaces:** the proof-of-pipeline page shipped on 2026-09-09

## 1. What this is

The public face of the DOP platform: what the product is, why it is worth
anyone's attention, how it is built, and how far along it is. Two languages,
first-class. No backend, no account, no JavaScript that the content depends on.

The site keeps every rule it already carries, because each one exists for a
reason the README records:

- `content/en/` and `content/pt-br/` mirror each other; `make check` refuses a
  page that exists in one language only.
- A block driven by `[params]` renders only when its value is non-empty. Blank
  means invisible, never an empty box.
- Hugo is pinned (0.148.2). Internal links are checked on every deploy.
- Open Graph tags on every page.

What changes: the site gets a design to hold, so the CSS leaves `baseof.html`
and becomes a file; the home stops being one paragraph and becomes a page with
sections; two pages join it.

## 2. Audience and the one action

**Who lands here:** an individual developer who ships with an agent, curious
whether DOP is worth watching; and a technical reader (a friend, a future
contributor, an investor doing homework) who wants to see how it is built.

**The one action, repeated down the page:** *See the code on GitHub*. The
repositories are public since 2026-09-16; "built in the open" is a value
proposition the visitor can verify in one click, not a claim.

A *Sign in* control appears in the header only when `params.app.signInURL` is
non-empty. Plans and legal pages stay switched off until they exist. The site
never advertises what cannot be bought or opened.

## 3. Information architecture

| Route | Page | What it answers |
|---|---|---|
| `/` | Home | What is DOP, why should I care, how is it built (in one picture), how far along is it |
| `/architecture/` | Architecture | The components and their roles; the five invariants; the life of an event; the decision record |
| `/status/` | Status | The subprojects and their state; what shipped, with dates; what is next; the QA environment |

Portuguese lives under `/pt-br/` with the same three routes
(`/pt-br/`, `/pt-br/architecture/`, `/pt-br/status/`). Slugs stay in English
on purpose: one URL shape, one link check, one `translationKey` per page.

**The menu**, identical on every page: `Product · Architecture · Status ·
GitHub · PT/EN`. *Product* points at the home. *GitHub* is an external link to
`https://github.com/barrosef/dop`. The language switch takes the visitor to the
same page in the other language, never to the other home.

**The footer:** the wordmark, the tagline, the three routes, the GitHub link,
and the legal links when `params.legal.*` are filled.

### 3.1 Home, section by section

1. **Hero.** Eyebrow in mono: *Delivery orchestration · built in the open*.
   Headline: *The developer and the agent build together. From the workspace to
   the delivered PR.* One paragraph. Two buttons: primary *See the code on
   GitHub*, secondary *Read the architecture*. On the right, the **demand
   timeline panel**: a static reproduction of the cockpit's event log for one
   demand — five events (`account-created`, `demand-opened`, `spec-approved`,
   `verification-green`, `pr-delivered`), each with a time and an actor, and a
   footer strip with the retry policy (5 attempts, 1s → 1min backoff, dead
   letters: 0, outbox: transactional). Every value in the panel is true of the
   platform as built; none is invented.
2. **Why it exists** — four value propositions, one card each:
   - *Two operators, one cockpit.* The agent runs the demand; the developer
     approves the spec, gives context and decides. The division of labour is in
     the flow, not in a prompt.
   - *Everything is an event.* Every write emits an event that carries its
     context — actor, request, session. A failure goes to a dead-letter queue
     with everything needed to replay it, not into a log nobody reads.
   - *A sandbox per demand.* Isolation between accounts and between demands,
     with the project's knowledge mounted inside as a git repository.
   - *Built in the open.* Every structuring decision is an ADR in a public
     repository; the code that implements it is next to it.
3. **How it is built** — the architecture in one diagram (see §5.4) and three
   sentences: a core in Go that is the only source of truth; a BFF in Python
   with no database and no secret; a cockpit that consumes a committed
   contract. Link: *The whole architecture →*.
4. **Where it stands** — three numbers that are true today and a line each:
   subprojects designed (5 of 7), the latest ADR (0030), the QA environment on
   GCP (live). Link: *Status in detail →*. The numbers are `[params.status]`
   values so they can be updated without touching a layout.
5. **Closing call** — the headline's promise restated in one line and the
   primary button once more.

### 3.2 Architecture page

1. **The components.** One row per repository: `dop-core` (Go — domain, state,
   transactions, event log; owns the schema and the vault), `dop-api` (Python
   BFF — REST+SSE for the cockpit, gRPC for the CLI; no database, no secret),
   `dop-app` (React/Vite — the cockpit), `dop-infra` (Terraform for GCP, k3d
   locally), the sandbox (a microVM per demand). Each row links to its
   repository.
2. **The five invariants**, verbatim from `AGENTS.md`, numbered because they
   are cited by number.
3. **The life of an event.** A horizontal flow: write in Postgres → outbox →
   NATS JetStream → consumer → (on exhaustion) dead-letter queue → error
   ledger. With the numbers: MaxDeliver 5, backoff 1s/5s/15s/1min.
4. **The decision record.** The ADR index — number, title, one line — each
   linking to the file on GitHub. Generated from a data file
   (`data/adrs.toml`), not typed into a layout.

### 3.3 Status page

1. **The subprojects.** The SP-0..SP-6 table from `docs/ROADMAP.md`: what
   each decides and its state, as a data file (`data/subprojects.toml`).
2. **Shipped**, most recent first, with dates: events with context and the
   dead-letter queue (2026-09-13); the QA environment on GCP, codified in
   Terraform (2026-09-08), with `api.qa.dop-t.com` and `auth.qa.dop-t.com`
   verified since (2026-09-09); the project's
   knowledge as a git repository, ADR-0028 (2026-09-03); the core verifying its
   callers, ADR-0029 (2026-09-03); the second factor end to end, ADR-0027
   (2026-09-02).
3. **Next.** The agreed order from the roadmap: the agent's tools and the
   cockpit; hosted Claude Code as the laboratory; then the user stories.
4. **An honest line** at the top: *DOP is pre-release. Nothing here can be
   signed into yet.* It disappears when `params.app.signInURL` is filled.

## 4. Content rules

- Copy is written per language in `content/<lang>/*.md`, never in layouts.
  Interface strings (menu, buttons, labels) live in `i18n/*.toml`. Data that is
  the same in both languages but has translatable labels (the ADR index, the
  subprojects) lives in `data/*.toml` with `title_en` / `title_pt` columns.
- No number on the site that the repository cannot back. The retry policy, the
  ADR count, the dates — each has a source in the umbrella repository.
- No lorem, no bracketed placeholders in a shipped page. A section without
  real content is a section that does not render.
- Portuguese is written as Brazilian Portuguese, addressing the reader as
  *você*. English is plain and short.

## 5. The visual system — "Panel"

The product is a cockpit. The site looks like one: dark by default, one signal
colour, numbers in a monospace face, everything else quiet.

### 5.1 Colour

Tokens on `:root`; a light scheme under `prefers-color-scheme: light`. Dark is
the design; light is a courtesy that must still read.

| Token | Dark (default) | Light |
|---|---|---|
| `--bg` | `#0f1218` | `#f6f7f9` |
| `--bg-raised` | `#131822` | `#ffffff` |
| `--line` | `#222938` | `#dfe3ea` |
| `--text` | `#e6e9f0` | `#161a22` |
| `--text-muted` | `#a7adbb` | `#5b6272` |
| `--text-dim` | `#7d8494` | `#7d8494` |
| `--accent` | `#f0b429` (amber) | `#b7791f` |
| `--accent-ink` | `#0f1218` | `#ffffff` |
| `--ok` | `#7fd6a5` | `#1f8a5b` |

Contrast: `--text` on `--bg` is 14:1; `--text-muted` on `--bg` is 7.6:1;
`--accent-ink` on `--accent` is 10:1. `--text-dim` is for labels 12px and up
only. The accent is used for: the primary button, the language switch's active
state, the eyebrow, the highlighted timeline row, link hover. Nowhere else.

### 5.2 Type

- **Display and body:** Archivo (400, 500, 600) — fallback `'Helvetica Neue',
  Arial, sans-serif`.
- **Data, labels, eyebrows:** JetBrains Mono (400, 500) — fallback `Menlo,
  Consolas, monospace`.
- Loaded from Google Fonts with `display=swap` and a `preconnect`; the fallback
  stacks are chosen for close metrics so the swap does not reflow the hero.
- Scale (px / line-height): 60/1.04 hero h1 (40 on phones), 36/1.15 page h1
  and section h2, 22/1.3 h3, 19/1.5 lead, 16/1.6 body, 14/1.5 small, 12
  labels with `letter-spacing: 0.12em` in mono uppercase.
- Headlines get `text-wrap: balance`; paragraphs `text-wrap: pretty`; measure
  for running text 62–68 characters.

### 5.3 Layout

- Content column 1120px max, 64px side padding on desktop, 20px on phones.
- The header is sticky, 64px tall, with a 1px `--line` bottom border and the
  page background at 92% opacity with `backdrop-filter: blur(8px)`.
- Sections are separated by space (96px desktop, 64px phones), not by rules or
  alternating backgrounds. The single exception is the raised panel surface
  (`--bg-raised` + 1px `--line` + 12px radius) used for the timeline panel,
  the value cards and the component rows.
- Grid: value cards 4-up on desktop, 2-up under 960px, 1-up under 600px. The
  hero is two columns to 960px, then stacks with the panel below the copy.
- Radius: 8px controls, 12px panels. No shadows; depth comes from the raised
  surface and the line.

### 5.4 The architecture diagram

Inline SVG in a partial (`layouts/_partials/diagram-architecture.html`), drawn
once, coloured by tokens via `currentColor` and CSS variables, with the labels
as real `<text>` so they are selectable, searchable and translatable through
`i18n`. Boxes: cockpit → BFF → core; core → Postgres and NATS; core → sandbox
(per demand); Identity Platform at the edge. Arrows are 1.5px strokes with a
small arrowhead; the core box carries the accent border because it is the only
source of truth, and that is the point the diagram makes.

### 5.5 Icons

Inline SVG, stroke-based, 24px grid, 1.8px stroke, `currentColor`. Four for
the value cards, one arrow for buttons, one external-link mark for GitHub. No
icon font, no emoji.

### 5.6 Motion

One reveal: the hero copy and the panel fade and rise 12px over 400ms on
load, staggered by 80ms. Nothing else moves except link and button hover
(colour, 150ms). Everything is disabled under `prefers-reduced-motion`.

### 5.7 Accessibility

- Skip link to `main`. Landmarks: `header`, `nav`, `main`, `footer`.
- Every control has a visible focus ring (2px `--accent`, 2px offset).
- The language switch is a `nav` with `aria-label` and the current language
  marked `aria-current="true"`; the links carry `hreflang` and `lang`.
- The diagram has a `<title>` and a `<desc>`; the timeline panel is a real
  `<table>` with a caption.
- Colour is never the only carrier of meaning: the highlighted timeline row
  also carries the arrow glyph; the status table carries words, not dots.

## 6. Implementation shape

```
assets/css/site.css              the whole stylesheet (Hugo Pipes: minify + fingerprint)
layouts/baseof.html              head, header, footer, skip link — no inline CSS
layouts/home.html                the five home sections
layouts/page.html                article layout for architecture and status
layouts/_partials/header.html
layouts/_partials/footer.html
layouts/_partials/lang-switch.html
layouts/_partials/diagram-architecture.html
layouts/_partials/timeline-panel.html
layouts/_partials/adr-index.html      renders data/adrs.toml
layouts/_partials/subprojects.html    renders data/subprojects.toml
layouts/_partials/icons/*.html
data/adrs.toml
data/subprojects.toml
content/{en,pt-br}/_index.md          hero copy, value props, section leads (front matter + body)
content/{en,pt-br}/architecture.md
content/{en,pt-br}/status.md
i18n/{en,pt-br}.toml                  every interface string
hugo.toml                             [params.github], [params.status] (the three numbers), [menus]
```

Home copy that is structured (four value cards, three closing numbers) goes in
the page's front matter as lists, so the layout iterates and the writer edits
Markdown, not HTML. Prose goes in the body.

The workflow gains nothing new: `make check` already builds and mirrors; lychee
already checks the links the new pages add.

## 7. Out of scope

- A theme toggle. The site follows the system; a control is JavaScript the
  content does not need.
- Plans/pricing and the legal pages — switched off by `[params]`, unchanged.
- Screenshots of the cockpit. The cockpit still runs a placeholder theme; a
  screenshot today would date the site in a week. The timeline panel stands in
  for it, drawn from real event names.
- Analytics, forms, newsletters. Nothing that needs consent.

## 8. Done means

- `make check` green; the deploy's internal link check green.
- Both languages complete for all three pages; the switch lands on the same
  page.
- Lighthouse on the home: accessibility 100, no contrast findings; no layout
  shift from the font swap.
- The page reads at 390px wide with no horizontal scroll.
- Every number on the site has a line in this spec's §3 pointing at its source.

## 9. Amendment (2026-09-17) — the site wears the product's chrome

The owner's review of the first build: *"muita cara de site feito por IA, sem
uma identidade própria"*, with two instructions — use the cockpit's palette,
and give the site an identity of its own. §5 is withdrawn; this section
replaces it.

**The identity is the product.** The cockpit calls itself *DOP IDE* and looks
like one. The site borrows its chrome outright, so a visitor who later opens
the product recognises where they are:

- **A title bar** (56px): the mark and wordmark, then the location as a path
  (`dop-t.com / status`); on the right, *Sign in* when configured, the GitHub
  chip, the language switch, and on phones a *Menu* that is a `<details>` —
  no script.
- **A sidebar** (248px, sticky), grouped as the cockpit groups its own:
  *Site* (Product, Architecture, Status), *Repositories* (the five, each
  tagged with its stack), *Documents* (roadmap, ADRs, PRD, glossary — all
  real links into the umbrella repository). On phones it becomes the menu.
- **An editor pane** holding one document per page: a header with chips
  (`product`, `source: docs/ROADMAP.md`, the pre-release warning), a
  monospaced h1, a lead, then sections with a ruled heading.
- **A status bar** (32px) at the bottom: *built in the open · main · hugo
  <version>*, the other language, legal links when configured, the
  repository. Small, monospaced, every item true.

**Tokens are the cockpit's, verbatim** — the same hsl triplets from
`repos/dop-app/artifacts/dop/src/index.css`, so the two never drift by a
rounding: dark `hsl(222 25% 7%)` ground, `hsl(222 25% 9%)` panels,
`hsl(220 20% 16%)` lines, `hsl(210 20% 90%)` text, `hsl(215 15% 60%)` muted,
primary `hsl(212 100% 60%)`; light from its `:root` block. The semantic
colours are the cockpit's KPI colours (emerald, amber, red, purple, blue —
tailwind 400s in dark, 600s in light). Radius 0.3rem. Inter for the interface,
JetBrains Mono for data — and for headlines, which is the one place the site
departs from the product: a monospaced h1 is what a document looks like
inside an editor.

**Components replaced, and why:**

| Was (§5) | Now | Because |
|---|---|---|
| Amber eyebrow in uppercase mono | Chips (`product`, `source: …`) | The eyebrow is the most recognisable AI-site trope; a chip carries a fact |
| Timeline as a table in a raised panel | The cockpit's *reader strip*: event chips with time, name and actor | It is the product's own component, drawn with the product's own values |
| Four equal icon cards | A key/value list, the shape of a front matter (`operators`, `events`, `sandbox`, `open`) | Four cards in a row is the second trope; a keyed list reads as a document |
| Three big stats on rules | Four KPI tiles as the cockpit draws them (icon in a semantic colour, mono number, uppercase label) | Same information, in the product's vocabulary |
| Centred closing call | None; the page ends in the status bar | A centred restatement with a second button is the third trope |
| Footer with columns | The status bar | See above |

**What did not change:** the three routes, the language switch landing on the
same page, the `[params]` convention, the data files, the content, the
accessibility requirements, the reduced-motion rule, and the done criteria.
