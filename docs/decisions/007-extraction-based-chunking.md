# 007: One LLM extraction per posting, with one requirement per vector

**Date:** 2026-09-28

## Context

The report needs the most common requirements and responsibilities across postings. A JD mixes 10–20 different requirements, worded differently from company to company.

## Decision

- **Clean, don't segment.** Code decodes HTML and removes recognizable boilerplate (equal-opportunity and privacy notices, how to apply) but keeps pay sections. The LLM reads the **whole** cleaned JD.
- **One structured-output call per posting** (`PostingExtraction`) returns level, years of experience, stated salary, workplace, industry, locations, **items** (one requirement/responsibility/nice-to-have each, worded close to the JD), and **skills** tagged `technical`, `soft` or `domain_knowledge`. Code fills every field already known from the API, and API salary wins over extracted salary.
- **Extraction-based chunking:** each item is one row in `posting_items` with its own embedding, the only vector column. The most common requirements are found by grouping similar items and counting distinct postings per group.
- **Skills:** technical names are open, then normalized by an alias map. Soft skills come from a small fixed list. Skills get exact SQL counts.
- **Throughput:** parallel single-posting calls with a concurrency limit. Embeddings are sent in large batches.

## Alternatives

- **One vector per whole JD:** blurs many ideas into one vector, so requirements can't be compared.
- **Fixed-size chunks:** cut through ideas at arbitrary points.
- **Code segmentation by headings:** headings vary and many JDs have none, so requirements outside the expected sections would be silently lost.
- **Several postings per LLM call:** items can be attached to the wrong posting, and one failure spoils the whole batch.
- **The provider's batch API:** cheaper, but results take hours.

## Consequences

- Requirements can be grouped by meaning and quoted as evidence, while skills get exact counts.
- Accuracy is measurable against hand-labeled postings. If one call can't handle all the fields well, the fallback is to split it into two calls.
- Soft-skill counts are fuzzier than technical ones and are reported as indicative.
