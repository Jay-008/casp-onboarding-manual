# Quick Onboarding with CASP

An interactive, searchable onboarding guide for the TCAA Civil Aviation Services Portal (CASP), built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

The landing page introduces CASP (account types, registration prerequisites, the applicant's journey), then offers two routes: a guided walkthrough for new users, or direct links to a specific task for people who already know the portal.

## Run it locally

```bash
source venv/bin/activate   # first time: python3 -m venv venv && pip install mkdocs-material
mkdocs serve --dev-addr 127.0.0.1:8020
```

Open <http://127.0.0.1:8020>.

## Build the static site

```bash
mkdocs build
```

Output goes to `site/` — a plain static site, deployable anywhere (a subdomain, a folder under the CASP portal, internal hosting).

## Editing the content

**See [MAINTAINING.md](MAINTAINING.md)** for step-by-step instructions on updating pages, adding new pages and sections, handling screenshots, and the house style that keeps the guide coherent.

## Layout

```
docs/
  index.md              landing page (About CASP + entry points)
  getting-started/      registration, first login, profile, password reset
  account/              profile, password, entity information
  dashboard/            dashboard orientation
  applications/         starting, drafts, status meanings, issued licences
  billing/              GePG fees and payment
  meetings/             invitations and records
  clarifications/       responding to a TCAA clarification request
  resubmissions/        responding to TCAA requests
  reference/            acronyms, PDF download
  assets/               images, the source PDF, and extra.css
mkdocs.yml              site config and navigation order
overrides/              theme override for the PDF download banner
```

## Scope

Currently documents **Core Functions** — the parts of CASP common to every regulatory service stream. Stream-specific guides (Airworthiness, Flight Operations, etc.) are planned but not started.

## Source material

Derived from the official **CASP Manual 2026 v01 — Core Functions** PDF, a copy of which ships with the site at `docs/assets/manual/` and is downloadable from every page. See [MAINTAINING.md](MAINTAINING.md) for how to pull in a new revision of that source PDF.
