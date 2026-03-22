---
name: themis-init
description: Use when setting up Themis QA contracts in a project for the first time. Scaffolds themis-qa/ directory, config, and INDEX.md. Triggers on "set up themis", "initialize contracts", "add QA contracts".
---

## Overview

Interactive setup that scaffolds the `themis-qa/` directory structure in the current project.

## Flow

1. **Ask platform**: "What platform(s) does this project target?"
   - Options: web, mobile, api, universal

2. **Ask base URL**: "What's your local dev URL?" (skip if api-only)

3. **Ask shaping doc path**: "Where do you keep shaping docs / briefs? (leave empty to skip)"

4. **Scaffold directory structure based on platform:**

   **Web:**
   ```
   themis-qa/
   ├── themis.config.md
   ├── INDEX.md
   ├── global/
   ├── pages/
   ├── features/
   └── components/
   ```

   **Mobile:**
   ```
   themis-qa/
   ├── themis.config.md
   ├── INDEX.md
   ├── global/
   ├── screens/
   ├── features/
   └── components/
   ```

   **API:**
   ```
   themis-qa/
   ├── themis.config.md
   ├── INDEX.md
   ├── global/
   ├── api/
   └── features/
   ```

   **Universal:**
   ```
   themis-qa/
   ├── themis.config.md
   ├── INDEX.md
   ├── global/
   ├── pages/
   ├── screens/
   ├── api/
   ├── features/
   └── components/
   ```

5. **Generate `themis.config.md`** from template with user's answers

6. **Generate empty `INDEX.md`** from template

7. **Add Themis instructions to project's CLAUDE.md:**
   Append this block:
   ```markdown
   ## Themis QA Contracts

   This project uses Themis QA contracts. Before making changes:

   - **Always load `themis-qa/INDEX.md` first** to know which contracts exist
   - Read the relevant contract files for any page/feature you're about to modify
   - Cross-reference modified files against contract paths
   - Never modify a locked contract without running `/themis-amend`
   - If your change would break a locked contract, stop and flag it
   - Use `/themis-generate` to create new contracts
   - Use `/themis-check` to validate changes against contracts
   ```

8. **Confirm**: "Themis is set up. Use `/themis-generate` to create your first contract."

## Important

- If `themis-qa/` already exists, warn and ask before overwriting
- Create directories using mkdir, not by writing placeholder files
- Use today's date in generated config
