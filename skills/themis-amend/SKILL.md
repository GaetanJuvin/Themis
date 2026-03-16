---
name: themis-amend
description: Use when a Themis contract violation was flagged and the developer wants to update the contract to reflect an intentional change. Triggers on "amend contract", "update contract", "themis amend", or after a /themis-check violation.
---

## Overview

Proposes a contract update when a violation represents an intentional change rather than a bug.

## Flow

### Step 1: Identify the contract

If called after a `/themis-check` violation:
- Use the contract and clause from the violation context

If called standalone:
- Ask: "Which contract needs updating?"
- List contracts from `themis-qa/INDEX.md`
- Or accept a direct path

### Step 2: Show current state

Display the current contract clause(s) that need updating.

### Step 3: Propose amendment

Based on the code changes (from git diff), propose updated clause text:

```
📝 Themis Amendment Proposal

Contract: themis-qa/pages/auth/login/login.contract.md

Current clause (Behavior #2):
  "Submitting with invalid credentials shows inline error message"

Proposed clause:
  "Submitting with invalid credentials shows toast notification with error message"

Reason: Code change replaces inline error div with toast component.
```

### Step 4: Developer review

Ask: "Accept this amendment?"

- **Yes** → Apply the change:
  1. Update the clause text in the contract file
  2. Set `status: draft` in frontmatter
  3. Update `updated:` date in frontmatter
  4. Present the full updated contract
  5. Ask: "Lock the updated contract?"
  6. If yes → set `status: locked`

- **No** → Ask what the clause should say instead, apply that version

- **Reject entirely** → Leave contract unchanged, developer should revert code change

### Step 5: Confirm

```
✅ Contract amended and locked.
   themis-qa/pages/auth/login/login.contract.md
   Updated: Behavior #2
```

## Multiple Amendments

If multiple clauses need updating in the same contract, present them all at once for efficiency. But still require explicit approval.

## New Clauses

If the code change introduces new behavior not covered by any clause:
- Propose adding a new clause to the appropriate section
- Flag it clearly as an ADDITION, not a modification
