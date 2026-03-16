<p align="center">
  <img src="https://img.shields.io/badge/version-0.1.0-blue" alt="Version">
  <img src="https://img.shields.io/badge/platform-Claude%20Code-blueviolet" alt="Platform">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="License">
</p>

<h1 align="center">Sibyl</h1>

<p align="center">
  <strong>QA contracts for the vibe coding era.</strong><br>
  Protect your pages, features, and APIs from unintended changes — even when AI is writing the code.
</p>

<p align="center">
  <em>Inspired by data contracts in data engineering.</em>
</p>

---

## The Problem

When vibe coding with AI assistants, things break silently. The AI adds a feature and accidentally removes a form field. It refactors a component and changes the API shape. It "improves" a page and breaks the user flow.

**You don't notice until production.**

## The Solution

Sibyl introduces **QA contracts** — human-readable documents that define what a page, feature, or API endpoint **is** and **does**. AI assistants read these contracts before making changes and stop when something would break.

```
You: "Add dark mode to the login page"

AI: ⚠️ Sibyl: This change would break a locked contract.
    Contract: sibyl-qa/pages/auth/login/login.contract.md
    Clause: Visual #3 — "Sign in primary button below fields"
    Your change removes the primary button styling.

    1. Find another approach
    2. /sibyl-amend to update the contract
```

---

## How It Works

```
                                    ┌─────────────────────┐
                                    │   Shaping Inputs     │
                                    │                     │
                                    │  - Prompts          │
                                    │  - Briefs           │
                                    │  - Screenshots      │
                                    │  - Narratives       │
                                    └────────┬────────────┘
                                             │
                                      /sibyl-generate
                                             │
                                             ▼
┌──────────────────────────────────────────────────────────────────┐
│                        sibyl-qa/                                  │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │ INDEX.md      │  │ config.md    │  │ global/              │   │
│  │ (routing      │  │ (platform,   │  │   auth-flow          │   │
│  │  table)       │  │  settings)   │  │   navigation         │   │
│  └──────────────┘  └──────────────┘  └──────────────────────┘   │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │ pages/        │  │ api/         │  │ components/          │   │
│  │   auth/       │  │   auth/      │  │   data-table         │   │
│  │     login/    │  │     login/   │  │   modal              │   │
│  │   dashboard/  │  │     post.md  │  │                      │   │
│  └──────────────┘  └──────────────┘  └──────────────────────┘   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
                         │                         │
              ┌──────────┘                         └──────────┐
              │                                               │
              ▼                                               ▼
   ┌─────────────────────┐                     ┌──────────────────────┐
   │  Layer 1: AI Guard   │                     │  Layer 2: CI Safety   │
   │                     │                     │                      │
   │  AI reads contracts │                     │  /sibyl-check runs   │
   │  before changes.    │                     │  against PR diff.    │
   │  Stops on violation.│                     │  Fails build on      │
   │                     │                     │  critical violations.│
   └─────────────────────┘                     └──────────────────────┘
```

---

## Contract Format

Contracts are `.contract.md` files — Markdown with YAML frontmatter. Human-readable, git-friendly, AI-native.

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
  - api/auth/login/post.contract.md
created: 2026-03-15
updated: 2026-03-15
---

## Visual

- Login form centered on page
- Fields: email (text input), password (password input)
- "Sign in" primary button below fields
- "Forgot password?" link below button

## Behavior

- Submitting with valid credentials redirects to `/dashboard`
- Submitting with invalid credentials shows inline error
- Empty fields show validation on blur

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

### Three Dimensions

| Section | What it captures | Example |
|---------|-----------------|---------|
| **Visual** | What it looks like | "Login form centered, two fields, one button" |
| **Behavior** | What it does | "Invalid credentials show inline error" |
| **Data** | What it sends/receives | "POST /auth/login with email + password" |
| **Rules** | Hard constraints (RFC 2119) | "MUST NOT remove form fields" |

---

## Contract Lifecycle

```
  ┌─────────┐     lock      ┌─────────┐    violation    ┌─────────────┐
  │  draft   │──────────────▶│  locked  │───────────────▶│  /sibyl-amend│
  └─────────┘               └─────────┘                └──────┬──────┘
       ▲                                                       │
       │                      approve                          │
       └───────────────────────────────────────────────────────┘
```

- **draft** — Contract is being shaped. Not enforced.
- **locked** — Contract is active. AI and CI enforce it.
- **deprecated** — Contract is retired. Skipped during checks.

---

## Three Scope Layers

Contracts are organized by scope, similar to how data contracts work in data engineering:

```
┌──────────────────────────────────────────────────────────────┐
│                        GLOBAL                                 │
│  Cross-cutting: auth flows, navigation, error handling        │
│  Applied everywhere. Always checked.                          │
├──────────────────────────────────────────────────────────────┤
│                     PAGE / FEATURE                            │
│  Specific pages or user-facing features                       │
│  Organized by URL path (web) or route (mobile)                │
├──────────────────────────────────────────────────────────────┤
│                      COMPONENT                                │
│  Reusable UI elements with their own contracts                │
│  Referenced by page contracts                                 │
└──────────────────────────────────────────────────────────────┘
```

---

## Skills

