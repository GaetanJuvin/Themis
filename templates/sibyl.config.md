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
