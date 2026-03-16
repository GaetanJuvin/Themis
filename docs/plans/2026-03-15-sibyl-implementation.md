# Sibyl Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build Sibyl as a Claude Code plugin that provides QA contract skills for protecting pages/features/components during vibe coding.

**Architecture:** Sibyl is a Claude Code plugin with 5 skills and 5 matching commands. Skills contain the AI logic (contract generation, validation, amendment). Commands are user-facing `/slash-commands` that invoke skills. Contract files are `.contract.md` (Markdown with YAML frontmatter). Everything lives in a `sibyl-qa/` directory within the target project.

**Tech Stack:** Claude Code plugin system (SKILL.md + commands), Markdown with YAML frontmatter, GitHub Actions for CI.

---

## PR 1: Foundation

### Task 1: Design Doc (already complete)

The design doc exists at `docs/plans/2026-03-15-sibyl-design.md`.

**Step 1: Commit the design doc**

```bash
git add docs/plans/2026-03-15-sibyl-design.md
git commit -m "docs: add Sibyl QA contract framework design doc"
```

---

### Task 2: Plugin Scaffolding

Create the Claude Code plugin structure so Sibyl can be installed as a plugin.

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `.claude-plugin/hooks.json`

**Step 1: Create plugin.json**

Create `.claude-plugin/plugin.json`:

```json
{
  "name": "sibyl",
  "version": "0.1.0",
  "description": "QA contract framework — protects pages, features, and components from unintended changes during vibe coding",
  "author": "GaetanJuvin",
  "repository": "https://github.com/GaetanJuvin/Sibyl"
}
```

**Step 2: Create hooks.json**

Create `.claude-plugin/hooks.json`:

```json
{
  "hooks": [
    {
      "event": "on_session_start",
      "script": "echo 'OK'"
    }
  ]
}
```

**Step 3: Commit**

```bash
git add .claude-plugin/
git commit -m "feat: scaffold Claude Code plugin structure"
```

---

### Task 3: Contract Templates

Create template files that `/sibyl-init` will use when scaffolding a project.

**Files:**
- Create: `templates/sibyl.config.md`
- Create: `templates/INDEX.md`
- Create: `templates/contract.template.md`

**Step 1: Create sibyl.config.md template**

Create `templates/sibyl.config.md`:

```markdown
---
name: Sibyl QA
version: 1.0
---

## Project

- **Platform**: {{platform}}
- **Base URL**: {{base_url}}

## Enforcement

- **Default priority**: high
- **AI guardrails**: enabled
- **CI checks**: enabled

## Conventions

- **Contract format version**: 1
- **Screenshot tool**: manual
- **Shaping doc path**: {{shaping_doc_path}}

## AI Context

Instructions for AI assistants working in this codebase:

- Always read `sibyl-qa/INDEX.md` before making changes
- Cross-reference modified files against contract paths
- Never modify a locked contract without running `/sibyl-amend`
- When generating new contracts, always update `INDEX.md`
```

**Step 2: Create INDEX.md template**

Create `templates/INDEX.md`:

```markdown
# Sibyl Contract Index

<!-- This file maps paths to contract files. Updated automatically by /sibyl-generate. -->

## Global

## Pages

## API

## Features

## Components
```

**Step 3: Create contract.template.md**

Create `templates/contract.template.md`:

```markdown
---
name: {{name}}
description: {{description}}
type: {{type}}
path: {{path}}
status: draft
priority: {{priority}}
platform: {{platform}}
references: []
created: {{date}}
updated: {{date}}
---

## Visual

{{visual_description}}

## Behavior

{{behavior_description}}

## Data

{{data_description}}

## Rules

{{rules}}
```

**Step 4: Commit**

```bash
git add templates/
git commit -m "feat: add contract and config templates"
```

---

### Task 4: Project CLAUDE.md

Create the project CLAUDE.md with development instructions for contributors.

**Files:**
- Create: `CLAUDE.md`

**Step 1: Create CLAUDE.md**

Create `CLAUDE.md`:

