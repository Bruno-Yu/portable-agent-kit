# Agent Instructions

## Purpose

Maintain a public, portable knowledge base for agent setup and HCL Notes/Domino development.

## Core principles

1. Think before coding: state assumptions and surface ambiguity.
2. Prefer the smallest solution that satisfies the request.
3. Make surgical changes and preserve surrounding conventions.
4. Define success criteria and verify them before reporting completion.
5. Use deterministic tools for deterministic checks.
6. Keep always-loaded context small; load deeper references only when needed.
7. Surface conflicting sources and prefer the newer, primary, verified source.
8. Read immediate context before editing.
9. Tests should encode why behavior matters.
10. Report skipped checks and uncertainty explicitly.

## Public-repository safety

- Never commit credentials, tokens, cookies, private URLs, internal hostnames, personal memory, customer information, or real production configuration.
- Use placeholders such as `<username>`, `<host>`, `<token>`, and `ExampleCorp`.
- Do not add screenshots unless their provenance and visible data have been reviewed.
- Run Gitleaks and inspect the staged diff before every publication batch.

## Content conventions

- Use Traditional Chinese for explanatory notes unless preserving an English API name.
- Use Obsidian wikilinks for internal notes and Markdown links for external sources.
- Prefer primary sources and record `last_verified` when an installation command may change.
