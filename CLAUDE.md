# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A prototype decision-support tool for Cambodia's CDC/IPM QIP (Qualified Investment
Project) tax and customs incentives, built under Sub-Decree 139 (Annexes 1–4). Given
a user's investment activity, QIP status, product market, capital, and turnover, it
outputs an incentive matrix (Corporate Tax / Import Duty / Export Duty, plus
eligibility warnings).

There is no build system, package manager, framework, or backend. Every "app" is a
single self-contained `.html` file with inline `<style>` and `<script>` — open it
directly in a browser to run it. There is nothing to install, build, lint, or test.

## Repo layout — read this before editing

The repo currently holds **several independent, non-interchangeable prototypes** of
the same tool, at different stages of completeness. They are not a single app split
into files — do not assume changing one updates the others.

- `index.html` — the file GitHub Pages actually deploys (see below). Currently
  contains only a **5-row sample `ACTIVITIES` dataset** as a placeholder
  (comment: "Replace this sample with your full Annex 492 dataset").
- `qip1.html` — the most complete prototype: full Annex 1–4 dataset embedded as
  `ACTIVITIES` (~490 activities, each `{annexNo, id, number, title, criteria, isic}`),
  activity search/select UI, print/PDF and JSON export buttons.
- `.github/workflows/CDC_IPM_Incentives_Finder_WORKING_v2.html` and
  `.github/workflows/CDC_IPM_Incentives_Finder_Mobile_Matrix.html` — two more
  standalone prototypes with different UI themes (dark/"matrix" style) and different
  eligibility logic (form-driven checklist rather than Annex-lookup). **These are
  HTML pages, not GitHub Actions workflows** — they only live under
  `.github/workflows/` because they were uploaded there by mistake. GitHub Actions
  ignores non-YAML files in that directory, so they have no effect on CI/deploy;
  don't confuse them with `static.yml`.
- `.github/workflows/static.yml` — the actual GitHub Actions workflow (see below).

When asked to "fix the incentive calculator" or similar, ask/confirm which file is
in scope, since fixing `qip1.html` will not change what's deployed at the Pages URL
unless it's also copied into (or replaces) `index.html`.

## Deployment

`static.yml` deploys the **entire repository root** to GitHub Pages on every push to
`main` (or manual `workflow_dispatch`), via `actions/upload-pages-artifact` +
`actions/deploy-pages`. GitHub Pages serves `index.html` as the site root, so
**`index.html` is the only file end users actually see** — the others are only
reachable by direct URL (including the two prototypes accidentally nested under
`.github/workflows/`, which are still served as static files by Pages since the
whole repo is uploaded as the artifact).

There's no separate staging/build step: whatever is committed to `index.html` on
`main` is what goes live.

## Core logic pattern (shared across prototypes)

Each file follows the same shape, just with different dataset/UI polish:

1. An `ACTIVITIES` array is the Annex 1–4 dataset, each entry tagged with an
   `annex`/`annexNo` (1–4) and a free-text `criteria` string (e.g.
   `"Minimum capital USD 2,000,000"`) that is parsed at runtime (see
   `extractCapitalThreshold()` in `index.html`/`qip1.html`) rather than stored as
   structured fields.
2. A `generate()` function branches on the selected activity's annex number:
   - **Annex 1** = negative list → not eligible for anything.
   - **Annex 2** = tax holiday tiers (grouped by `G1`/`G2`/`G3` in the activity `id`
     → 9/6/3 years) plus import/export duty rules.
   - **Annex 3** = customs/VAT exemption only, no CIT tax holiday; capital compared
     against the parsed threshold to warn if below minimum.
   - **Annex 4** = domestic production input incentives; CIT depends on the
     user-selected QIP status dropdown rather than the annex itself.
   - A turnover check (`> 4,000,000,000` Riel) appends a "Patent: R 3m" note to the
     Corporate Tax result in every branch.
3. Results are written directly into table cells via `innerText`/`innerHTML` — there
   is no templating library or state management; UI updates are plain DOM writes
   triggered by inline `onclick`/`addEventListener` handlers.

If you extend the incentive rules, keep this pattern (dataset entry → annex branch →
direct DOM write) unless asked to restructure the app.
