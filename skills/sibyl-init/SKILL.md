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
