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
