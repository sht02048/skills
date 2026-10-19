---
name: skill-refine
description: Audit an existing agent skill (`SKILL.md` and its bundled files) for vague rules, missing reasoning, filler, weak triggers, and mixed responsibilities, then rewrite it to be strict, focused, and reasoned while preserving the author's intent. Use when the user asks to analyze, review, critique, tighten, improve, or refactor a skill, asks why a skill behaves inconsistently, says "스킬 분석", "스킬 개선", or "스킬 검사", or invokes `$skill-refine`.
---

# Skill Refine

A skill exists to make a probabilistic agent walk the same decision process
every time. Judge every line by one question: does it change what the agent
does? Rewrite the skill so the answer is yes for every line that remains.

## Scope

1. Resolve the target skill directory. If the user names none and the current
   repository has exactly one changed `SKILL.md`, use it; otherwise ask.
   Edit the skill's source repository, never an installed copy such as a
   plugin cache, because the next install or update overwrites it.
2. Read the full `SKILL.md`, every file it references, and sibling metadata
   such as `agents/openai.yaml`. Never edit a file you have not read.
3. Read repository instructions and two sibling skills when they exist. Match
   their language, heading style, frontmatter fields, and output conventions;
   a refined skill that diverges from its siblings becomes harder to maintain.
4. Default to analysis followed by edits. Stop after the report when the user
   asks only to analyze, review, or diagnose.

## Analysis

Check each item in order and record every violation with its exact location.

1. **Trigger.** The `description` is the only text the agent sees when choosing
   a skill. It must state what the skill does and when to use it, with the
   concrete phrases, file types, or symptoms a user would actually say. Flag a
   description that is generic, lacks a "Use when" clause, or would match tasks
   the body does not handle.
2. **Focus.** One skill owns one job. Flag a skill that combines distinct tasks
   whose steps rarely run together, such as building and reviewing, or two
   unrelated domains. Propose the split with a name and description for each
   part; do not perform a split without the user's approval, because it changes
   how they invoke their tools.
3. **Strictness.** Soft wording leaves the agent on its own defaults. Flag
   hedges and unmeasurable qualifiers: `reasonably`, `appropriate`, `where
   appropriate`, `tasteful`, `good`, `clean`, `try to`, `consider`, `if
   possible`, `as needed`, `generally`, `usually`, `ideally`, `should probably`,
   `적절히`, `가능하면`, `필요에 따라`, `되도록`, `깔끔하게`, `잘`. Each must become
   a number, threshold, enumerated condition, ordered decision, or explicit
   exception.
4. **Reasoning.** A rule without a reason is applied blindly, including where
   it does not fit, and cannot be extended to unlisted cases. Flag every
   non-obvious rule that lacks a one-clause reason. Do not demand reasons for
   rules whose purpose is self-evident.
5. **Weight.** `Never`, `always`, and `must` signal hard constraints. Flag hard
   constraints written softly, and flag emphasis spread across preferences,
   because uniform emphasis makes the real constraints indistinguishable.
6. **Line economy.** Flag sentences that explain what the agent already knows
   (what a modal, commit, or CSS transform is), restate another line, describe
   motivation or history, or address a human reader. These dilute the lines
   that carry domain expertise.
7. **Decision process.** Where outputs would otherwise vary between runs, the
   skill needs an ordered procedure, a precedence rule for conflicts, or an
   exact output shape. Flag open-ended steps such as "review the code" that give
   no order or stopping condition.
8. **Examples.** For each rule that is easy to misread, a short bad/good pair
   anchors the interpretation. Flag ambiguous rules that have none; flag long
   examples that teach nothing beyond their rule.
9. **Structure.** Flag broken references to bundled files, reference material
   inlined into `SKILL.md` that is only needed for some runs, and metadata that
   contradicts the rewritten description.

## Report

Write the report in the user's language as a Markdown list grouped by the
analysis items above, omitting items without findings. Do not use a table.
Each finding uses this shape:

```markdown
- **Strictness** `SKILL.md:24`
  - 원문: "Keep animations reasonably short."
  - 문제: "reasonably"는 행동을 바꾸지 못한다.
  - 수정안: "UI animations stay under 300ms; exits run ~20% faster than entrances."
```

End with the line count of `SKILL.md` and the count of findings per item.

## Rewrite

1. Preserve the author's intent. Never invent domain facts, thresholds, or
   reasons the source does not support; a fabricated number is stricter but
   wrong. When a vague rule needs a value only the author knows, collect those
   rules and ask once with `AskUserQuestion`, offering 2-4 concrete candidates
   per rule. When refining several skills, analyze all of them first and ask
   every open decision in one round. If the user declines, keep the rule and mark it in the report as
   unresolved.
2. Apply findings in this order: trigger, focus, strictness, reasoning,
   weight, line economy, decision process, examples, structure. Earlier fixes
   change what later checks see.
3. Write for the agent: imperative sentences, one rule per bullet, the reason
   in the same sentence or the next. Never add a section the source did not
   need.
4. Keep the frontmatter `name` unchanged unless the user approved a rename.
   Update `agents/openai.yaml`, README skill lists, and plugin descriptions when
   the description or name changed.
5. Do not commit, push, or install anything.

## Verify

Re-run every analysis item against the rewritten files. Then report, in the
user's language:

- line count before and after;
- findings resolved, findings left unresolved with the reason;
- any proposed split or rename awaiting approval.
