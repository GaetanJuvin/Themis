---
name: sibyl-check
description: Use when validating code changes against locked Sibyl QA contracts. Run before completing work, before commits, or in CI. Triggers on "check contracts", "validate contracts", "sibyl check", or when about to complete a task in a project with sibyl-qa/.
---

## Overview

Validates current code changes against all locked contracts. Reads the diff, cross-references against contracts, and flags violations.

## Flow

### Step 1: Load contracts

- Read `sibyl-qa/INDEX.md` to get all contract paths
- Read each referenced contract file
- Filter to `status: locked` contracts only (skip draft/deprecated)

### Step 2: Get the diff

- Run `git diff` to get unstaged changes
- Run `git diff --cached` to get staged changes
- Combine into a full picture of what changed

If called with a specific diff reference (e.g., CI mode with base SHA):
- Run `git diff <base>..HEAD`

### Step 3: Map changes to contracts

For each changed file, determine which contracts might be affected:
- Match file paths to page/screen routes
- Match API handler files to API contracts
- Match component files to component contracts
- Global contracts are always checked

### Step 4: Validate each relevant contract

For each matched contract, check every clause:

**Visual clauses**: Check if the change removes, renames, or restructures UI elements mentioned in the Visual section.

**Behavior clauses**: Check if the change alters user flows, interactions, or state changes described in the Behavior section.

**Data clauses**: Check if the change modifies API shapes, request/response structures, or data types described in the Data section.

**Rules**: Check each MUST/MUST NOT/MAY rule explicitly.

### Step 5: Report results

**No violations found:**
```
✅ Sibyl: All contracts satisfied. X contracts checked, 0 violations.
```

**Violations found — format per violation:**
```
⚠️ Sibyl: Contract violation detected.

Contract: sibyl-qa/pages/auth/login/login.contract.md
Clause:   [Section] #[number] — "[clause text]"
Priority: [critical|high|medium|low]

What changed: [description of the code change]
What breaks: [description of what contract clause is violated]

Options:
1. Revert this change and find another approach
2. /sibyl-amend to propose updating the contract
```

### Step 6: Enforce based on priority

| Priority | Action |
|----------|--------|
| `critical` | STOP. Do not proceed. Require resolution. |
| `high` | STOP. Do not proceed. Require resolution. |
| `medium` | WARN. Show violation. Ask developer to confirm or amend. |
| `low` | INFO. Show violation. Continue unless developer objects. |

### CI Mode

When running in CI (detected by CI environment variable or explicit flag):
- Output violations as structured text
- Exit code 1 if any critical/high violations
- Exit code 0 with warnings for medium/low
- Format suitable for PR comment (markdown)