```markdown
# Sibyl

QA contract framework for protecting pages, features, and components during vibe coding.

## Project Structure

- `.claude-plugin/` — Claude Code plugin config (plugin.json, hooks.json)
- `skills/` — Sibyl skills (SKILL.md files)
- `commands/` — User-facing slash commands
- `templates/` — Template files used by /sibyl-init
- `docs/plans/` — Design docs and implementation plans

## Skills

Each skill lives in `skills/<skill-name>/SKILL.md`. Skills are the AI logic.

| Skill | Command | Purpose |
|-------|---------|---------|
| `sibyl-init` | `/sibyl-init` | Interactive project setup |
| `sibyl-generate` | `/sibyl-generate` | Create contracts from prompts/docs/screenshots |
| `sibyl-check` | `/sibyl-check` | Validate changes against locked contracts |
| `sibyl-amend` | `/sibyl-amend` | Propose contract updates after violations |
| `sibyl-scan` | `/sibyl-scan` | Auto-generate baseline contracts from existing code |

## Contract Format

Contracts are `.contract.md` files with YAML frontmatter + Visual/Behavior/Data/Rules sections. See `docs/plans/2026-03-15-sibyl-design.md` for full spec.

## Development

- Follow the design doc at `docs/plans/2026-03-15-sibyl-design.md`
- Skills must have `name` and `description` in frontmatter only (max 1024 chars total)
- Skill descriptions start with "Use when..."
- Commands reference skills with "Invoke the sibyl:<skill-name> skill"
```

**Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: add project CLAUDE.md"
```

---

### Task 5: Push PR 1

**Step 1: Create branch and push**

```bash
git checkout -b main
git push -u origin main
```

---

## PR 2: Skills

Create a feature branch for skills work.

```bash
git checkout -b feat/skills
```

---

### Task 6: /sibyl-init skill + command

The setup skill that scaffolds `sibyl-qa/` in a target project.

**Files:**
- Create: `skills/sibyl-init/SKILL.md`
- Create: `commands/sibyl-init.md`

**Step 1: Create the skill**

Create `skills/sibyl-init/SKILL.md`:

```markdown
---
name: sibyl-init
description: Use when setting up Sibyl QA contracts in a project for the first time. Scaffolds sibyl-qa/ directory, config, and INDEX.md. Triggers on "set up sibyl", "initialize contracts", "add QA contracts".
---

## Overview

Interactive setup that scaffolds the `sibyl-qa/` directory structure in the current project.

## Flow

1. **Ask platform**: "What platform(s) does this project target?"
   - Options: web, mobile, api, universal

2. **Ask base URL**: "What's your local dev URL?" (skip if api-only)

3. **Ask shaping doc path**: "Where do you keep shaping docs / briefs? (leave empty to skip)"

4. **Scaffold directory structure based on platform:**

   **Web:**
   ```
   sibyl-qa/
   ├── sibyl.config.md
   ├── INDEX.md
   ├── global/
   ├── pages/
   ├── features/
   └── components/
   ```

   **Mobile:**
   ```
   sibyl-qa/
   ├── sibyl.config.md
   ├── INDEX.md
   ├── global/
   ├── screens/
   ├── features/
   └── components/
   ```

   **API:**
   ```
   sibyl-qa/
   ├── sibyl.config.md
   ├── INDEX.md
   ├── global/
   ├── api/
   └── features/
   ```

   **Universal:**
   ```
   sibyl-qa/
   ├── sibyl.config.md
   ├── INDEX.md
   ├── global/
   ├── pages/
   ├── screens/
   ├── api/
   ├── features/
   └── components/
   ```

5. **Generate `sibyl.config.md`** from template with user's answers

6. **Generate empty `INDEX.md`** from template

7. **Add Sibyl instructions to project's CLAUDE.md:**
   Append this block:
   ```markdown
   ## Sibyl QA Contracts

   This project uses Sibyl QA contracts. Before making changes:

   - Read `sibyl-qa/INDEX.md` to know which contracts exist
   - Cross-reference modified files against contract paths
   - Never modify a locked contract without running `/sibyl-amend`
   - If your change would break a locked contract, stop and flag it
   - Use `/sibyl-generate` to create new contracts
   - Use `/sibyl-check` to validate changes against contracts
   ```

