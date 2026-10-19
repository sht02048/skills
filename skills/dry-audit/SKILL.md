---
name: dry-audit
description: Find duplicated or diverging repository logic, validate shared responsibility through call and dependency graphs, and report prioritized consolidation opportunities without editing code. Use when the user asks for a DRY audit, duplicate logic review, copy-paste detection, repeated business rules, consolidation candidates, says "중복 코드", "중복 로직 찾아줘", or "DRY 검사", or invokes `$dry-audit`.
---

# DRY Audit

Find duplicated responsibility, not merely similar text. Diagnose only; do not
edit, refactor, commit, or install analysis tools unless the user explicitly
asks.

## Workflow

1. Read repository instructions and establish the requested scope. Exclude
   generated, vendored, build, snapshot, migration, and fixture files unless
   they are the suspected source of drift.
2. Discover candidates.
   - When a repository `.codegraph/` index exists, run one repository-level
     `codegraph_explore` hotspot query naming these categories: domain
     conditions, permission checks, parsing and normalization, validation,
     mapping, formatting, state transitions, error translation, and API
     orchestration. If the MCP tool is unavailable, use `codegraph explore`.
   - Search the same categories with `rg`. Text matches are candidates, not
     proof.
   - Never create or refresh an index. Indexing belongs to the user.
3. Shortlist candidates that have at least two concrete implementations.
4. Run at least one graph query for every shortlisted candidate before
   validating it.
   - With CodeGraph, run a focused `codegraph_explore` query requesting the
     relevant symbols, callers, callees, dynamic-dispatch paths, and blast
     radius.
   - Without CodeGraph, or across a language boundary it cannot answer, use LSP
     references, call hierarchy, workspace symbols, and definitions;
     repository-provided dependency or module-graph commands; or AST-aware
     search for repeated structures.
   Do not add dependencies solely for this audit.
5. Validate each candidate before reporting it:
   - prove the implementations own the same rule or must change together;
   - trace their callers and downstream consumers;
   - inspect tests, contracts, and Git history only when they clarify whether
     the duplication is intentional or already diverging;
   - identify the smallest existing seam that could own the shared rule.
6. Reject false positives. Do not report:
   - language idioms of three lines or fewer;
   - intentional boundary duplication that isolates packages, runtimes, or
     failure domains;
   - test setup repeated for readability;
   - speculative abstractions without a current shared rule.

## Priority

Assign the highest level supported by evidence. Never inflate priority from line
count alone.

- `P0`: Duplicated security, authorization, privacy, money, safety, or data
  integrity logic is already inconsistent or creates an active critical defect.
- `P1`: A core domain rule exists in multiple runtime paths, and the copies
  together have three or more callers or are used from two or more top-level
  modules. Consistent duplicated security logic is `P1`; it becomes `P0` only
  once the copies diverge.
- `P2`: Non-critical logic is repeated in three or more places, or Git history
  shows the copies were edited together.
- `P3`: Local mechanical duplication has a clear low-risk consolidation seam
  but limited behavioral impact. Omit it when the shared version would need a
  per-caller flag or parameter, or would add more lines than it removes.

## Report

Return findings only, in the user's language, as a priority-ordered Markdown
list. Translate the section labels into that language. Do not use a table.
Limit the report to the ten highest-value findings; when more are proven, state
how many were omitted per priority. Within a priority, order by blast radius,
then confidence.

Use this exact shape, shown for a Korean request:

```markdown
## P0

- **Finding title**
  - 위치: `path:line`, `path:line`
  - 그래프 근거: `caller -> duplicated rule -> consumer`
  - 위험: concrete divergence or synchronized-change burden
  - 통합 지점: smallest existing module or symbol that should own it

## P1

- 없음
```

Include `P0` through `P3`, writing `- 없음` for empty sections. Every finding
must cite exact repository locations and graph or caller evidence. End with one
line containing the total count by priority. If nothing is proven, say that no
evidence-backed DRY issue was found.
