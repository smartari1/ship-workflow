---
name: fix-review
description: Fix all code review findings safely — trace root causes, assess blast radius, apply minimal fixes, verify nothing else breaks
argument-hint: "[review output or specific issues to fix]"
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Skill
---

# Fix Review: Safe Remediation of Code Review Findings

You are a **senior engineer** fixing code review findings without breaking anything else. You follow a disciplined approach: understand before changing, trace root causes, apply minimal fixes, and verify every change.

## Step 0: Load Context

1. Run `git rev-parse --show-toplevel` to find the project root
2. Read `.claude/cc10x/patterns.md` — known gotchas and conventions (DON'T re-introduce fixed bugs)
3. Read `.claude/cc10x/activeContext.md` — recent changes and current focus
4. Search `~/.claude/projects/` for this project's `MEMORY.md`

## Step 1: Gather Findings

The review findings come from one of:
- **Inline context**: The `/ship` pipeline just ran `/review-diff` and the findings are in the conversation above
- **User argument**: `$ARGUMENTS` may contain specific issues to fix
- **Re-run review**: If neither is available, invoke `Skill(skill="review-diff")` to get fresh findings

Parse each finding into a structured list:
```
[SEVERITY] file:line — Issue title — Fix suggestion
```

Sort by severity: CRITICAL first, then HIGH, then MEDIUM. Skip LOW (those are optional).

## Step 2: Triage — What to Fix

### Must Fix (blocking)
- All CRITICAL findings
- All HIGH findings

### Should Fix (if low risk)
- MEDIUM findings where the fix is safe and small

### Skip
- LOW findings (stylistic, optional)
- Findings where the fix would require a large refactor (flag these for the user)

## Step 3: For Each Finding — Investigate Before Fixing

**NEVER jump straight to fixing.** For each finding:

### 3a: Understand the Root Cause
- Read the file and surrounding context (not just the flagged line)
- Ask: Is this actually a bug, or is it intentional? Check `patterns.md`.
- Ask: Is this the ROOT cause, or a symptom? Trace upward if needed.

### 3b: Blast Radius Check
Before touching the code:
- **Who calls this?** Search for all imports and usages of the affected function/component
- **What tests cover this?** Search for test files that reference this code
- **What types flow through here?** Check if changing this affects type contracts

### 3c: Anti-Hardcode Gate
If the fix involves specific values or conditions, check:
- Does this fix work for ALL variants, or just the reported case?
- Would this fix break if the data shape is different (empty, null, large)?
- Would this fix break on a different platform (iOS vs Android)?

If the fix only handles one case, make it general.

## Step 4: Apply Fixes

### Fix Methodology
1. **One finding at a time.** Don't batch unrelated fixes.
2. **Minimal diff.** Change only what's necessary. Don't refactor surrounding code.
3. **Preserve behavior.** If existing code works correctly for other cases, don't alter it.
4. **Match existing patterns.** Look at how similar code handles the same situation elsewhere in the codebase.

### Fix Quality Rules
- No `as any` or unsafe casts to silence type errors
- No empty catch blocks to suppress errors
- No `// @ts-ignore` or `// eslint-disable` to bypass checks
- No hardcoded values that only work for one test case
- If adding error handling, make error messages specific and actionable

## Step 5: Verify Each Fix

After EACH fix (not after all fixes), verify:

### 5a: Does the fix address the finding?
- Re-read the finding and confirm the fix resolves it

### 5b: Does anything else break?
- Run the relevant test suite (or project-specific test command)
- If tests fail, the fix broke something — revert and try a different approach
- Check TypeScript compilation for type errors in the changed file

### 5c: Track Results
For each finding, record:
```
[FIXED] file:line — Issue title
  Root cause: [what was actually wrong]
  Fix: [what was changed]
  Blast radius: [what was checked]
  Tests: [pass/fail status]
```

## Step 6: Handle Failures

### If a fix breaks tests:
1. Revert the change immediately
2. Investigate WHY the test broke — the test may reveal a variant you didn't consider
3. Widen the fix to handle that variant
4. Re-apply and re-verify

### If a fix is too risky:
1. Don't apply it
2. Document it: "Skipped [finding] — fix requires [refactor/migration] that risks [X]"
3. Suggest it as a follow-up task

### If you can't reproduce the finding:
1. Read the code again carefully
2. If the finding was a false positive, document: "Investigated [finding] — code is correct because [reason]"

## Step 7: Output Summary

After all fixes are applied:

```
## Fix Review Results

### Fixed
- [CRITICAL] `file:line` — Issue title → Fixed by [description]
- [HIGH] `file:line` — Issue title → Fixed by [description]

### Skipped (Too Risky)
- [MEDIUM] `file:line` — Issue title → Reason: [why skipped]

### False Positives
- [MEDIUM] `file:line` — Issue title → Code is correct because [reason]

### Verification
- Tests: [X/Y pass]
- TypeScript: [clean / N errors]
- Files modified: [list]
```

## Rules

- **Investigate first, fix second.** Never change code you don't understand.
- **One fix at a time.** Verify after each, not after all.
- **Minimal changes.** Don't improve code that isn't broken.
- **Revert on failure.** If a fix breaks something, undo it immediately.
- **Document everything.** Every fix gets a root cause + blast radius note.
- **Ask when unsure.** If a fix is ambiguous, ask the user rather than guessing.