8. **Confirm**: "Sibyl is set up. Use `/sibyl-generate` to create your first contract."

## Important

- If `sibyl-qa/` already exists, warn and ask before overwriting
- Create directories using mkdir, not by writing placeholder files
- Use today's date in generated config
```

**Step 2: Create the command**

Create `commands/sibyl-init.md`:

```markdown
---
disable-model-invocation: true
---

Invoke the sibyl:sibyl-init skill to set up Sibyl QA contracts in this project.
```

**Step 3: Commit**

```bash
git add skills/sibyl-init/ commands/sibyl-init.md
git commit -m "feat: add /sibyl-init skill and command"
```

---

### Task 7: /sibyl-generate skill + command

The core contract creation skill.

**Files:**
- Create: `skills/sibyl-generate/SKILL.md`
- Create: `commands/sibyl-generate.md`

**Step 1: Create the skill**

Create `skills/sibyl-generate/SKILL.md`:

```markdown
---
name: sibyl-generate
description: Use when creating a new QA contract for a page, API endpoint, component, or feature. Accepts natural language prompts, shaping documents, screenshots, or any mix. Triggers on "create contract", "add contract", "define contract", "sibyl generate".
---

## Overview

Generates a `.contract.md` file from natural language input, shaping documents, or screenshots. Asks clarifying questions, then produces a structured contract.

## Flow

### Step 1: Verify setup

- Check that `sibyl-qa/` exists. If not: "Sibyl isn't set up yet. Run `/sibyl-init` first."
- Read `sibyl-qa/sibyl.config.md` to get platform and conventions.

### Step 2: Determine contract type

Ask: "What are we contracting?"
- Options: page, api, component, feature, global

### Step 3: Gather input

Ask: "Describe it, paste a brief, or reference a file/screenshot."

Accept any combination of:
- Natural language description
- File path to a shaping document (read it)
- Screenshot path (read the image)
- Inline structured brief

If the input is thin, ask follow-up questions ONE AT A TIME:
- "What does it look like?" (visual)
- "What does it do?" (behavior)
- "What data does it send/receive?" (data)
- "Any hard constraints?" (rules)

### Step 4: Determine path

Based on contract type and platform:
- **page (web)**: Ask for URL path → `sibyl-qa/pages/<path>/`
- **page (mobile)**: Ask for screen route → `sibyl-qa/screens/<route>/`
- **api**: Ask for method + endpoint → `sibyl-qa/api/<path>/<method>.contract.md`
- **component**: Ask for name → `sibyl-qa/components/<name>.contract.md`
- **feature**: Ask for name → `sibyl-qa/features/<name>.contract.md`
- **global**: Ask for name → `sibyl-qa/global/<name>.contract.md`

### Step 5: Generate contract

Create the `.contract.md` file with:
- Frontmatter: name, description, type, path, status: draft, priority (ask or default from config), platform, references, created/updated dates
- **Visual** section: layout, elements, screenshots (if provided)
- **Behavior** section: user flows, interactions, state changes
- **Data** section: API shapes, request/response (if applicable)
- **Rules** section: MUST/MUST NOT/MAY constraints derived from the description

Omit sections that don't apply (e.g., no Visual for API contracts, no Data for pure visual components).

### Step 6: Update INDEX.md

Add an entry to `sibyl-qa/INDEX.md` under the appropriate section.

### Step 7: Review and lock

Present the generated contract to the developer. Ask: "Does this look right? Lock it?"

- If yes → set `status: locked` in frontmatter
- If no → ask what to change, regenerate

## Contract Quality

- Each bullet in Visual/Behavior/Data should be specific and testable
- Rules should use RFC 2119 keywords: MUST, MUST NOT, SHOULD, MAY
- Priority should reflect actual importance (critical = auth, payments; low = cosmetic)
- References should link to related contracts when they exist

