---
name: dry-audit
description: Find duplicated or diverging repository logic, validate shared responsibility through call and dependency graphs, and report prioritized consolidation opportunities without editing code. Use when the user asks for a DRY audit, duplicate logic review, copy-paste detection, repeated business rules, consolidation candidates, or `$dry-audit`.
---

# DRY Audit

Find duplicated responsibility, not merely similar text. Diagnose only; do not
edit, refactor, commit, or install analysis tools unless the user explicitly
asks.

## Workflow

1. Read repository instructions and establish the requested scope. Exclude
   generated, vendored, build, snapshot, migration, and fixture files unless
   they are the suspected source of drift.
2. Check for a repository `.codegraph/` index before broad text search.
   - When present, use `codegraph_explore` first. If the MCP tool is unavailable,
     use `codegraph explore`.
   - Run one repository-level hotspot query, then a focused query for every
     shortlisted finding. Request the relevant symbols, callers, callees,
     dynamic-dispatch paths, and blast radius.
   - Never create or refresh an index. Indexing belongs to the user.
3. Use other existing graph surfaces aggressively when CodeGraph is absent or
   cannot answer a language boundary:
   - LSP references, call hierarchy, workspace symbols, and definitions;
   - repository-provided dependency or module-graph commands;
   - AST-aware search for repeated structures.
   Do not add dependencies solely for this audit.
4. Use `rg` for candidate discovery, not proof. Search repeated domain
   conditions, permission checks, parsing and normalization, validation,
   mapping, formatting, state transitions, error translation, and API
   orchestration.
5. Validate each candidate before reporting it:
   - identify at least two concrete implementations;
   - prove they own the same rule or must change together;
   - trace their callers and downstream consumers;
   - inspect tests, contracts, and Git history only when they clarify whether
     the duplication is intentional or already diverging;
   - identify the smallest existing seam that could own the shared rule.
6. Reject false positives. Do not report:
   - coincidentally similar syntax with different responsibilities;
   - tiny language idioms whose abstraction would be longer or harder to read;
   - intentional boundary duplication that isolates packages, runtimes, or
     failure domains;
   - test setup repeated for readability;
   - speculative abstractions without a current shared rule.

## Priority

Assign the highest level supported by evidence. Never inflate priority from line
count alone.

- `P0`: Duplicated security, authorization, privacy, money, safety, or data
  integrity logic is already inconsistent or creates an active critical defect.
- `P1`: A core domain rule exists in multiple runtime paths with high fan-in or
  blast radius, and divergence can produce materially different behavior.
- `P2`: Non-critical logic is repeated across modules and has concrete drift or
  a meaningful synchronized-change burden.
- `P3`: Local mechanical duplication has a clear low-risk consolidation seam
  but limited behavioral impact. Omit it when the abstraction would not pay for
  itself.

## Report

Return findings only, in the user's language, as a priority-ordered Markdown
list. Do not use a table. Limit the report to the ten highest-value findings.
Within a priority, order by blast radius and confidence.

Use this exact shape:

```markdown
## P0

- **Finding title**
  - Locations: `path:line`, `path:line`
  - Graph evidence: `caller -> duplicated rule -> consumer`
  - Risk: concrete divergence or synchronized-change burden
  - Consolidation seam: smallest existing module or symbol that should own it

## P1

- 없음
```

Include `P0` through `P3`, writing `- 없음` for empty sections. Every finding
must cite exact repository locations and graph or caller evidence. End with one
line containing the total count by priority. If no actionable duplication is
proven, report all sections as `없음` and say that no evidence-backed DRY issue
was found.
