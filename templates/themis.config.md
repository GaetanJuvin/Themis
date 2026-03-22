---
name: Themis QA
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

- **Always load `themis-qa/INDEX.md` first** to know which contracts exist
- Read the relevant contract files for any page/feature you're about to modify
- Cross-reference modified files against contract paths
- Never modify a locked contract without running `/themis-amend`
- When generating new contracts, always update `INDEX.md`
