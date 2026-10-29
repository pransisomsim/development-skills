# Development Skills

A collection of reusable AI agent skills for software development workflows. Each skill lives in its own directory under [`skills/`](./skills) and is defined by a `SKILL.md` file with metadata and instructions.

## Available skills

| Skill | Description |
| --- | --- |
| [`backend-standard`](./skills/backend-standard/SKILL.md) | Backend architecture, API, database, code, and dependency standards. |
| [`caveman`](./skills/caveman/SKILL.md) | Terse communication while preserving technical substance and exact errors. |
| [`coding-standard`](./skills/coding-standard/SKILL.md) | Code style, architecture, API, database, error-handling, and security guidance. |
| [`debugging`](./skills/debugging/SKILL.md) | Root-cause analysis and verification for bugs, errors, crashes, and failing tests. |
| [`explaining`](./skills/explaining/SKILL.md) | Teaching and walkthrough guidance for code, concepts, and architecture. |
| [`frontend-contract`](./skills/frontend-contract/SKILL.md) | Generates frontend-facing integration contracts from backend workflows. |
| [`frontend-integration`](./skills/frontend-integration/SKILL.md) | Generates structured frontend integration guides for completed backend features. |
| [`github-issue-generator`](./skills/github-issue-generator/SKILL.md) | Generates professional GitHub Issues from project context. |
| [`http-file-generator`](./skills/http-file-generator/SKILL.md) | Generates runnable `.http` API test files from backend routes and documentation. |
| [`planning`](./skills/planning/SKILL.md) | Produces implementation plans after inspecting the existing architecture and behavior. |
| [`refactoring`](./skills/refactoring/SKILL.md) | Guides behavior-preserving restructuring, impact analysis, and verification. |
| [`simulate-html`](./skills/simulate-html/SKILL.md) | Generates single-file HTML simulations for manually testing backend features. |

## Usage

Browse the skill that matches your task and follow its `SKILL.md` instructions. Skills can be used independently or combined when a workflow requires multiple concerns, such as planning, implementation standards, debugging, and refactoring.

## Structure

```text
skills/
└── <skill-name>/
    └── SKILL.md
```

## Contributing

Keep skills focused, implementation-oriented, and consistent with the existing Markdown frontmatter and directory structure. Update this README when adding or removing a skill.