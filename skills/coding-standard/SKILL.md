---
name: coding-standard
description: Use when writing, reviewing, or refactoring code to match this developer's code style and architecture rules. Covers naming, function design, architecture fit, dependencies, and error handling.
---

# Coding Standards

## General

- Prefer simple, readable, maintainable solutions.
- Understand existing code before modifying it.
- Follow existing project conventions before introducing new ones.
- Make the smallest change that correctly solves the task.
- Do not refactor unrelated code.
- Do not introduce abstractions without a concrete reason.
- Do not add dependencies unless necessary.
- Preserve existing behavior unless the task requires changing it.

## Code Style

- Prefer clear and descriptive names.
- Keep functions focused.
- Avoid unnecessary nesting.
- Avoid duplicated logic when a reasonable existing abstraction already exists.
- Keep implementations explicit rather than overly clever.
- Prefer composition over unnecessary inheritance.
- Avoid premature optimization.
- Keep comments focused on explaining why, not what.
- Do not add comments that merely restate the code.

## Architecture

- Respect the existing architecture.
- Do not introduce a new architectural pattern without first determining whether it is appropriate.
- Do not introduce classes, dependency injection, repositories, services, or factories merely because they are common.
- Reuse existing project abstractions when appropriate.
- Keep responsibilities separated according to the established architecture.
- Avoid mixing unrelated responsibilities in the same module.

If a design pattern would meaningfully improve the implementation, explain the trade-offs before introducing it.

## Existing Code

- Understand how the current implementation works.
- Identify its callers and dependencies when relevant.
- Check whether changing it affects API contracts or other modules.
- Do not rewrite working code to match personal preferences.

## API Changes

- Preserve existing endpoints unless the task requires changing them.
- Preserve request and response structures unless explicitly required.
- Preserve existing error behavior when possible.
- Validate input at system boundaries.
- Return appropriate errors instead of silently failing.
- Do not expose internal implementation details unnecessarily.
- Preserve authentication and authorization behavior.
- Do not weaken existing security controls to make implementation easier.

## Database Changes

- Inspect the existing schema before making changes.
- Preserve existing relationships and constraints unless the task requires changing them.
- Prefer database constraints for data integrity when appropriate.
- Consider indexes for frequently queried fields.
- Check migration implications before modifying models.
- Do not modify production data directly unless explicitly requested.

## Error Handling

- Handle expected errors explicitly.
- Do not silently swallow errors.
- Preserve useful error context.
- Avoid broad catch blocks when more specific handling is appropriate.
- Do not expose sensitive internal errors to API consumers.

## Security

Always consider authentication, authorization, input validation, SQL/database injection, command injection, path traversal, sensitive information exposure, credential handling, and token/session security.

Never hardcode secrets, API keys, passwords, or tokens.

## Dependencies

Before adding one:

1. Check whether the project already has a dependency that solves the problem.
2. Check whether it can reasonably be implemented without one.
3. Consider maintenance and project complexity.
4. Add it only when it provides meaningful value.

Do not add libraries for trivial functionality.

## Ambiguity

If ambiguity could materially affect the implementation, identify it, explain the options briefly, and ask. For minor details, choose what matches existing project conventions.
