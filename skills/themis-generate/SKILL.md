---
name: themis-generate
description: Use when creating a new QA contract for a page, API endpoint, component, or feature. Accepts natural language prompts, shaping documents, screenshots, or any mix. Triggers on "create contract", "add contract", "define contract", "themis generate".
---

## Overview

Generates a `.contract.md` file from natural language input, shaping documents, or screenshots. Asks clarifying questions, then produces a structured contract.

## Flow

### Step 1: Verify setup

- Check that `themis-qa/` exists. If not: "Themis isn't set up yet. Run `/themis-init` first."
- Read `themis-qa/themis.config.md` to get platform and conventions.

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
- **page (web)**: Ask for URL path → `themis-qa/pages/<path>/`
- **page (mobile)**: Ask for screen route → `themis-qa/screens/<route>/`
- **api**: Ask for method + endpoint → `themis-qa/api/<path>/<method>.contract.md`
- **component**: Ask for name → `themis-qa/components/<name>.contract.md`
- **feature**: Ask for name → `themis-qa/features/<name>.contract.md`
- **global**: Ask for name → `themis-qa/global/<name>.contract.md`

### Step 5: Generate contract

Create the `.contract.md` file with:
- Frontmatter: name, description, type, path, status: draft, priority (ask or default from config), platform, references, created/updated dates
- **Visual** section: layout, elements, screenshots (if provided)
- **Behavior** section: user flows, interactions, state changes
- **Data** section: API shapes, request/response (if applicable)
- **Rules** section: MUST/MUST NOT/MAY constraints derived from the description

Omit sections that don't apply (e.g., no Visual for API contracts, no Data for pure visual components).

### Step 6: Update INDEX.md

Add an entry to `themis-qa/INDEX.md` under the appropriate section.

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
