# Maintaining this guide

How to update or add content without breaking the look, the navigation, or the voice of the site.

This file is for maintainers. It lives outside `docs/`, so it never appears on the published site.

---

## 0. Set up and run locally (do this first, every time)

```bash
cd "/Users/mac/Desktop/Old Pc/ICT DEV CASP/CASP Interactive Manual"
source venv/bin/activate
mkdocs serve --dev-addr 127.0.0.1:8020
```

Open <http://127.0.0.1:8020>. Leave it running — it live-reloads on every save, so you see changes as you type.

First time on a new machine:

```bash
python3 -m venv venv && source venv/bin/activate && pip install mkdocs-material
```

---

## 1. Which kind of change are you making?

| You want to… | Go to |
|---|---|
| Fix a typo, update a step, swap a screenshot on an existing page | [Section 2](#2-edit-an-existing-page) |
| Add a new task/article to a section that already exists | [Section 3](#3-add-a-new-page) |
| Add a whole new section (e.g. an Airworthiness stream guide) | [Section 4](#4-add-a-new-section) |
| Add or replace screenshots | [Section 5](#5-screenshots) |
| **Roll a whole new revision of the source PDF into the site** | **[Section 8](#8-updating-the-site-from-a-new-source-pdf)** |
| Swap the downloadable PDF for a newer one | [Section 9](#9-the-downloadable-pdf) |

Whatever you do, finish with [Section 6 — verify](#6-verify-before-you-publish).

---

## 2. Edit an existing page

1. Find the file. Page URL maps directly to the file path:
   `/billing/pay-fee/` → `docs/billing/pay-fee.md`
2. Edit the Markdown.
3. Save — the browser reloads automatically. Check it.

**No `mkdocs.yml` change is needed** when you're only editing an existing page's content.

---

## 3. Add a new page

Three steps. Miss step 3 and the page exists but nobody can navigate to it.

### Step 1 — Create the file

Put it in the folder for its section, named after the task in lowercase-with-hyphens:

```bash
# example: adding "Renew a licence" to the Applications section
touch docs/applications/renew-licence.md
```

### Step 2 — Write it using the house template

Every task article follows the same shape. Copy this:

```markdown
# Renew a licence

One or two sentences: what this task is and when you'd do it.

!!! tip "Before you start"
    Anything the user must have ready first. Delete this block if there's nothing.

## 1. First stage of the task

1. Do this thing.
2. Then this thing.

    ![Descriptive alt text](../assets/images/screenshot-name.png)

3. Click **Button Name**.

## 2. Second stage of the task

1. Continue…

## What happens next

Where the user ends up, and where to go from here — with a link.
See [Application status meanings](statuses.md).
```

### Step 3 — Register it in the navigation

Open `mkdocs.yml` and add it under the right section in `nav:`:

```yaml
  - Applications:
      - Start a new application: applications/start-application.md
      - Save and resume drafts: applications/drafts.md
      - Renew a licence: applications/renew-licence.md   # ← new
      - Application status meanings: applications/statuses.md
```

**Order matters.** `nav:` order controls both the sidebar order *and* the Next/Previous buttons at the bottom of each page — which is how a first-time user walks the whole guide. Put new pages in the order someone would actually do them, not alphabetically.

---

## 4. Add a new section

For example, the stream-specific guides listed in `docs/reference/roadmap.md`.

1. Create the folder and its pages:

    ```bash
    mkdir -p docs/streams/airworthiness
    ```

2. Add the section to `nav:` in `mkdocs.yml`. Place it **after** Document Resubmission and **before** Reference, so Core Functions still reads as one continuous walkthrough:

    ```yaml
      - Document Resubmission:
          - Respond to a resubmission request: resubmissions/respond.md
      - Airworthiness:                                    # ← new section
          - Certificate of Airworthiness: streams/airworthiness/coa.md
          - Aircraft registration: streams/airworthiness/registration.md
      - Reference:
          - Acronyms glossary: reference/acronyms.md
    ```

3. Cross-link it from the relevant Core Functions page. A new stream guide is useless if nobody finds it — add a pointer in `docs/applications/start-application.md`.

4. Add a card for it on the home page, in the `Quick access` grid in `docs/index.md`.

5. Remove it from the "planned" list in `docs/reference/roadmap.md`.

---

## 5. Screenshots

### Where they go

All images live flat in `docs/assets/images/`. Reference them with a relative path — `../assets/images/` from any article, `assets/images/` from `index.md`.

### Naming

Name by **what it shows**, not where it came from:

- Good: `request-control-number-modal.png`, `drafts-tab.png`
- Bad: `screenshot-12.png`, `page22-img3.png`

### Pulling screenshots out of a source PDF

To extract from a source manual:

```bash
# needs poppler: brew install poppler
mkdir -p assets_raw/2026v01
pdfimages -png -p "CASP Manual-2026_v01-Core_Functions.pdf" assets_raw/2026v01/shot
```

This produces `assets_raw/2026v01/shot-PPP-NNN.png` (PPP = the **PDF** page number, which is usually 3 higher than the printed page number — the cover, contents, and acronyms pages come first).

Two things to know before you go hunting:

- Every page yields four tiny images (roughly 5–27 KB) that are just the letterhead — coat of arms, TCAA logo, ISO mark. Ignore anything under ~60 KB:

    ```bash
    ls -la assets_raw/2026v01 | awk '$5 > 60000 {print $9, $5}'
    ```

- Screenshots come out in reading order per page, so `shot-009-041` and `shot-009-042` are the first and second screenshots on PDF page 9.

Copy the ones you want into place with a real name:

```bash
cp assets_raw/2026v01/shot-008-034.png docs/assets/images/otp-verification.png
```

Keep the extraction folder per-revision (`2026v01/`, `2027v01/`, …) so you can diff revisions later. `assets_raw/` is gitignored — it's a scratch area, not part of the site.

### Screenshots supplied outside the PDF

Sometimes a colleague sends screenshots the manual doesn't have. Same rules apply — name by what they show, check for real data, drop them in `docs/assets/images/`, and note in the article where they came from if the PDF doesn't back them up.

### Before you use one

- **Check it for real data.** Source screenshots may contain applicant names, emails, or licence numbers. Blur or re-shoot anything identifying.
- Prefer screenshots that already have the red highlight boxes/arrows — they're doing useful work.
- The site adds a subtle border and rounded corners automatically. Don't bake borders into the image.

---

## 6. Verify before you publish

Run all four. The first one catches the mistakes that matter most.

```bash
# 1. Strict build — fails on broken internal links and bad nav paths
mkdocs build --strict
```

2. **Click through the affected pages** in the browser at <http://127.0.0.1:8020>.
3. **Check the Next/Previous buttons** at the bottom of your new page point somewhere sensible.
4. **Search for a phrase** from your new page in the site search box — confirm it's indexed and findable.

Also worth a look if you touched layout or the home page:

- Toggle dark mode (moon icon, top bar) — the TCAA logo should invert to white.
- Narrow the window to phone width — cards and the journey pipeline should stack to one column.

---

## 7. Publish

```bash
mkdocs build
```

Everything needed is written to `site/`. That folder is a plain static site — no server, no database, no build step at the far end. Copy it to wherever it's hosted (a subdomain, a folder served by the CASP portal, or internal hosting).

Do not edit anything inside `site/` by hand; it is wiped and regenerated on every build.

---

## 8. Updating the site from a new source PDF

This is the big one: TCAA issues a revised manual and the site has to catch up. Work through it in order — the sequence matters, because step 2 tells you how much of steps 4–7 you actually need to do.

Worked example throughout: the jump to `CASP Manual-2026_v01-Core_Functions.pdf`.

### Step 1 — Put the new PDF where the old one was

Drop it in the project root, alongside `mkdocs.yml`. Keep the publisher's filename; the version is in it, and you'll want it when someone asks which revision the site reflects.

Don't delete the previous PDF yet. You need it for step 2.

### Step 2 — Diff it against what's already published

**Do not start editing pages yet.** Read the new PDF end to end first and write down what changed. The output of this step is a list, and everything after this is working through that list.

Read the table of contents of both revisions side by side — new sections show up immediately as new headings and shifted page numbers. Then read the body, and sort every difference into one of five buckets:

| Bucket | What it means | What it costs you |
|---|---|---|
| **New procedure** | A section that has no page on the site | A new page (Section 3) |
| **Changed procedure** | Steps, button labels, or screens differ from the published page | Edit the page, replace the screenshot |
| **New detail on an existing procedure** | Same steps, more facts (a limit, a warning, an extra field) | Add a sentence or an admonition |
| **Resolved gap** | Something listed in `docs/reference/roadmap.md` is now documented | Write it up, then delete it from the roadmap |
| **Still missing** | Empty heading, `[Insert Screenshot]` placeholder, contradiction | Do **not** invent it — record it in the roadmap |

Two things to watch for specifically, because they bite:

- **Renamed UI labels.** These are easy to miss and they're the ones that strand a user mid-task. In the 2026 v01 pass, the entity menu item published as *Shareholders* had become **Client Employees** — the article was sending people to a tab that no longer existed.
- **The source contradicting itself.** 2026 v01 calls a returned application *Document Resubmission Required* in chapter 6 and *Returned for Resubmission* in the status table in chapter 5. Don't silently pick one. Document both and say they mean the same thing — a user staring at a badge that doesn't match the guide will call the Help Desk.

### Step 3 — Extract the new screenshots

See [Section 5](#5-screenshots). Do it now, while the diff is fresh — you'll know which figures you need.

### Step 4 — Write the new pages

Follow [Section 3](#3-add-a-new-page) for each **new procedure** on your list: create the file, use the house template, register it in `nav:`.

Place new pages in the order a user meets them, not at the end of their section. *Reset a forgotten password* belongs after *First login*, because that's when you need it.

### Step 5 — Edit the existing pages

Work through the **changed** and **new detail** buckets. For each one:

1. Map the source section to the page — `/billing/pay-fee/` → `docs/billing/pay-fee.md`.
2. Make the edit in the house voice. Don't paste the manual's prose in; the source writes *"the applicant is notified by email"* and this site writes *"you'll be notified by email"*.
3. Replace the screenshot if the screen changed.
4. Add a cross-link if the new material touches another page's task.

### Step 6 — Reconcile the roadmap

`docs/reference/roadmap.md` is the site's honesty ledger, and it goes stale faster than anything else. Every time you update from a new source:

- **Delete** each gap the new revision closed.
- **Add** each gap it opened or left open — including headings that exist with no content beneath them.
- **Record** what you just documented under *Documented since the last source revision*, so the next maintainer can see the site moving.

### Step 7 — Update the version markers

Three places name the revision. Miss one and the site quietly claims to be something it isn't:

| File | What to change |
|---|---|
| `mkdocs.yml` | `extra.manual_pdf` and `extra.manual_version` |
| `docs/reference/download.md` | Filename in the download link, version, page count, file size, and the status note |
| `docs/index.md` | The source manual named in the *What's covered right now* admonition |

Then swap the downloadable copy — [Section 9](#9-the-downloadable-pdf).

### Step 8 — Verify

[Section 6](#6-verify-before-you-publish), plus two checks specific to a source update:

- Search the site for a phrase from each **new** page. If it isn't findable, users won't find it either.
- Walk the Next/Previous chain from Home to the last page. A new page inserted mid-sequence is the easiest way to break that walkthrough.

---

## 9. The downloadable PDF

Every page carries a **Download this manual as a PDF** bar at the top, and `docs/reference/download.md` is the full download page.

### How it's wired

| Piece | File |
|---|---|
| The file that gets served | `docs/assets/manual/CASP-Manual-2026_v01-Core-Functions.pdf` |
| Path + version label | `extra.manual_pdf` and `extra.manual_version` in `mkdocs.yml` |
| The top bar | `overrides/main.html` (Material's `announce` block) |
| Bar and card styling | `.md-banner`, `.pdf-announce`, `.pdf-card` in `docs/assets/extra.css` |
| The download page | `docs/reference/download.md` |

Anything in `docs/` is copied to `site/` verbatim, so the PDF ships with the build — no plugin, no extra step.

### Swapping in a new PDF

```bash
cp "CASP Manual-2027_v01-Core_Functions.pdf" \
   docs/assets/manual/CASP-Manual-2027_v01-Core-Functions.pdf
rm docs/assets/manual/CASP-Manual-2026_v01-Core-Functions.pdf
```

Then update `extra.manual_pdf` and `extra.manual_version` in `mkdocs.yml`, and the details on `docs/reference/download.md`.

Use hyphens rather than spaces in the filename — a space becomes `%20` in the URL and makes the link fragile.

Two things to check:

- **Only one PDF in that folder.** It's copied into the build as-is, so a leftover 13 MB file doubles the site's weight for no reason.
- **`extra.manual_pdf` is a path, not a URL.** `overrides/main.html` runs it through Jinja's `| url` filter, which is what makes the link resolve correctly from a page three levels deep. Don't hardcode a leading `/`; that breaks the site if it's ever hosted in a subfolder.

### If you change the theme override

`theme.custom_dir: overrides` is what enables `overrides/main.html`. If you remove one, remove the other, or the build fails.

---

## House style — the rules that keep it coherent

These are the conventions the existing pages already follow. Matching them is what makes a new page feel like part of the same guide rather than a bolt-on.

**Voice**

- Address the reader as **you**. "You'll receive an email", not "the applicant will receive an email".
- Use the imperative for actions: "Click **Submit**", not "The user should click Submit".
- Plain language over regulatory register — this is a how-to, not the Act. Save formal phrasing for the home page's About section, where it's appropriate.

**Structure**

- One page = one task the user is trying to finish.
- Number the steps. Numbered steps are the backbone of every article.
- Bold every literal UI label exactly as it appears on screen: **New Application**, **Request Control Number**, **Go Perform Task**.
- Put the screenshot immediately *after* the step it illustrates, indented 4 spaces so it nests inside the numbered item.
- End with a "What happens next" or "Next step" pointing onward.

**Admonitions** — use the right one, sparingly:

| Type | Use for |
|---|---|
| `!!! tip` | Helpful but optional advice, prerequisites |
| `!!! warning` | Consequences of inaction (missed deadlines, delays) |
| `!!! danger` | Irreversible actions (e.g. category can't be changed after Proceed) |
| `!!! info` | Scope notes and known gaps |
| `!!! success` | Confirming a stage is complete, pointing to the next one |

**Linking**

- Link the first mention of a related task, using relative paths: `[Pay an application fee](../billing/pay-fee.md)`.
- Link `.md` files, not URLs — `--strict` then catches it when a link breaks.

**When you don't know something** — this is the important one:

Do **not** guess at portal behaviour. If the source material doesn't cover it, say so explicitly and add it to `docs/reference/roadmap.md`:

```markdown
!!! info "Not covered in the source manual"
    The exact mechanics of **+ Add attendee** (free-text or a list of registered users)
    aren't documented yet.
```

A named gap is honest and actionable. An invented step sends a user down the wrong path and costs TCAA a support call.

---

## Source material and its limits

Content is derived from **`CASP Manual-2026_v01-Core_Functions.pdf`** (in the project root, and served from the site as a download), plus the official `readme.html` onboarding page in the parent `ICT DEV CASP/` folder.

Be aware of what this revision still isn't:

- It is labelled *Release Version 1.0*, but **Effective Date**, **Approved By**, and **Review Date** are all blank. It has not been formally approved.
- **Section 6.4, "Managing Issued Licences, Permits and Certificates", is a heading with nothing under it.** The entire issued-credential lifecycle — viewing, downloading, renewing, and the Licences/Certificates/Expired tabs — is undocumented. It's the largest single gap in the site.
- Section 2.3.2 ends with an **`[Insert Screenshot]`** placeholder.
- The manual contradicts itself on the resubmission status name (chapter 5 vs chapter 6).

Treat it as a starting point, and verify against the live portal where you can.

Some material on the site comes from screenshots supplied directly rather than from the PDF — the per-tab detail on `docs/account/entity-information.md`, for instance. That's fine, but it means the PDF alone won't tell the next maintainer where every fact came from. When you add something the source doesn't cover, keep the wording specific enough that it can be checked against the portal.

`docs/reference/roadmap.md` is the running list of everything visible in the CASP interface that isn't documented yet. Keep it current — when you document something, remove it from that list; when you spot a new gap, add it.