## Multiple Contracts

If the user describes something that spans multiple contracts (e.g., "the login flow" which involves a page + API), generate each contract separately and link them via `references`.
```

**Step 2: Create the command**

Create `commands/sibyl-generate.md`:

```markdown
---
disable-model-invocation: true
---

Invoke the sibyl:sibyl-generate skill to create a new QA contract.
```

**Step 3: Commit**

```bash
git add skills/sibyl-generate/ commands/sibyl-generate.md
git commit -m "feat: add /sibyl-generate skill and command"
```

---

### Task 8: /sibyl-check skill + command

The validation skill that checks changes against locked contracts.

**Files:**
- Create: `skills/sibyl-check/SKILL.md`
- Create: `commands/sibyl-check.md`

**Step 1: Create the skill**

Create `skills/sibyl-check/SKILL.md`:

```markdown
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
```

**Step 2: Create the command**

Create `commands/sibyl-check.md`:

```markdown
---
disable-model-invocation: true
---

Invoke the sibyl:sibyl-check skill to validate current changes against locked QA contracts.
```

**Step 3: Commit**

```bash
git add skills/sibyl-check/ commands/sibyl-check.md
git commit -m "feat: add /sibyl-check skill and command"
```

---

### Task 9: /sibyl-amend skill + command

The amendment skill for updating contracts after intentional changes.

**Files:**
- Create: `skills/sibyl-amend/SKILL.md`
- Create: `commands/sibyl-amend.md`

**Step 1: Create the skill**

Create `skills/sibyl-amend/SKILL.md`:

```markdown
---
name: sibyl-amend
description: Use when a Sibyl contract violation was flagged and the developer wants to update the contract to reflect an intentional change. Triggers on "amend contract", "update contract", "sibyl amend", or after a /sibyl-check violation.
---

## Overview

Proposes a contract update when a violation represents an intentional change rather than a bug.

## Flow

### Step 1: Identify the contract

If called after a `/sibyl-check` violation:
- Use the contract and clause from the violation context

If called standalone:
- Ask: "Which contract needs updating?"
- List contracts from `sibyl-qa/INDEX.md`
- Or accept a direct path

### Step 2: Show current state

Display the current contract clause(s) that need updating.

### Step 3: Propose amendment

Based on the code changes (from git diff), propose updated clause text:

```
📝 Sibyl Amendment Proposal

Contract: sibyl-qa/pages/auth/login/login.contract.md

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
   sibyl-qa/pages/auth/login/login.contract.md
   Updated: Behavior #2
```

## Multiple Amendments

If multiple clauses need updating in the same contract, present them all at once for efficiency. But still require explicit approval.

## New Clauses

If the code change introduces new behavior not covered by any clause:
- Propose adding a new clause to the appropriate section
- Flag it clearly as an ADDITION, not a modification
```

**Step 2: Create the command**

Create `commands/sibyl-amend.md`:

```markdown
---
disable-model-invocation: true
---

Invoke the sibyl:sibyl-amend skill to propose updating a QA contract after an intentional change.
```

**Step 3: Commit**

```bash
git add skills/sibyl-amend/ commands/sibyl-amend.md
git commit -m "feat: add /sibyl-amend skill and command"
```

---

### Task 10: /sibyl-scan skill + command

The auto-generation skill that creates baseline contracts from existing code.

**Files:**
- Create: `skills/sibyl-scan/SKILL.md`
- Create: `commands/sibyl-scan.md`

**Step 1: Create the skill**

Create `skills/sibyl-scan/SKILL.md`:

```markdown
---
name: sibyl-scan
description: Use when generating baseline QA contracts from an existing codebase. Scans code to identify pages, API endpoints, and components, then generates draft contracts. Triggers on "scan codebase", "generate baseline contracts", "sibyl scan".
---

## Overview

Auto-generates draft contracts by scanning the existing codebase. Identifies pages, API endpoints, and components, then creates `.contract.md` files for each.

## Flow

### Step 1: Verify setup

