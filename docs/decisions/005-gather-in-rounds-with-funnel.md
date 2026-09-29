# 005: Gather postings in rounds through a relevance funnel, storing as it goes

**Date:** 2026-09-28

## Context

The ATS APIs have no search: one call returns a company's entire board. A run may see tens of thousands of postings, and only a few percent are relevant to the requested role. Title variants ("AI Engineer", "Agent Engineer") must be included, and look-alikes ("AI Sales Engineer") excluded.

## Decision

Gather in **rounds**. Each round:
1. Claims the next ~300 boards from a list shuffled once per run (`run_boards.round`). A board is never fetched twice.
2. Holds the fetched postings **in memory** and runs an **embedding filter** against the brief's role description.
3. Runs a **cheap LLM judge** on the shortlist, which returns keep/drop plus a reason.
4. **Extracts and stores the kept postings in the same round.**
5. Saves a brief row per posting to `candidates`, with a status that only moves forward (`skipped → shortlisted → kept/dropped`).
6. Hands a summary built from the extracted facts to an LLM decision: continue or stop.

Hard limits (rounds, time, postings) are enforced in code. A round's board claims and inserts are committed in one transaction.

## Alternatives

- **Send every posting to the LLM:** accurate but slow and expensive.
- **Keywords only:** cheap, but misses title variants.
- **Store every fetched JD, then filter:** the full text of irrelevant postings would fill the database.
- **Fetch first, extract later in a separate phase:** the "enough?" decision would only see titles, not real levels or salary coverage.

## Consequences

- LLM cost scales with *relevant* postings, not all fetched ones.
- The full text of irrelevant postings is never stored.
- Statuses and board claims make every step idempotent, so a crashed run can resume without double LLM cost.
- The kept/dropped reasons become labeled data for evaluating the relevance filter.
