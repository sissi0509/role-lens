# Decision Records

One short file per significant design decision, numbered in order: `001-short-title.md`, `002-...`.

Each record answers:

- **Context:** what problem or choice came up
- **Decision:** what we chose
- **Alternatives:** what else we considered, and why not
- **Consequences:** what this makes easier, and what it makes harder

## Index

| # | Decision |
|---|---|
| [001](001-src-package-layout.md) | Use the `src/rolelens/` package layout |
| [002](002-job-data-from-public-ats-apis.md) | Get job postings from public ATS job-board APIs |
| [003](003-fetch-per-run-with-snapshots.md) | Fetch per report run, and store each run as a snapshot |
| [004](004-postgres-pgvector-on-supabase.md) | Use Postgres + pgvector (Supabase) as the only database |
| [005](005-gather-in-rounds-with-funnel.md) | Gather postings in rounds through a relevance funnel, storing as it goes |
| [006](006-agents-only-where-next-step-depends-on-results.md) | Use an agent only where the next step depends on what was found |
| [007](007-extraction-based-chunking.md) | One LLM extraction per posting, with one requirement per vector |
