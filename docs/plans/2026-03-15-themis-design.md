# Themis — QA Contract Framework

## Overview

Themis is a stack-agnostic QA contract framework that protects pages, features, and components from unintended changes during AI-assisted (vibe) coding. Inspired by data contracts in data engineering, Themis defines what a page or feature IS — visually, behaviorally, and in terms of data — and ensures AI assistants and CI pipelines respect those contracts.

## Core Concepts

### What is a Contract?

A contract is a `.contract.md` file that describes the expected state of a page, feature, component, or API endpoint. It covers three dimensions:

- **Visual**: What it looks like (layout, elements, styling)
- **Behavior**: What it does (user flows, interactions, state changes)
- **Data**: What it sends/receives (API shapes, request/response contracts)

Contracts also include explicit **Rules** using RFC 2119 keywords (MUST, MUST NOT, MAY) that define hard constraints.

### Contract Lifecycle

```
draft → locked → (violation detected) → amend proposed → draft → locked
```

- **draft**: Contract is being shaped, not enforced
- **locked**: Contract is enforced — AI and CI will flag violations
- **deprecated**: Contract is no longer active

## Directory Structure

```
themis-qa/
├── themis.config.md              # Project config (platform, settings)
├── INDEX.md                     # Maps paths → contract files
├── global/                      # App-wide contracts
│   ├── navigation.contract.md
│   └── auth-flow.contract.md
├── pages/                       # Page contracts, organized by URL path
│   ├── auth/
│   │   └── login/
│   │       ├── login.contract.md
│   │       └── screenshots/
│   └── dashboard/
│       ├── dashboard.contract.md
│       └── screenshots/
├── api/                         # API endpoint contracts
│   └── auth/
│       └── login/
│           └── post.contract.md
├── features/                    # Feature-level contracts
│   └── search.contract.md
└── components/                  # Reusable component contracts
    └── data-table.contract.md
```

### Three Scope Layers

| Layer | Scope | Example |
|-------|-------|---------|
| **Global** | Cross-cutting concerns, app-wide invariants | "Navigation always shows 5 menu items" |
| **Page/Feature** | A specific page or user-facing feature | "Login page has email + password fields" |
| **Component** | A reusable UI element | "DataTable always renders headers, supports sorting" |

Contracts cascade: a page contract can reference component contracts, and global contracts apply everywhere.

### INDEX.md

The index acts as a routing table mapping paths to contracts:

```markdown
# Themis Contract Index

## Pages
- `/auth/login` → [pages/auth/login/login.contract.md]
- `/dashboard` → [pages/dashboard/dashboard.contract.md]

## API
- `POST /auth/login` → [api/auth/login/post.contract.md]

## Components
- `DataTable` → [components/data-table.contract.md]
```

## Contract File Format

```markdown
---
name: Login Page
description: User authentication via email and password
type: page
path: /auth/login
status: locked
priority: critical
platform: web
references:
  - components/button.contract.md
  - api/auth/login/post.contract.md
created: 2026-03-15
updated: 2026-03-15
---

## Visual

- Login form centered on page
- Fields: email (text input), password (password input)
- "Sign in" primary button below fields
- "Forgot password?" link below button
- Company logo above the form

### Screenshots
- ![desktop](./screenshots/desktop.png)
- ![mobile](./screenshots/mobile.png)

## Behavior

- Submitting with valid credentials redirects to `/dashboard`
- Submitting with invalid credentials shows inline error message
- Empty fields show validation on blur
- "Forgot password?" navigates to `/auth/reset`
- Form is keyboard-navigable (tab order: email → password → button)

## Data

### Request
- `POST /auth/login`
- Body: `{ email: string, password: string }`

### Response
- 200: `{ token: string, user: { id, email, name } }`
- 401: `{ error: "invalid_credentials" }`

## Rules

- MUST NOT remove or rename form fields
- MUST NOT change the authentication endpoint
- MUST preserve tab navigation order
- MAY update styling if visual layout is maintained
```

### Frontmatter Fields

| Field | Required | Values |
|-------|----------|--------|
| `name` | yes | Human-readable name |
| `description` | yes | One-line summary |
| `type` | yes | `page`, `api`, `component`, `feature`, `global` |
| `path` | yes | URL path, endpoint, or component name |
| `status` | yes | `draft`, `locked`, `deprecated` |
| `priority` | yes | `critical`, `high`, `medium`, `low` |
| `platform` | yes | `web`, `mobile`, `api`, `universal` |
| `references` | no | List of related contract paths |
| `created` | yes | Date created |
| `updated` | yes | Date last modified |

### Body Sections

