---
name: debugging
description: Use when diagnosing a bug, error, crash, or unexpected behavior. Covers root-cause analysis, inspection order, minimal fixes, and verification. Triggers on stack traces, logs, failing tests, and "why is this happening".
---

# Debugging

## Process

1. Reproduce or inspect the reported problem.
2. Identify the actual failure point.
3. Trace relevant data and control flow.
4. Determine the root cause.
5. Fix the root cause, not the symptom.
6. Verify the fix.

Do not make random changes until the problem disappears.

## Inspect

- Stack traces
- Logs
- Request data
- Database state
- Relevant source code
- Configuration
- Recent changes

## Rules

- Make the smallest fix that addresses the root cause.
- Avoid unrelated refactoring.
- Do not mask symptoms with defensive checks or broad catch blocks.
- Preserve useful error context when diagnosing.

## Explaining an Error

Report in this order:

1. What happened
2. Why it happened
3. What should be checked
4. The smallest appropriate fix

## Verification

- Re-run the failing case.
- Run the most relevant tests.
- Run linting or type checks when available.
- Do not claim something was tested if it was not actually tested. If tests cannot be run, state why.
