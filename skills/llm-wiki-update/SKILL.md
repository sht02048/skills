---
name: llm-wiki-update
description: Refresh a repository's tracked `llm-wiki` from current source, configuration, tests, documentation, and Git evidence while preserving local wiki conventions. Use when the user asks to update, refresh, sync, or maintain an existing `llm-wiki`, or invokes `$llm-wiki-update`.
---

# LLM Wiki Update

Keep an existing `llm-wiki/` aligned with the repository using concise,
source-backed synthesis. If the repository has no `llm-wiki/`, stop and report
that this skill updates existing wikis rather than inventing a new schema.

## Scope and safety

- Edit only `llm-wiki/**` unless the user explicitly expands the scope.
- Read and follow the closest applicable `AGENTS.md` plus wiki-local index,
  schema, contribution, and naming guidance when present.
- Preserve any documented raw-source versus synthesis boundary.
- Treat non-wiki working-tree changes as evidence only.
- Do not commit or push unless explicitly requested.
- Mark inaccessible facts as `Open question:` instead of inventing them.

## Evidence order

Resolve conflicts in this order:

1. Executable source, configuration, generated contracts, and tests
2. Current Git diff and recent history
3. Repository-local documentation and the existing wiki
4. Accessible external source material
5. Explicitly labeled `Inference:`

## Workflow

1. Inspect repository status, current branch, `llm-wiki/` structure, and its
   local instructions.
2. Determine the requested evidence range. Prefer an explicit base or range.
   Otherwise use upstream and remote-default evidence rather than assuming a
   branch name. Include staged and unstaged diffs when local work is in scope.
3. Read only the source files needed to verify changed facts.
4. Map facts into the wiki's existing information architecture. Update an
   existing page before adding a new one. Add a page only when current pages
   would become ambiguous or overly broad.
5. Preserve required page sections and citation style. Attach concrete
   repository paths to durable claims. Label inference and unresolved questions.
6. Update the wiki catalog, index, chronology, or change log only when its local
   conventions require it.
7. Validate:
   - run `git diff --check -- llm-wiki`;
   - verify changed synthesized pages contain the repository-required source
     section or citation form;
   - verify added or renamed pages appear in the local index when one exists;
   - verify required chronology entries were added exactly once;
   - confirm the skill did not modify non-wiki files.

## Report

Report in Korean:

- changed `llm-wiki/**` files;
- repository evidence reflected;
- validation commands and results;
- remaining `Open question:` items or stale-data risks.

Keep the report concise and do not claim code tests were run unless they were
relevant and actually executed.
