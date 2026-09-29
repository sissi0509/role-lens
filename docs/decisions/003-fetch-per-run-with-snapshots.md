# 003: Fetch per report run, and store each run as a snapshot

**Date:** 2026-09-28

## Context

Job postings go stale within weeks, and each report is about one role. A report run can take several minutes (5–20 minutes is acceptable to the user).

## Decision

Fetch postings **when a report is requested**, not ahead of time. Everything a run stores is tagged with a `run_id`, and a report uses only its own run's data. Old runs are kept as history (a few MB each). The temporary `candidates` table is deleted after each run.

## Alternatives

- **A nightly sweep of every board into the database:** fast answers, but hundreds of thousands of rows (too big for a free hosted database), LLM extraction for postings nobody asks about, and a scheduler to run.
- **One shared posting table reused across reports:** data from different dates mixes silently, so reports quietly use stale postings.

## Consequences

- Every report uses fresh data, and there's no background job.
- Runs are reproducible, which makes debugging easier and gives evals a fixed dataset.
- Trend comparisons across runs ("October vs December") become possible later at no extra cost.
- Each report pays the fetch time. Gathering in rounds keeps it bounded (see 005).
