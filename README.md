<p align="center">
  <img src="https://img.shields.io/badge/version-0.1.0-blue" alt="Version">
  <img src="https://img.shields.io/badge/platform-Claude%20Code-blueviolet" alt="Platform">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="License">
</p>

<h1 align="center">Themis</h1>

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

Themis introduces **QA contracts** — human-readable documents that define what a page, feature, or API endpoint **is** and **does**. AI assistants read these contracts before making changes and stop when something would break.

```
You: "Add dark mode to the login page"

AI: ⚠️ Themis: This change would break a locked contract.
    Contract: themis-qa/pages/auth/login/login.contract.md
    Clause: Visual #3 — "Sign in primary button below fields"
    Your change removes the primary button styling.

    1. Find another approach
    2. /themis-amend to update the contract
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
                                      /themis-generate
                                             │
                                             ▼
┌──────────────────────────────────────────────────────────────────┐
│                        themis-qa/                                  │
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
   │  AI reads contracts │                     │  /themis-check runs   │
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
  │  draft   │──────────────▶│  locked  │───────────────▶│  /themis-amend│
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

Themis is a [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin. Everything runs as skills — no separate CLI, no build step.

### `/themis-init`

Interactive setup. Asks your platform, scaffolds the `themis-qa/` directory, generates config, and wires Themis into your `CLAUDE.md`.

```
> /themis-init

Themis: What platform(s) does this project target?
  1. Web  2. Mobile  3. API  4. Universal

You: 1

Themis: What's your local dev URL?
You: http://localhost:3000

Themis: ✓ Created themis-qa/ with config, INDEX.md, and directory structure.
       Use /themis-generate to create your first contract.
```

### `/themis-generate`

Create contracts from anything — a prompt, a shaping document, a screenshot, or a mix.

```
> /themis-generate

Themis: What are we contracting? (page, api, component, feature, global)
You: page

Themis: Describe it, paste a brief, or reference a file/screenshot.
You: Login page with email and password fields. Submits to POST /auth/login.
     Redirects to /dashboard on success. Shows inline error on failure.

Themis: ✓ Generated themis-qa/pages/auth/login/login.contract.md (draft)
       Lock it?
You: yes
```

### `/themis-check`

Validate your changes against all locked contracts. Run it before committing, or let CI run it on every PR.

```
> /themis-check

✅ Themis: All contracts satisfied. 12 contracts checked, 0 violations.
```

```
> /themis-check

⚠️ Themis: Contract violation detected.

Contract: themis-qa/pages/auth/login/login.contract.md
Clause:   Behavior #2 — "Submitting with invalid credentials shows inline error"
Priority: critical

What changed: Replaced inline error div with toast notification.
What breaks: Contract specifies inline error, not toast.

Options:
1. Revert this change and find another approach
2. /themis-amend to propose updating the contract
```

### `/themis-amend`

When a violation is intentional, amend the contract instead of reverting the code.

```
> /themis-amend

📝 Themis Amendment Proposal

Contract: themis-qa/pages/auth/login/login.contract.md

Current clause (Behavior #2):
  "Submitting with invalid credentials shows inline error message"

Proposed clause:
  "Submitting with invalid credentials shows toast notification with error"

Accept this amendment? yes

✅ Contract amended and locked.
```

### `/themis-scan`

Already have a codebase? Scan it to auto-generate baseline contracts.

```
> /themis-scan

📋 Themis Scan Results

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

Add Themis to your GitHub Actions:

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

---

## Platform Support

Themis is stack-agnostic. The contract format works across platforms — only the directory structure adapts.

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
/plugin install themis
```

Then in any project:

```bash
/themis-init
```

---

## Project Structure

```
Themis/
├── .claude-plugin/          # Claude Code plugin config
│   ├── plugin.json
│   └── hooks.json
├── skills/                  # Themis skills (the AI logic)
│   ├── themis-init/
│   ├── themis-generate/
│   ├── themis-check/
│   ├── themis-amend/
│   └── themis-scan/
├── commands/                # User-facing slash commands
│   ├── themis-init.md
│   ├── themis-generate.md
│   ├── themis-check.md
│   ├── themis-amend.md
│   └── themis-scan.md
├── templates/               # Templates used by /themis-init
│   ├── themis.config.md
│   ├── INDEX.md
│   └── contract.template.md
└── docs/plans/              # Design docs
```

---

## Philosophy

**Contracts, not tests.** Tests verify implementation. Contracts define intent. When you vibe code, the implementation changes constantly — but the intent shouldn't change without you knowing.

**Human-readable first.** Contracts are Markdown. Read them on GitHub, in your editor, in a PR review. No special tooling needed to understand what a page should do.

**Defense in depth.** AI reads contracts before coding (guardrail). CI checks contracts after coding (safety net). Two layers, one source of truth.

**Amend, don't fight.** When you intentionally change something, Themis doesn't block you — it asks you to update the contract. The contract evolves with your app.

---

<p align="center">
  <em>Themis — divine law for your codebase.</em>
</p>