- Check that `sibyl-qa/` exists. If not: "Sibyl isn't set up yet. Run `/sibyl-init` first."
- Read `sibyl-qa/sibyl.config.md` to get platform and conventions.

### Step 2: Scan the codebase

Based on platform, look for:

**Web (pages):**
- Route definitions (Next.js `app/` or `pages/`, React Router config, Vue Router, etc.)
- Each route = potential page contract

**API:**
- Route/handler definitions (Express routes, FastAPI endpoints, Rails controllers, etc.)
- Each endpoint = potential API contract

**Components:**
- Component directories (React `components/`, Vue `components/`, etc.)
- Focus on "smart" or page-level components, skip primitives

**Mobile:**
- Screen definitions (React Navigation, Flutter routes, etc.)
- Each screen = potential screen contract

### Step 3: Present findings

Show what was found:

```
📋 Sibyl Scan Results

Found:
  Pages: 12
  API endpoints: 8
  Components: 23

Recommended contracts (high-value targets):
  ✓ /auth/login — page with form + API call
  ✓ /dashboard — main page, complex layout
  ✓ POST /api/auth/login — auth endpoint
  ✓ DataTable — reused in 5 pages
  ... (list all)

Generate contracts for all, or select specific ones?
```

### Step 4: Generate contracts

For each selected item:
1. Read the source files for that page/endpoint/component
2. Analyze the code to extract:
   - Visual: UI elements, layout structure
   - Behavior: User flows, event handlers, state changes
   - Data: API calls, request/response shapes, props
   - Rules: Infer MUST/MUST NOT from critical paths (auth, payments, etc.)
3. Generate `.contract.md` as `status: draft`
4. Update `sibyl-qa/INDEX.md`

### Step 5: Review cycle

Present each generated contract for review:
- Show the contract
- Ask: "Looks right? Edit anything? Lock it?"
- Apply changes and lock on approval

For large scans (10+ contracts), offer batch mode:
- Generate all as drafts
- Developer reviews and locks individually later

## Scan Depth

- **Quick scan**: Route/endpoint names only → minimal contracts with paths and types
- **Deep scan**: Read source files → full contracts with visual/behavior/data/rules

Ask which mode before starting: "Quick scan (paths only) or deep scan (full analysis)?"
```

**Step 2: Create the command**

Create `commands/sibyl-scan.md`:

```markdown
---
disable-model-invocation: true
---

Invoke the sibyl:sibyl-scan skill to auto-generate baseline QA contracts from the existing codebase.
```

**Step 3: Commit**

```bash
git add skills/sibyl-scan/ commands/sibyl-scan.md
git commit -m "feat: add /sibyl-scan skill and command"
```

---

### Task 11: Push PR 2

**Step 1: Push feature branch**

```bash
git push -u origin feat/skills
```

**Step 2: Create PR**

```bash
gh pr create --title "feat: add Sibyl skills and commands" --body "$(cat <<'EOF'
## Summary
- Add 5 Sibyl skills: /sibyl-init, /sibyl-generate, /sibyl-check, /sibyl-amend, /sibyl-scan
- Add matching slash commands for each skill
- Skills handle contract creation, validation, amendment, and codebase scanning

## Skills
| Skill | Purpose |
|-------|---------|
| `/sibyl-init` | Interactive project setup — scaffolds sibyl-qa/ directory |
| `/sibyl-generate` | Create contracts from prompts, docs, or screenshots |
| `/sibyl-check` | Validate code changes against locked contracts |
| `/sibyl-amend` | Update contracts after intentional changes |
| `/sibyl-scan` | Auto-generate baseline contracts from existing code |

## Test plan
- [ ] Install plugin and verify skills appear in Claude Code
- [ ] Run /sibyl-init and verify directory structure created
- [ ] Run /sibyl-generate with a prompt and verify contract created
- [ ] Make a change that violates a contract and run /sibyl-check
- [ ] Run /sibyl-amend after a violation and verify contract updated
- [ ] Run /sibyl-scan on a sample project and verify contracts generated

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```
