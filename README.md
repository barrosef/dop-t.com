# dop-t.com — the DOP platform's site

A Hugo static site, published to GitHub Pages on push to `main`. It is the
product's public face: what DOP is, what it costs, and the legal pages the
platform's own sign-in depends on.

It has no backend, no database and no account of its own. Whoever signs in leaves
this site for the cockpit.

## Run

```bash
make serve     # live preview, drafts included
make build     # what CI builds
make check     # build + the translation mirror — run this before pushing
make test      # the translation checker's own test suite
```

## Two languages, checked

`content/en/` and `content/pt-br/` are both first-class, paired by
`translationKey` in each page's front matter. `make check` fails when a page
exists in one language and not the other.

The check is not bureaucracy. A half-translated product site does not degrade
gracefully — the missing half is exactly what a prospect happens to land on, and
what they see is a 404 or an English page where they expected Portuguese. The
same discipline runs in the cockpit, for the same reason.

## What lives where

| | |
|---|---|
| Pages, per language | `content/en/`, `content/pt-br/` |
| Interface strings | `i18n/en.toml`, `i18n/pt-br.toml` |
| Layouts and partials | `layouts/` |
| Content that is configuration | `hugo.toml`, under `[params]` |
| The design spec | `docs/superpowers/specs/2026-09-16-site-design.md` — the site wears the product's chrome (§9) |
| The stylesheet | `assets/css/site.css` — tokens first, then components; served minified and fingerprinted |
| Data the layouts render | `data/adrs.toml`, `data/subprojects.toml` — bilingual columns |
| Specs for work on this site | `docs/superpowers/specs/` |
| The custom domain | `CNAME` |

## The `[params]` convention

Every block driven by `[params]` renders **only when its value is non-empty**. A
blank disappears; it never becomes an empty box or a bracketed placeholder.

This is what lets the site ship before every section has real content, without
looking unfinished. Adding a section means filling a value, not editing a
layout.

## Two dependencies that are not obvious

**The privacy policy blocks production sign-in.** Google's OAuth verification —
the step that takes the consent screen out of *Testing* and removes the
seven-day token expiry — requires a privacy policy hosted on a domain the
applicant owns. Until this site serves one, the platform's Google sign-in stays
limited to named test users. The site is upstream of the product going live.

**The plan names are duplicated on purpose, and must not drift.** The four plans
appear here as marketing copy and in the platform's database as a catalog the
sign-up wizard reads. Nothing links them. The wording may differ; the **set of
names** may not. A plan added or renamed changes in both places, or this site
advertises something nobody can buy.

## Deploy

Push to `main`. `.github/workflows/deploy.yml` builds with a **pinned** Hugo
version and publishes to GitHub Pages. The pin is deliberate: a static site that
rebuilds on every push is a site that can break without anybody changing it.
