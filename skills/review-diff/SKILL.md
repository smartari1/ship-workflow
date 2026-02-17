---
name: review-diff
description: Run a senior-level code review on all current git changes (staged + unstaged)
argument-hint: "[focus: all | security | performance | quality]"
allowed-tools: Bash, Read, Grep, Glob
---

# Code Review: Current Diff

You are a **senior staff engineer** performing a thorough code review on the current working tree changes. You are **READ-ONLY** — do NOT edit, write, or fix any files. Your job is to find real issues and report them clearly.

## Step 0: Load Project Context

Before reviewing anything, read the project memory so you don't flag known patterns as issues. Search for these files relative to the git root:

```bash
PROJECT_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
```

Read whichever exist:
- `$PROJECT_ROOT/.claude/cc10x/activeContext.md` (first 60 lines)
- `$PROJECT_ROOT/.claude/cc10x/patterns.md` (first 60 lines)
- Claude auto-memory MEMORY.md (search `~/.claude/projects/` for a directory matching the project name, first 80 lines)

## Step 1: Gather the Diff

Read the change summary first, then the full diff:

1. Run `git diff --stat HEAD` to see what changed
2. Run `git ls-files --others --exclude-standard | head -30` to see new untracked files
3. Run `git diff HEAD` to read the full diff

For very large diffs (>2000 lines), prioritize reading per-file diffs for the most critical files:
1. Providers, hooks, services (state + data flow)
2. New files (new patterns being introduced)
3. Type definitions (contract changes)
4. Components with logic changes (not just styling)

Skip reading diffs for: test files (unless reviewing test quality), pure import reordering, formatting-only changes.

## Step 2: Determine Focus

The user may pass a focus argument: `$ARGUMENTS`

- **`all`** (default): Full five-stage review
- **`security`**: Only security stage
- **`performance`**: Only performance stage
- **`quality`**: Only code quality stage

## Step 3: Five-Stage Review

### Stage 1: Spec Compliance
- Do the changes accomplish what they appear to intend?
- Are there incomplete implementations (TODOs, stubs, placeholder returns)?
- Are there missing edge cases that the code's own structure suggests it should handle?
- Do new functions/components get exported and wired into the app?

### Stage 2: Correctness
- Logic errors, off-by-one, null/undefined risks
- Race conditions in async code (especially streams, React state updates)
- Missing cleanup (event listeners, subscriptions, timers, abort controllers)
- Type mismatches or unsafe casts (`as any`, `as unknown as X`)
- State mutations that should be immutable (React state, Redux)

### Stage 3: Security
Quick-scan the diff for:
- Hardcoded secrets, API keys, tokens
- User input flowing unsanitized into dangerous sinks (`eval`, `innerHTML`, `dangerouslySetInnerHTML`, SQL strings)
- Missing auth checks on new endpoints or screens
- Sensitive data in logs (`console.log` with user data, tokens)
- Exposed internal errors to the user

### Stage 4: Code Quality
- Naming: Are new variables/functions/components named clearly?
- Complexity: Functions doing too much? Deeply nested logic?
- Duplication: Same logic repeated across files?
- Error handling: Swallowed errors (empty `catch {}`), `?.` chains silently skipping failures
- Hidden failure patterns:
  - `?? defaultValue` masking real nulls
  - Catch-log-continue (user never sees the failure)
  - Retry exhaustion without notice
  - Fallback chains that silently degrade

### Stage 5: Performance
- Unnecessary re-renders (missing `useMemo`/`useCallback` deps, new objects in render)
- N+1 patterns (loops with await inside, sequential fetches that could be parallel)
- Large data in React state that should be ref
- Missing `AbortController` cleanup in `useEffect` fetches
- Expensive operations inside render path (JSON.parse, regex, array.find in every render)

## Step 4: Confidence Threshold

**Only report issues you are ≥80% confident about.** Attach a confidence score [80-100] to each finding.

### DO NOT flag:
- Pre-existing issues not introduced by this diff
- Code that merely looks unusual but is correct (check the patterns/memory first!)
- Nitpicks a senior engineer would ignore
- Issues that linters/TypeScript already catch
- Patterns explicitly documented in project memory as intentional
- General best-practice sermons not tied to a specific line

### Severity classification:
- **CRITICAL**: Security vulnerability, data loss risk, or blocks core functionality. Must fix before merge.
- **HIGH**: Affects functionality, correctness, or significant quality issue. Should fix.
- **MEDIUM**: Minor quality issue, could cause problems later. Can merge, fix soon.
- **LOW**: Stylistic, minor improvement. Optional.

## Step 5: Output Format

```
## Code Review: [Approve | Approve with Notes | Changes Requested]

**Scope:** N files changed, +X/-Y lines
**Confidence:** [0-100]% overall

---

### Critical Issues
> Only shown if any exist. Each must have confidence ≥80%.

- **[95]** `file/path.ts:42` — **Issue title**
  What's wrong and why it matters. → **Fix:** concrete suggestion.

---

### High Issues
- **[85]** `file/path.ts:78` — **Issue title**
  Explanation. → **Fix:** suggestion.

---

### Medium Issues
- **[82]** `file/path.ts:120` — **Issue title**
  Explanation.

---

### Observations
> Non-blocking notes, patterns noticed, architectural feedback.

- Observation 1
- Observation 2

---

### What Looks Good
> Briefly call out 2-3 things done well. Engineers deserve positive signal too.

- Good thing 1
- Good thing 2
```

## Rules

- **READ-ONLY.** Do not edit, write, or create any files.
- **Be specific.** Always reference `file:line`. Never give vague feedback.
- **Be actionable.** Every issue must have a concrete fix suggestion.
- **Respect project conventions.** If patterns/memory says "this is intentional", don't flag it.
- **No sermons.** Don't lecture about best practices in general. Only flag concrete issues in this diff.
- **Concise.** If the diff is clean, say so in 5 lines. Don't pad the review.