Sibyl is a [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin. Everything runs as skills — no separate CLI, no build step.

### `/sibyl-init`

Interactive setup. Asks your platform, scaffolds the `sibyl-qa/` directory, generates config, and wires Sibyl into your `CLAUDE.md`.

```
> /sibyl-init

Sibyl: What platform(s) does this project target?
  1. Web  2. Mobile  3. API  4. Universal

You: 1

Sibyl: What's your local dev URL?
You: http://localhost:3000

Sibyl: ✓ Created sibyl-qa/ with config, INDEX.md, and directory structure.
       Use /sibyl-generate to create your first contract.
```

### `/sibyl-generate`

Create contracts from anything — a prompt, a shaping document, a screenshot, or a mix.

```
> /sibyl-generate

Sibyl: What are we contracting? (page, api, component, feature, global)
You: page

Sibyl: Describe it, paste a brief, or reference a file/screenshot.
You: Login page with email and password fields. Submits to POST /auth/login.
     Redirects to /dashboard on success. Shows inline error on failure.

Sibyl: ✓ Generated sibyl-qa/pages/auth/login/login.contract.md (draft)
       Lock it?
You: yes
```

### `/sibyl-check`

Validate your changes against all locked contracts. Run it before committing, or let CI run it on every PR.

```
> /sibyl-check

✅ Sibyl: All contracts satisfied. 12 contracts checked, 0 violations.
```

```
> /sibyl-check

⚠️ Sibyl: Contract violation detected.

Contract: sibyl-qa/pages/auth/login/login.contract.md
Clause:   Behavior #2 — "Submitting with invalid credentials shows inline error"
Priority: critical

What changed: Replaced inline error div with toast notification.
What breaks: Contract specifies inline error, not toast.

Options:
1. Revert this change and find another approach
2. /sibyl-amend to propose updating the contract
```

### `/sibyl-amend`

When a violation is intentional, amend the contract instead of reverting the code.

```
> /sibyl-amend

📝 Sibyl Amendment Proposal

Contract: sibyl-qa/pages/auth/login/login.contract.md

Current clause (Behavior #2):
  "Submitting with invalid credentials shows inline error message"

Proposed clause:
  "Submitting with invalid credentials shows toast notification with error"

Accept this amendment? yes

✅ Contract amended and locked.
```

### `/sibyl-scan`

Already have a codebase? Scan it to auto-generate baseline contracts.

```
> /sibyl-scan

📋 Sibyl Scan Results

Found:
  Pages: 12     API endpoints: 8     Components: 23

Recommended contracts (high-value targets):
  ✓ /auth/login — page with form + API call
  ✓ /dashboard — main page, complex layout
  ✓ POST /api/auth/login — auth endpoint
  ✓ DataTable — reused in 5 pages

Generate contracts for all, or select specific ones?
```

---

## Enforcement

### Priority Levels

| Priority | AI-time | CI |
|----------|---------|-----|
| `critical` | Hard block | Fails build |
| `high` | Hard block | Fails build |
| `medium` | Warning + propose amend | Warning in PR comment |
| `low` | Silent | Info in PR comment |

### CI Integration

Add Sibyl to your GitHub Actions:

```yaml
# .github/workflows/sibyl.yml
name: Sibyl Contract Check
on: [pull_request]
jobs:
  sibyl:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Sibyl Contract Check
        run: claude --skill sibyl-check --diff ${{ github.event.pull_request.base.sha }}
```

---

## Platform Support

Sibyl is stack-agnostic. The contract format works across platforms — only the directory structure adapts.

| Platform | Structure | Visual | Behavior | Data |
|----------|-----------|--------|----------|------|
| **Web** | `pages/` by URL path | DOM, layout, CSS | User flows, navigation | API calls, forms |
| **Mobile** | `screens/` by route | Native components, gestures | Screen flows, transitions | API calls, state |
| **API** | `api/` by endpoint | N/A | Endpoint behavior | Request/response shapes |
| **Universal** | All of the above | Per-platform | Per-platform | Shared |

---

## Installation

```bash
# In Claude Code
/plugin install sibyl
```

Then in any project:

```bash
/sibyl-init
```

---

## Project Structure

```
Sibyl/
├── .claude-plugin/          # Claude Code plugin config
│   ├── plugin.json
│   └── hooks.json
├── skills/                  # Sibyl skills (the AI logic)
│   ├── sibyl-init/
│   ├── sibyl-generate/
│   ├── sibyl-check/
│   ├── sibyl-amend/
│   └── sibyl-scan/
├── commands/                # User-facing slash commands
│   ├── sibyl-init.md
│   ├── sibyl-generate.md
│   ├── sibyl-check.md
│   ├── sibyl-amend.md
│   └── sibyl-scan.md
├── templates/               # Templates used by /sibyl-init
│   ├── sibyl.config.md
│   ├── INDEX.md
│   └── contract.template.md
└── docs/plans/              # Design docs
```

---

## Philosophy

**Contracts, not tests.** Tests verify implementation. Contracts define intent. When you vibe code, the implementation changes constantly — but the intent shouldn't change without you knowing.

**Human-readable first.** Contracts are Markdown. Read them on GitHub, in your editor, in a PR review. No special tooling needed to understand what a page should do.

**Defense in depth.** AI reads contracts before coding (guardrail). CI checks contracts after coding (safety net). Two layers, one source of truth.

**Amend, don't fight.** When you intentionally change something, Sibyl doesn't block you — it asks you to update the contract. The contract evolves with your app.

---

<p align="center">
  <em>Sibyl — the oracle that knows what shouldn't change.</em>
</p>
