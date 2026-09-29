# 006: Use an agent only where the next step depends on what was found

**Date:** 2026-09-28

## Context

It's tempting to make every part of an LLM system an "agent" with tools. But agent loops cost more, are less predictable, and are harder to evaluate than fixed steps.

## Decision

Apply one test to each part: **does an LLM decide what happens next?**

| Part | Kind | Tools |
|---|---|---|
| Scope | agent: decides what to ask and when the brief is complete | none (structured output) |
| Gather | small agent (router): the only LLM decision is continue/stop per round | none |
| Analyze | agent: chooses which questions to ask the data next | stats, requirement grouping, semantic search, think |
| Write | single LLM step | none |
| Follow-up chat | agent: the user's question decides the tool | same tools as Analyze |

Built with LangGraph: fixed transitions via `add_edge`, state-based routing via `add_conditional_edges`, and safety limits in code.

## Alternatives

- **A fixed pipeline everywhere:** predictable, but it can't adapt analysis to what the data shows or answer open follow-up questions.
- **A tool-using Gather agent that writes its own candidate searches** (like a deep-research agent's web queries): smarter at finding unusual titles, but it isn't needed unless the embedding filter misses them.

## Consequences

- LLM control goes where it adds value, and fixed steps stay cheap and easy to test.
- The tool-using Gather is an **eval-gated upgrade**: adopt it only if measured recall on unusual titles is poor.
- Asking Analyze to gather more data is a planned upgrade (a tool capped at one extra round). v1 reports low-confidence findings instead.
