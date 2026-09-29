# 004: Use Postgres + pgvector (Supabase) as the only database

**Date:** 2026-09-28

## Context

RoleLens needs exact statistics (counts, percentages, salary ranges by level) and search by meaning (grouping similar requirements, finding evidence quotes for follow-up questions).

## Decision

Use one **Postgres database with the pgvector extension**, hosted on **Supabase**. Code connects with a plain Postgres driver (`psycopg`), so the same code runs against a local Postgres in tests. SQL handles the numbers, and pgvector handles meaning-based retrieval (RAG).

## Alternatives

- **A separate vector database (Pinecone, Chroma) next to Postgres:** two systems to keep in sync, for a data size (a few thousand vectors per run) that pgvector handles easily.
- **SQLite:** simplest for local runs, but it would need a migration before deployment and has weaker vector support.
- **Only files (JSON per run):** counting and debugging become awkward Python code instead of SQL.

## Consequences

- One system for exact stats and vector search, joined in a single query when needed.
- The free-tier size limit is comfortable, because only relevant postings are stored (see 005).
- Queries that need exactness use SQL, and only meaning-based questions use vectors.
