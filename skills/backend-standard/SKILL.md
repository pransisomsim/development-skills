---
name: backend-standard
description: Backend development standards for this developer
---

# Backend Standards

Follow these rules when working on backend code.

## Architecture

- Prefer modular architecture.
- Keep controllers thin.
- Keep business logic in services.
- Keep database access in repositories.
- Do not access Prisma directly from controllers.
- Do not put business logic inside routes.
- Prefer function-based modules unless a class is justified.

## API

- Preserve existing response shapes unless explicitly instructed otherwise.
- Validate request input at the boundary.
- Return appropriate HTTP status codes.
- Preserve existing error behavior during refactors.

## Database

- Use database constraints when the rule belongs to the database.
- Consider indexes based on actual query patterns.
- Avoid unnecessary queries.
- Preserve transaction boundaries.

## Code

- Follow existing project conventions.
- Do not introduce abstractions without a concrete reason.
- Prefer small, focused functions.
- Avoid unnecessary dependencies.
