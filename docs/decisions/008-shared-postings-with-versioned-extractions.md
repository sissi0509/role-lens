# 008: Store each posting once, version its extractions, and link runs to what they used

**Date:** 2026-09-29
**Supersedes:** part of [003](003-fetch-per-run-with-snapshots.md) (per-run copies of postings)

## Context

Decision 003 gave each run its own copy of every posting, with all rows tagged by `run_id`. In practice, the same role gets re-run often, especially during development ("AI Engineer" today, again tomorrow). Most postings are unchanged between those runs, so copying them per run means **paying for the same LLM extraction again**.

Two more things need handling:
- **Companies edit postings** (new salary range, rewritten requirements), so "the same posting" can change.
- **The extraction prompt changes during development**, and results from an old prompt must not be passed off as results from the new one.

## Decision

Separate **what the posting says** from **what the LLM extracted from it**, and link runs to exactly what they used:

```
runs ──< run_postings >── posting_facts ──< posting_items / posting_skills / posting_locations
                               │  (one row per posting × extractor_version)
                               ▼
                           postings        (raw API data, stored once)
```

- **`postings`:** raw API data only (title, company, URL, date, cleaned JD, etc.). Unique on `(ats, job_id, jd_hash)` and never modified. `jd_hash` is a SHA-256 of the normalized input the LLM reads (title, location, cleaned JD, API salary, serialized as JSON). If a company edits the JD, the hash changes and a new row is created.
- **`posting_facts`:** the LLM's single-value extraction (level, years, salary, workplace, industry). Primary key `(posting_id, extractor_version)`. Bumping `extractor_version` when the prompt or model changes creates new rows. Old ones are never overwritten.
- **`posting_items`, `posting_skills`, `posting_locations`:** carry `(posting_id, extractor_version)`, with a composite foreign key to `posting_facts`.
- **`run_postings (run_id, posting_id, extractor_version)`:** "this report considered this posting, read with this extraction." A run links only to postings its own fetch found and kept.
- **Board size** stays in `run_boards.open_jobs` (it's per run and per board) and is reached via `postings.board_slug`, so it isn't duplicated in the link table.

For each kept posting:

| Situation | Result |
|---|---|
| Same JD text, same extractor version | only a `run_postings` link (no LLM or embedding calls) |
| Same JD text, new extractor version | a new extraction; the raw posting is untouched |
| JD edited by the company | a new posting row → a new extraction → a link |

## Alternatives

- **Per-run copies (the original 003):** simple, but repeated runs re-extract identical postings.
- **Per-run copies plus an extraction cache table:** avoids the LLM cost, but keeps two mechanisms (copying and caching) and duplicates the JD text.
- **Overwrite the extraction in place when the prompt changes:** fewer rows, but old runs silently change underneath their saved reports.
- **Extracted facts as rows in `posting_items`** (one table with a `kind` column, i.e. an entity–attribute–value design): salary becomes text, and simple group-bys need self-joins.
- **Use the API's `updated_at` to detect edits:** it changes for trivial reasons and doesn't prove whether the content changed. The hash compares the actual content.

## Consequences

- Re-running a role costs LLM calls only for new or changed postings. That matters most during development.
- Each run is still an exact snapshot: it links to the precise posting version and extraction it used. 003's concern about "stale data mixing silently" is handled by the hash (content versions) plus the links (a run sees only what its own fetch found).
- Comparing two extractor versions on the same postings works as an eval of a prompt change.
- Queries join through `run_postings` (one extra join).
- Deleting a run no longer deletes postings. A cleanup command for unlinked postings and extractions is needed later.
- Dev helpers: a fixed board-sample seed (so test runs hit the same boards, and links get reused), and `--from-run` (re-run Analyze and Write on an existing run without fetching).
