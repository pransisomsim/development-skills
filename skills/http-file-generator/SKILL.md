---
name: http-file-generator
description: Generate runnable .http API testing files by tracing backend routes, validators, controllers, services, and Swagger/OpenAPI documentation. Use when asked to generate HTTP requests for a route file, module, or API endpoint.
---

# HTTP File Generator

## Objective

Generate `.http` files for testing existing backend API endpoints.

Analyze the actual source code before generating requests. Produce test requests with accurate URLs, headers, parameters, request bodies, and authentication requirements.

Do not modify application source code.

## Workflow

### 1. Discover endpoints

Read the requested route file.

Identify every endpoint, including:
- HTTP method and route path
- Router prefixes and mounted paths
- Path parameters and query parameters
- Authentication and role middleware
- Validation middleware
- Content type requirements

Resolve the full URL path by tracing router mounts and prefixes when possible.

Do not assume a route's local path is its complete API path.

### 2. Trace request requirements

For each endpoint, inspect relevant files in this order:

1. Route definitions and middleware.
2. Request validators and schemas (Zod, Joi, express-validator, or equivalent).
3. Controllers and referenced services.
4. Swagger/OpenAPI specifications and documented examples.
5. Existing `.http` files and environment conventions.

Follow imports and references as needed. Do not read the entire repository unnecessarily.

Determine:
- Required and optional body fields
- Field types, formats, enums, limits, and nested structures
- Query and path parameters
- Authentication tokens and role requirements
- Required headers and content types
- Business rules that affect valid requests
- Dependencies on existing database records or prior operations

Do not infer accepted fields from naming conventions alone.

### 3. Resolve uncertainty

Use this source-of-truth order for runtime input requirements:

1. Actual validation and request-handling behavior.
2. Controller and service business rules.
3. Swagger/OpenAPI documentation.
4. Existing test examples.
5. Reasonable assumptions, clearly identified.

If documentation conflicts with runtime validation, follow the runtime behavior and flag the discrepancy in a short note.

If a required value cannot be determined, use a clearly marked placeholder instead of inventing a supposedly valid value.

Never invent authentication tokens, database IDs, enum members, response fields, or undocumented endpoints.

### 4. Generate the HTTP file

Create one `.http` file for the requested route file or module.

Follow existing repository conventions if available. Otherwise:

- Use `# @kulala-vscode-restclient-compat` as the first line of the `.http` file to ensure broad client support (Kulala in Neovim, VS Code REST Client, JetBrains HTTP Client).
- **Declare all global file variables at the very top**: Place all `@variable = value` definitions before the first request separator (`###`). Never place file-level variables after a `###` separator, as clients like Kulala scope variables following a delimiter locally to that request block.
- **Strict delimiter rules**: `###` is exclusively a request separator.
  - NEVER use `###` for section headers, comments, or decorative banners (e.g., do NOT use `### ===== SECTION =====`).
  - Use `#` or `//` for comments and section headers (e.g., `# ===== SECTION =====`).
  - Name requests directly on the delimiter (e.g., `### loginCustomer`) and/or with `# @name loginCustomer`.
- Use standard HTTP syntax.
- Use `Content-Type: application/json` for JSON bodies.
- Include `Authorization: Bearer {{token}}` when authentication is required.
- Use environment variables for the base URL and reusable credentials.
- Keep or create an `http-client.env.json` at the project root with `default` and environment profiles when supporting clients like Kulala and JetBrains.
- Include path and query parameters in their correct locations.
- Include request bodies only when appropriate for the endpoint.
- Use valid JSON and schema-compliant example values.
- Include optional fields when useful, without adding unnecessary noise.
- Reuse response-derived variables only when supported by the chosen HTTP client.
- Group requests logically by endpoint or operation.

Prefer realistic, valid example requests over arbitrary placeholder data.

### 5. Handle dependencies

When an endpoint requires existing records or authentication:

- Identify how the necessary IDs and tokens can be obtained.
- Reuse existing environment variables or supported response references when available.
- If the request depends on another operation, document that dependency.
- Do not assume database seed data exists unless the repository confirms it.
- Do not execute destructive requests or modify remote data merely to generate the file.

Include useful CRUD operations and workflow requests supported by the actual routes.

### 6. Request coverage

Generate at least one valid example request per discovered endpoint.

Where useful and supported by the schema, add additional requests for:
- Missing required fields
- Invalid field types or enum values
- Unauthorized access
- Invalid path or query parameters
- Relevant business-rule failures

Do not generate excessive negative tests when the user requests only basic endpoint coverage.

Do not assume every failure returns a particular status code. Verify the implementation or documentation first.

### 7. Output and verification

Save the result to the requested location. If no location is specified, use the repository's existing HTTP-test convention or create a clearly named file near the relevant module's test files.

Before finishing:
- Check that every generated request corresponds to a real route.
- Check HTTP methods and full paths.
- Validate JSON syntax and field types.
- Check required headers and authentication.
- Check that all global `@variables` appear before the first `###` delimiter.
- Check that `###` is never used for section comments or decorative banners (use `#`).
- Check that environment variables are defined in the file and/or `http-client.env.json`.
- Identify unresolved placeholders and external dependencies.
- Do not claim requests were executed or passed unless they were actually tested.

Report:
1. The generated file path.
2. The number of endpoints covered.
3. Any assumptions, placeholders, or missing documentation.
4. Any conflicts between Swagger and runtime validation.

Keep the final report concise.
