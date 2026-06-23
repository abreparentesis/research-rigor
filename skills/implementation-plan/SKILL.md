---
name: implementation-plan
description: Use when you have a spec or requirements for a multi-step task, before touching code. Turns a spec into a step-by-step, reviewable implementation plan of bite-sized, independently testable tasks, then hands off to execution.
---

<!-- Adapted from obra/superpowers `writing-plans` + `executing-plans` (MIT, (c) 2025
     Jesse Vincent), decoupled from superpowers-only sub-skills, with task-ordering
     discipline influenced by github/spec-kit (MIT). See ../../NOTICE. -->

# Implementation Plan

## Overview

Write a comprehensive implementation plan assuming the engineer has zero context for this codebase and questionable taste. Document everything they need: which files to touch per task, the code, the tests, the docs to check, and how to verify. Give the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume a skilled developer who knows almost nothing about this toolset or problem domain, and does not know good test design well.

**Announce at start:** "I'm using the implementation-plan skill to create the plan."

**Save plans to:** `docs/plans/YYYY-MM-DD-<feature-name>.md` (user preferences for location override this).

## Scope check

If the spec covers multiple independent subsystems, split it into one plan per subsystem. Each plan should produce working, testable software on its own.

## File structure

Before defining tasks, map which files will be created or modified and what each is responsible for. This is where decomposition gets locked in.

- Design units with clear boundaries and one responsibility each. Prefer smaller, focused files.
- Files that change together live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. Include a split only when a file you are modifying has grown unwieldy.

## Task right-sizing and ordering

A task is the smallest unit that carries its own test cycle and is worth a fresh reviewer's gate. Fold setup, configuration, scaffolding, and docs into the task whose deliverable needs them; split only where a reviewer could meaningfully reject one task while approving its neighbor. Each task ends with an independently testable deliverable.

Order tasks by dependency: a task comes after every task it consumes from. Where two tasks are independent, say so, they can be done in parallel or in any order.

## Bite-sized step granularity

Each step is one action (2-5 minutes):
- "Write the failing test" (step)
- "Run it to confirm it fails" (step)
- "Implement the minimal code to pass" (step)
- "Run the tests and confirm they pass" (step)
- "Commit" (step)

## Plan document header

Every plan MUST start with this header:

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** implement this plan task-by-task with a review checkpoint
> between tasks (see "Executing the plan" below). Steps use checkbox (`- [ ]`) syntax.

**Goal:** [one sentence describing what this builds]

**Architecture:** [2-3 sentences about the approach]

**Tech Stack:** [key technologies/libraries]

## Global Constraints

[The spec's project-wide requirements, version floors, dependency limits, naming and
copy rules, platform requirements, one line each, exact values copied verbatim from the
spec. Every task's requirements implicitly include this section.]

---
```

## Task structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [what this task uses from earlier tasks, exact signatures]
- Produces: [what later tasks rely on, exact function names, parameter and return
  types. A task's implementer sees only their own task; this block is how they learn
  the names and types neighboring tasks use.]

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## No placeholders

Every step must contain the actual content an engineer needs. These are plan failures, never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" without the actual test code
- "Similar to Task N", repeat the code, the engineer may read tasks out of order
- Steps that say what to do without showing how (code steps need code blocks)
- References to types, functions, or methods not defined in any task

## Remember
- Exact file paths always.
- Complete code in every step that changes code.
- Exact commands with expected output.
- DRY, YAGNI, TDD, frequent commits.

## Self-review

After writing the complete plan, check it against the spec with fresh eyes. This is a checklist you run yourself.

1. **Spec coverage**: skim each requirement in the spec. Can you point to a task that implements it? List any gaps and add tasks for them.
2. **Placeholder scan**: search the plan for the red flags above and fix them.
3. **Type consistency**: do types, signatures, and names used in later tasks match what earlier tasks defined? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

Fix issues inline. No need to re-review.

## Executing the plan

After saving the plan, offer the execution choice:

> "Plan complete and saved to `docs/plans/<filename>.md`. Two ways to execute:
> 1. **Subagent-driven (recommended where subagents are available)**: a fresh subagent per task with a review checkpoint between tasks. Higher quality, fast iteration.
> 2. **Inline**: execute tasks in this session with checkpoints for review.
> Which approach?"

For either approach, follow this discipline:

1. **Load and review the plan critically.** Identify questions or concerns. Raise blockers before starting. Create a todo per task.
2. **Execute each task in order.** Mark in_progress, follow each step exactly, run the verifications as written, mark completed.
3. **Stop and ask, do not guess,** when you hit a blocker (missing dependency, failing test, unclear instruction) or the plan has a critical gap. Return to the plan when the approach needs rethinking.
4. **Never start implementation on `main`/`master` without explicit user consent.** Prefer an isolated branch or git worktree.
5. **When all tasks pass,** verify the full test suite, then present completion options (merge, PR, or further work) and execute the user's choice.
