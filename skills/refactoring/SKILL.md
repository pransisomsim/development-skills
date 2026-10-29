---
name: refactoring
description: Use when restructuring existing code without changing behavior. Covers preserving contracts, incremental change, impact analysis, and verification. Triggers on "refactor", "clean up", "restructure", "extract", "rename".
---

# Refactoring

## Preserve

- Behavior, unless explicitly instructed otherwise.
- API contracts, unless required.
- Database behavior, unless required.
- Existing error behavior when possible.

## Process

1. Inspect the current implementation.
2. Identify affected files and modules.
3. Explain architectural implications.
4. Make incremental changes.
5. Verify each meaningful change.
6. Inspect the final diff.

## Rules

- Avoid combining unrelated refactors.
- Do not use a refactor as an excuse to redesign unrelated architecture.
- Do not add abstractions without a concrete reason.
- Do not add dependencies unless necessary.
- Do not discard user changes.
- Do not rewrite history or use destructive git commands.

## Verification

- Run the most relevant tests.
- Run linting and type checks when available.
- Check the final diff.
- Verify that unrelated files were not modified.

Do not claim something was tested if it was not actually tested.
