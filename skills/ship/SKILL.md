---
name: ship
description: Full pre-commit pipeline — review, investigate, fix, update memory, then create a detailed commit
disable-model-invocation: true
argument-hint: "[skip: skip-look | skip-fix | skip-memory]"
allowed-tools: Bash, Read, Write, Edit, Grep, Glob, Skill
---

# Ship Pipeline

Run the full pre-commit pipeline in sequence:
1. **Review Diff** — senior code review on all changes
2. **Look** — architect-level investigation of flagged areas
3. **Fix Review** — fix all findings safely with blast radius awareness
4. **Update Memory** — extract knowledge from the final diff into memory files
5. **Big Commit** — create a detailed, comprehensive commit

## Pipeline Execution

Execute these skills **strictly in order**. Each must complete before the next starts.

### Optional Skip Argument

The user may pass: `$ARGUMENTS`

- No argument: run all 5 steps
- `skip-look`: skip step 2, go review → fix → memory → commit
- `skip-fix`: skip steps 2+3, go review → memory → commit (review-only, no auto-fix)
- `skip-memory`: skip step 4, go review → look → fix → commit

---

### Step 1: Review Diff

Invoke the `review-diff` skill to perform a full code review:

```
Skill(skill="review-diff")
```

Wait for completion. Then evaluate the result:

- **If clean (no CRITICAL or HIGH issues)**: Skip steps 2+3, proceed directly to step 4.
- **If CRITICAL or HIGH issues found**: Show the user the review summary, then proceed to step 2.

---

### Step 2: Look — Investigate Findings

Invoke the `look` skill to deeply understand the areas flagged in the review. Pass the affected files/areas as context so it knows where to focus:

```
Skill(skill="look", args="<summary of flagged areas from review>")
```

Wait for completion. The investigation output gives step 3 the context it needs — root causes, blast radius, variant dimensions — so fixes are safe.

---

### Step 3: Fix Review Findings

Invoke the `fix-review` skill to fix all CRITICAL and HIGH findings. It has the review findings AND the look investigation in conversation context:

```
Skill(skill="fix-review")
```

Wait for completion. Then evaluate:

- **If all findings fixed and tests pass**: Proceed to step 4.
- **If some findings were skipped as too risky**: Show them to the user and ask: "These issues need manual attention. Continue to commit with known issues, or stop to fix manually?"
- **If fixes broke tests and couldn't be resolved**: STOP the pipeline. Show the user what happened. Tell them: "Fix attempt caused regressions — manual intervention needed. Run `/ship` after fixing to restart."

---

### Step 4: Update Memory

Invoke the `update-memory` skill to capture knowledge from the final state of the diff (including any fixes from step 3):

```
Skill(skill="update-memory")
```

Wait for completion. Briefly confirm what was updated, then proceed.

---

### Step 5: Big Commit

Invoke the `big-commit` skill to create a detailed commit:

```
Skill(skill="big-commit")
```

This will show the commit message and ask for user approval before committing.

---

## Pipeline Output

After all steps complete, show a brief summary:

```
## Ship Pipeline Complete

1. Review: [Approve / Approve with Notes / Changes Requested] — [N critical, N high, N medium]
2. Look: Investigated [N areas]
3. Fix: [N fixed, N skipped, N false positives]
4. Memory: Updated [N files]
5. Commit: [commit hash] on branch [branch-name]
```
