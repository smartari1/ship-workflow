---
name: big-commit
description: Create a highly detailed git commit with comprehensive description of all changes
allowed-tools: Bash, Read, Grep, Glob
---

# Big Commit: Detailed Change Documentation

You are creating a **comprehensive, production-quality git commit** that fully documents every significant change in the working tree. This is not a one-liner — it's a detailed commit that serves as permanent project history.

## Step 0: Safety Checks

Before anything:
1. Verify you are NOT on `main` or `master` by running `git branch --show-current`
2. If on main/master, **STOP** and tell the user to create a branch first.

## Step 1: Understand the Changes

Read the full picture:

1. Run `git status --short` to see all changed files
2. Run `git diff --stat HEAD` to see change volume
3. Run `git ls-files --others --exclude-standard | head -30` to see untracked files
4. Run `git log --oneline -5` to see recent commit style
5. Run `git diff HEAD` to read the full diff. For large diffs, read per-file.

Understand every change — you need to describe them all.

## Step 2: Classify All Changes

Group every change into categories. A single diff may span multiple categories:

| Category | Prefix | Example |
|----------|--------|---------|
| New feature | `feat` | New component, new API, new hook |
| Enhancement | `enhance` | Improve existing feature |
| Bug fix | `fix` | Correct broken behavior |
| Refactor | `refactor` | Restructure without behavior change |
| Performance | `perf` | Optimization |
| Types | `types` | Type definitions, interfaces |
| Tests | `test` | New or updated tests |
| Chore | `chore` | Config, dependencies, tooling |

Pick the **primary** category for the commit title prefix. If changes span many, use the dominant one.

## Step 3: Build the Commit Message

Structure:

```
<prefix>: <concise title under 70 chars>

<2-3 sentence summary of the overall change and WHY it was made>

## What Changed

### <Category 1 heading>
- <file or component>: <what changed and why>
- <file or component>: <what changed and why>

### <Category 2 heading>
- ...

## Technical Details
- <Key architectural decisions or patterns introduced>
- <Breaking changes or migration notes if any>
- <Dependencies added/removed if any>

## Files Changed
- <N> files modified, <N> files added, <N> files deleted

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

### Rules for the commit message:
- **Title**: Imperative mood, under 70 chars, lowercase after prefix (`feat: add XHR streaming fallback for React Native`)
- **Summary**: Explain the WHY, not just the what. What problem does this solve?
- **What Changed**: Group by logical area (not by file). Each bullet is `component/area: what + why`. Be specific — mention function names, type names, patterns.
- **Technical Details**: Only include if there are architectural decisions, new conventions, or breaking changes worth documenting.
- **No test counts** — they change constantly.
- **No filler** — every line should carry information. Skip categories with nothing meaningful.

## Step 4: Stage and Commit

1. Show the user the full commit message and the list of files to be staged
2. **Ask for explicit approval** before committing — NEVER commit without user confirmation
3. Stage specific files (prefer `git add <file>` over `git add -A` to avoid secrets/binaries)
4. Create the commit using a HEREDOC:

```bash
git commit -m "$(cat <<'EOF'
<commit message here>
EOF
)"
```

5. Run `git status` after to verify success

## Step 5: Do NOT Push

Never push to remote. Just create the local commit and tell the user it's ready.
