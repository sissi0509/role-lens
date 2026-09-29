# 002: Get job postings from public ATS job-board APIs

**Date:** 2026-09-28

## Context

RoleLens analyzes a role across many real job postings, so it needs full job-description text, posting dates and locations from many companies of every size. It must be legitimate, free or cheap, and reliable.

## Decision

For v1, use the **official public job-board APIs of Greenhouse, Lever and Ashby**:
- Greenhouse: `boards-api.greenhouse.io/v1/boards/{slug}/jobs?content=true`
- Lever: `api.lever.co/v0/postings/{slug}`
- Ashby: `api.ashbyhq.com/posting-api/job-board/{slug}?includeCompensation=true`

They're free, need no key, and return full JD text in the list call.

These APIs work per company, so RoleLens needs a list of board slugs. The list comes from the [job-board-aggregator](https://github.com/Feashliaa/job-board-aggregator) company lists, which were built from Common Crawl, so they aren't hand-picked. The lists are CC BY-NC 4.0 and credited. During development, each run samples boards at random.

Each ATS gets its own provider module that returns one shared posting shape, so new sources can be added as new modules.

## Alternatives

- **Scraping LinkedIn, Indeed or Glassdoor:** login walls and terms of service that forbid it. Rejected.
- **A hand-picked company list:** simple, but it biases the sample toward whatever companies were picked.
- **JSearch (Google for Jobs via RapidAPI):** covers big tech and large enterprises with full JDs, but it's a third-party middleman with a small free tier. Deferred to a later version as a second provider.
- **Snippet-only APIs (Adzuna, Jooble, Careerjet):** no full JD text, which extraction needs.
- **Static datasets (e.g. Kaggle LinkedIn 2023–24):** frozen before the current hiring market.

## Consequences

- Free, legitimate, structured data with full JDs and real posting dates.
- Coverage leans toward startups and mid-size tech companies. Big tech runs its own career sites, and many large enterprises use Workday, so neither is covered yet. Reports must state their sample, and big-tech coverage is planned via more providers.
- Only currently open postings are visible, so long-term history exists only if RoleLens keeps its own snapshots.
- Requests must be polite: limited concurrency per host, timeouts, retries, and skipping dead boards.
