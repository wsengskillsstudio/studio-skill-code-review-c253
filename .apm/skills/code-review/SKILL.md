---
name: code-review
description: Analyzes code changes (diffs, PRs, or files) for logic issues, security gaps, performance problems, and style concerns. Provides structured, actionable feedback with severity rankings.
---

# Code Review

This skill performs systematic code review on diffs, pull requests, commits, or individual files. It checks for logic errors, security vulnerabilities, performance bottlenecks, style inconsistencies, and test coverage gaps, then produces a structured report with severity-ranked findings and constructive suggestions.

## When this skill triggers

- when the user asks to review a pull request
- when the user mentions reviewing code changes or diffs
- when the user explicitly says 'code review' or 'CR'
- when reviewing commits or branches

## Inputs

- **target** — Path to file(s), PR number, commit SHA, branch name, or diff text to review.
- **scope** — Optional focus area: 'security', 'performance', 'style', 'tests', or 'all' (default).
- **context** — Optional project context like language, framework, or specific patterns to check.

## Outputs

- **review_report** — Structured markdown report with summary, severity-ranked issues, suggestions, and positive highlights.

## How to perform this task

### 1. Extract the code to review

- If `target` is a file path, read the file contents directly.
- If `target` looks like a PR number (e.g., `#123`, `PR123`), use git or GitHub CLI to fetch the diff: `gh pr diff 123` or equivalent.
- If `target` is a commit SHA, run `git show <SHA>` to get the diff.
- If `target` is a branch name, compare against the base branch: `git diff main...<branch>`.
- If `target` is inline diff text (starts with `diff --git` or similar), use it as-is.
- If the diff is very large (>2000 lines), inform the user and offer to review in chunks or focus on specific files.

### 2. Identify language and framework context

- Examine file extensions, imports, and package manifests to determine the primary language(s) and framework(s).
- If the user provided `context`, prioritize that information (e.g., "Django project", "React with TypeScript").
- Note any project-specific patterns mentioned in context (e.g., "check for proper error handling per our standards").

### 3. Run systematic checks across five dimensions

For each dimension, look for common issues and best-practice violations. Adjust depth based on `scope` input:

#### Logic and correctness
- Off-by-one errors, incorrect loop bounds, wrong operators.
- Null/undefined handling, missing edge case checks.
- Race conditions, concurrency issues, improper locking.
- Dead code, unreachable branches, contradictory conditions.
- Incorrect algorithm implementation or data structure misuse.

#### Security
- Injection vulnerabilities (SQL, command, XSS, path traversal).
- Authentication/authorization bypasses or weak checks.
- Hardcoded secrets, credentials in code or comments.
- Insecure deserialization, unsafe reflection.
- Missing input validation, improper output encoding.
- Use of deprecated/vulnerable libraries or functions.

#### Performance
- Inefficient algorithms (O(n²) where O(n log n) is feasible).
- Unnecessary database queries, N+1 query patterns.
- Missing indexes, full table scans.
- Memory leaks, excessive allocations, unbounded buffers.
- Blocking I/O in hot paths, missing async/await.
- Repeated computation that could be cached or memoized.

#### Style and maintainability
- Inconsistent naming conventions, unclear variable names.
- Overly complex functions (>50 lines, deep nesting).
- Missing documentation for non-obvious logic.
- Violation of language idioms (e.g., non-Pythonic code in Python).
- Poor error messages, swallowed exceptions.
- Code duplication that should be factored out.

#### Testing
- New functionality without corresponding tests.
- Edge cases not covered by existing tests.
- Tests that are too brittle or tightly coupled to implementation.
- Missing negative test cases, error path validation.
- Inadequate assertions, tests that always pass.

### 4. Rank findings by severity

Assign each issue one of four levels:

- **Critical**: Security vulnerabilities, data loss risks, crashes, or logic errors causing incorrect behavior.
- **High**: Performance problems impacting user experience, significant maintainability issues, missing critical tests.
- **Medium**: Code smells, minor performance inefficiencies, style inconsistencies, incomplete documentation.
- **Low**: Nitpicks, optional refactorings, subjective improvements.

### 5. Identify positive highlights

Note 2-3 things done well in the code:
- Good error handling, clear variable names, elegant solutions.
- Proper test coverage, thoughtful edge case handling.
- Effective use of language features, good performance considerations.

This balances the critique and makes the review more constructive.

### 6. Format the review report

Structure the output as markdown with these sections:

```markdown
# Code Review: [target description]

## Summary
[1-2 sentences: overall assessment, number of issues found, primary concerns.]

## Critical Issues
[List critical findings with file:line, description, and fix suggestion.]

## High Priority Issues
[Same format.]

## Medium Priority Issues
[Same format.]

## Low Priority Issues
[Same format.]

## Positive Highlights
- [Well-executed aspect 1]
- [Well-executed aspect 2]
- [Well-executed aspect 3]

## Recommendations
[2-3 actionable next steps, prioritized by impact.]
```

For each issue, use this format:
```
**file.py:42** — [Brief description of the problem]
- **Concern**: [Why this matters]
- **Suggestion**: [Specific fix or improvement]
```

### 7. Handle edge cases

- If no issues are found, still produce a report with the summary and positive highlights.
- If the code is in an unfamiliar language, note this in the summary and focus on universal principles (logic, security concepts).
- If focusing on a specific scope (e.g., only security), mention that other dimensions were not reviewed.
- If changes are trivial (whitespace, formatting only), acknowledge this briefly and keep the report short.

### 8. Deliver the report

- Present the full markdown report to the user.
- Offer to elaborate on any finding or provide example code for suggested fixes.
- If the diff was chunked, offer to continue reviewing subsequent chunks.

## Failure modes

- Target code is unreachable (invalid path, private repository without access)
- Diff too large to analyze in a single pass (requires chunking)
- Language or framework unknown, limiting review depth
- No changes detected in the provided diff or commit

## Dependencies

### CLI
- git