---
name: update-memory
description: Analyze the current git diff and update project memory files with new patterns, architecture changes, and progress
argument-hint: "[scope: all | ship-memory | claude-memory]"
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# Update Memory from Diff

You are a senior engineer extracting durable knowledge from code changes. Your job is to read the current diff, understand what changed architecturally, and update the project memory files so future sessions have accurate context.

## Step 1: Gather Context

Discover the project root and memory file locations:

```bash
PROJECT_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
```

Read the current diff and locate existing memory files:

1. Run `git diff --stat` and `git diff --staged --stat` to see what changed
2. Look for memory files in these standard locations (read whichever exist):
   - `$PROJECT_ROOT/.claude/ship-memory/activeContext.md`
   - `$PROJECT_ROOT/.claude/ship-memory/patterns.md`
   - `$PROJECT_ROOT/.claude/ship-memory/progress.md`
   - Claude auto-memory MEMORY.md (search `~/.claude/projects/` for a directory matching the project name)

If ship-memory files don't exist, create them with appropriate headers:
- `activeContext.md`: `<!-- Ship-Memory Session Memory - Do not delete this file -->`
- `patterns.md`: `<!-- Ship-Memory Patterns - Do not delete this file -->`
- `progress.md`: `<!-- Ship-Memory Progress - Do not delete this file -->`

## Step 2: Read the Full Diff

Run `git diff` and `git diff --staged` (separately if needed) to read the actual code changes. For large diffs, read per-file diffs for the most architecturally significant files first. Prioritize:
1. New files (new patterns, new components)
2. Provider/hook changes (state management shifts)
3. Service layer changes (API, streaming, data flow)
4. Type changes (new interfaces, changed contracts)
5. Test changes (reveal intent and edge cases)

Skip purely cosmetic changes (formatting, import reordering).

## Step 3: Determine Scope

The user may pass a scope argument: `$ARGUMENTS`

- **`all`** (default if no argument): Update all memory targets
- **`ship-memory`**: Only update `.claude/ship-memory/` files
- **`claude-memory`**: Only update the Claude MEMORY.md

## Step 4: Classify What Changed

For each significant change, classify it as:

| Category | Goes into | Example |
|----------|-----------|---------|
| **Architecture decision** | patterns.md + MEMORY.md | "SSE now uses XHR on RN, fetch on web" |
| **Gotcha / trap** | patterns.md + MEMORY.md | "data_snippet rows are string[][], not number[][]" |
| **New component / file** | activeContext.md | "Added FormatWebSearch formatter" |
| **Completed work** | progress.md + activeContext.md | "Phase 3 complete: 8 formatters" |
| **Deleted / replaced** | Remove from all | Old patterns for deleted code |
| **Convention** | patterns.md | "Formatters return ToolFormatterResult" |

## Step 5: Update Rules

Follow these rules strictly:

### What to write
- **Decisions and their rationale** ("XHR for RN because fetch ReadableStream unreliable on iOS")
- **Patterns that would take 10+ minutes to rediscover** from code
- **Gotchas that caused bugs or confusion** during this session
- **Structural changes** (new directories, new provider hierarchy, new registries)
- **Key type signatures** that define contracts between modules

### What NOT to write
- Line-by-line change descriptions (that's what git log is for)
- Test counts (they change constantly)
- Temporary debugging state
- Implementation details that are obvious from reading the code
- Anything already accurately captured in existing memory

### How to write
- Be concise. One bullet per concept. 1-2 sentences max.
- Use `code formatting` for file paths, function names, type names
- Group related items under clear headings
- **Remove outdated entries** — don't just append. If a pattern was replaced, delete the old one.
- Preserve the `<!-- Ship-Memory ... -->` comment headers in ship-memory files
- Keep MEMORY.md under 120 lines (it's loaded into every system prompt)
- Keep activeContext.md "Recent Changes" to last ~10 entries (trim oldest)
- Deduplicate: if the same knowledge exists in multiple files, keep the canonical version and remove redundant copies

## Step 6: Apply Updates

Use the Edit tool to surgically update existing files. Only use Write for complete rewrites when >50% of the file content is changing.

After updating, briefly summarize what you changed and why for the user.
