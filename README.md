# Sansad Record

A citizen-run, self-updating record of Nepal's Federal Parliament: members, bill
status, committees, and a running history of what's changed — hosted free on
GitHub Pages, kept current by GitHub Actions.

## How the "auto-update" actually works

GitHub Pages only serves files — it can't run a server. So instead of a backend,
a scheduled GitHub Action (`.github/workflows/update-data.yml`) runs
`scripts/scrape.py` every 6 hours *on GitHub's own servers*, which:

1. Re-fetches the official parliament pages.
2. Compares the fresh data to what's already in `data/*.json`.
3. Appends any real differences to `data/history/changelog.json` — this is
   your "past history" feed, equivalent to knowyourreps.io's activity log.
4. Commits the updated files.

Pushing to `main` then makes GitHub Pages redeploy automatically. The site
updates itself; you don't have to touch it.

## Before you trust this in production

Being direct about the current state, since accuracy matters a lot for a civic
transparency tool:

- **`data/committees.json`** has real data — the 12 House committee names, taken
  directly from `hr.parliament.gov.np`.
- **`data/members.json`** has only a handful of current officeholders I could
  cross-verify (Speaker, Deputy Speaker, Leader of the House, Leader of the
  Opposition, National Assembly Chair). The full 275+59 roster is intentionally
  left empty rather than guessed — fabricated MP data in a tool like this would
  actively mislead people, which is worse than an honest gap.
- **`data/bills.json`** is an empty, schema-ready file — no bill data has been
  fetched yet.
- **`scripts/scrape.py`** is a working skeleton, not a tested scraper. I could
  confirm `hr.parliament.gov.np` is a plain server-rendered site (no login,
  no JS wall), which means scraping it will work — but I don't have live
  network access in this environment, so the actual CSS selectors in
  `parse_members()` and `parse_bills()` are best-guess, marked `TODO`. The
  very first thing to do is: open the real "List of Members" and "Current
  Status of Bills" pages in a browser, view source, and fix those selectors.
  After that one pass, the schedule above should keep working unattended.
- Nepal's parliament does **not** appear to publish individual per-member
  roll-call votes the way the U.S. Congress does (most bills pass by voice
  vote / show of hands). That's why this tracks bill *status* and *committee
  assignment* rather than "how did my MP vote" — the headline feature of
  knowyourreps.io doesn't have a public data source to build on here, as far
  as I could confirm. If you later find Hansard-style division records do
  exist, that would be the single highest-value feature to add.

## Deploying it

1. Create a new GitHub repo, push everything in this folder to it.
2. Repo Settings → Pages → Deploy from branch → `main` → `/ (root)`.
3. Repo Settings → Actions → General → Workflow permissions →
   "Read and write permissions" (so the scheduled job can commit).
4. Fix the selectors in `scripts/scrape.py` (see above), then either wait
   for the schedule or trigger it manually from the Actions tab
   ("Run workflow").

## Local development

Just open `index.html` in a browser, or serve the folder locally
(`python -m http.server`) so `fetch()` can load the JSON files — opening the
raw file with `file://` will block those fetches in most browsers.

## What's deliberately not here

No login, no database, no server you have to maintain or pay for. Everything
is a flat file; the only moving part is the scheduled Action. If you later add
collaboration or user accounts, that's the point where you'd need to leave
GitHub Pages for something with a real backend.
