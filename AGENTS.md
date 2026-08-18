# Agent Instructions

## Purpose

Maintain a public, portable knowledge base for agent setup and HCL Notes/Domino development.

## Core principles

The first four rules are the Karpathy-inspired baseline. Rules 5–12 are community extensions for longer agent workflows.

1. Think before coding: state assumptions, ambiguity, and tradeoffs instead of guessing.
2. Prefer the smallest solution that satisfies the request; add nothing speculative.
3. Make surgical changes, clean up only your own changes, and avoid unrelated refactors.
4. Define verifiable success criteria and iterate until they are met.
5. Use the model for judgment; use deterministic code and tools for routing, retries, transforms, and checks.
6. Treat explicit token budgets as hard limits; avoid repeated rereads and summarize before context drifts.
7. Surface conflicting sources or code patterns; choose the newer or better-tested source and explain why.
8. Read exports, immediate callers, shared utilities, and adjacent tests before editing.
9. Tests should encode why behavior matters and fail when the intended logic changes.
10. After a significant step, state what changed, what was verified, and what remains.
11. Match the codebase's conventions; flag a harmful convention instead of silently creating a competing pattern.
12. Fail loud: report skipped checks, partial results, and uncertainty instead of claiming completion.

## Public-repository safety

- Never commit credentials, tokens, cookies, private URLs, internal hostnames, personal memory, customer information, or real production configuration.
- Use placeholders such as `<username>`, `<host>`, `<token>`, and `ExampleCorp`.
- Do not add screenshots unless their provenance and visible data have been reviewed.
- Run Gitleaks and inspect the staged diff before every publication batch.

## Content conventions

- Use Traditional Chinese for explanatory notes unless preserving an English API name.
- Use Obsidian wikilinks for internal notes and Markdown links for external sources.
- Prefer primary sources and record `last_verified` when an installation command may change.