| Section | Purpose |
|---------|---------|
| **Visual** | What it looks like, with optional screenshot references |
| **Behavior** | What it does — user flows, interactions, state changes |
| **Data** | API shapes, request/response contracts |
| **Rules** | Explicit MUST/MUST NOT/MAY constraints |

## Config File

`themis-qa/themis.config.md`:

```markdown
---
name: Themis QA
version: 1.0
---

## Project

- **Platform**: web | mobile | api | universal
- **Base URL**: http://localhost:3000

## Enforcement

- **Default priority**: high
- **AI guardrails**: enabled
- **CI checks**: enabled

## Conventions

- **Contract format version**: 1
- **Screenshot tool**: playwright | manual
- **Shaping doc path**: docs/briefs/

## AI Context

Instructions for AI assistants working in this codebase:

- Always read `themis-qa/INDEX.md` before making changes
- Cross-reference modified files against contract paths
- Never modify a locked contract without running `/themis-amend`
- When generating new contracts, always update `INDEX.md`
```

## Skills

### /themis-init

Interactive project setup.

1. Asks platform type (web, mobile, api, universal)
2. Asks local dev URL
3. Asks shaping doc path
4. Scaffolds `themis-qa/` directory structure
5. Generates `themis.config.md`
6. Creates empty `INDEX.md`
7. Adds Themis instructions to `CLAUDE.md`

### /themis-generate

Creates a contract from natural language input.

**Accepts:**
- Prompt (natural language description)
- Shaping document (narrative brief, structured pitch)
- Screenshot/mockup (annotated images)
- Any mix of the above

**Flow:**
1. Asks what type of contract (page, api, component, feature, global)
2. Accepts description via prompt, file reference, or screenshot
3. Generates `.contract.md` as draft
4. Updates `INDEX.md`
5. Presents draft for review
6. Locks on approval

### /themis-scan

Auto-generates baseline contracts from existing app/code.

1. Reads current codebase structure
2. Identifies pages, components, API endpoints
3. Generates draft contracts for each
4. Developer reviews, refines, and locks

### /themis-check

Validates current changes against locked contracts.

1. Reads `INDEX.md` to load all locked contracts
2. Cross-references the current diff against relevant contracts
3. For each violation:
   - Identifies the broken clause
   - Shows the contract, clause, and priority
   - Proposes options (revert or amend)

**Priority enforcement:**

| Priority | AI-time | CI |
|----------|---------|-----|
| `critical` | Hard block | Fails build |
| `high` | Hard block | Fails build |
| `medium` | Warning + propose amend | Warning in PR comment |
| `low` | Silent | Info in PR comment |

### /themis-amend

Proposes a contract update after a violation is flagged.

1. Shows current contract clause
2. Shows proposed change
3. Presents diff between current and proposed contract
4. If approved: updates contract to draft with changes
5. Developer reviews and locks

## Enforcement

### Layer 1: AI Guardrails (at coding time)

When the developer is vibe coding, the AI assistant:

1. **On session start** — reads `themis-qa/INDEX.md` to know which contracts exist
2. **Before modifying files** — cross-references the file being changed against relevant contracts
3. **On violation detected** — stops and flags:

```
⚠️ Themis: This change would break a locked contract.

Contract: themis-qa/pages/auth/login/login.contract.md
Clause:   Behavior #2 — "Submitting with invalid credentials shows inline error"
Priority: critical

Your change removes the inline error in favor of a toast notification.

Options:
1. Revert this change and find another approach
2. /themis-amend to propose updating the contract
```

### Layer 2: CI Safety Net

A CI step that validates contracts against the PR diff:

```yaml
# .github/workflows/themis.yml
name: Themis Contract Check
on: [pull_request]
jobs:
  themis:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Themis Contract Check
        run: claude --skill themis-check --diff ${{ github.event.pull_request.base.sha }}
```

## Contract Generation Inputs

Themis accepts multiple input formats for contract generation:

1. **Natural language prompt** — "Login page with email/password, calls POST /auth/login"
2. **Narrative brief** — plain English document describing the page/feature
3. **Structured pitch** — Shape Up style with problem, appetite, solution
4. **Screenshots/mockups** — images with annotations
5. **Mix of any above** — combine narrative + visuals

All inputs are interpreted by the AI and translated into the structured contract format.

## Platform Support

Themis is stack-agnostic. The contract format is identical across platforms. What changes is:

- **Web**: `pages/` organized by URL path, visual contracts reference DOM/CSS
- **Mobile**: `screens/` organized by route, visual contracts reference native components
- **API**: `api/` organized by endpoint and method, no visual section

The platform is set in `themis.config.md` and determines which top-level folders are scaffolded by `/themis-init`.
