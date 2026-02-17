---
name: init
description: Initialize ship-workflow memory files for a project. Detects existing memory, creates missing files, and writes a config so all skills know where to find them.
argument-hint: "[force — re-run even if already initialized]"
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# Init: Ship Workflow Setup

First-time setup for the ship-workflow pipeline. Detects existing project memory files, creates any that are missing, and writes a config file so all skills know where everything lives.

**This skill is idempotent** — safe to run multiple times. It never overwrites existing content.

## Step 1: Detect Project Root

```bash
PROJECT_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
```

Check if already initialized:
- Look for `$PROJECT_ROOT/.claude/ship-config.json`
- If it exists AND `$ARGUMENTS` is not `force`, report "Already initialized" with a summary of what's configured, and stop.

## Step 2: Scan for Existing Memory Files

Search for memory files the project may already have. Check ALL of these locations:

### CC10x Memory (project-level)
- `$PROJECT_ROOT/.claude/cc10x/activeContext.md`
- `$PROJECT_ROOT/.claude/cc10x/patterns.md`
- `$PROJECT_ROOT/.claude/cc10x/progress.md`

### Claude Auto-Memory (user-level)
Search `~/.claude/projects/` for a directory whose name matches or contains the project directory name. Inside it, look for `memory/MEMORY.md`.

```bash
# Find the auto-memory directory for this project
PROJECT_DIR=$(basename "$PROJECT_ROOT")
find ~/.claude/projects/ -maxdepth 1 -type d -name "*${PROJECT_DIR}*" 2>/dev/null
```

### Other common patterns
- `$PROJECT_ROOT/.claude/memory/`
- `$PROJECT_ROOT/.memory/`
- `$PROJECT_ROOT/docs/architecture/`

Record which files exist and which are missing.

## Step 3: Create Missing Memory Files

For each missing file, create it with useful starter content. **Never overwrite existing files.**

### If `.claude/cc10x/` directory is missing, create all three:

**activeContext.md:**
```markdown
<!-- CC10X Session Memory - Do not delete this file -->
# Active Context

## Current Focus
- [Ship-workflow initialized — ready for first /ship run]

## Recent Changes
- [Pending first session]
```

**patterns.md:**
```markdown
<!-- CC10X Patterns - Do not delete this file -->
# Patterns

## Architecture
- [Run /look to investigate and document your project's architecture]

## Common Gotchas
- [Patterns will be captured here as you work — gotchas, conventions, and decisions that take time to rediscover]
```

**progress.md:**
```markdown
<!-- CC10X Progress - Do not delete this file -->
# Progress

## Current Workflow
- Ship-workflow initialized

## Completed
- [x] Ship-workflow setup
```

### If Claude auto-memory MEMORY.md is missing:

Find or create the auto-memory directory:
```bash
PROJECT_DIR=$(basename "$PROJECT_ROOT")
# Look for existing
MEMORY_DIR=$(find ~/.claude/projects/ -maxdepth 1 -type d -name "*${PROJECT_DIR}*" 2>/dev/null | head -1)
# If not found, we can't create it (Claude Code manages this directory)
```

If the directory exists but MEMORY.md doesn't, create a starter:
```markdown
# [Project Name] - Key Patterns

## Architecture
- [Will be populated by /update-memory after first /ship run]

## Project Structure
- [Will be populated automatically]
```

If the auto-memory directory doesn't exist at all, skip — Claude Code will create it on its own. Note this in the config.

## Step 4: Analyze the Project

Do a quick scan to seed the memory files with useful starting context (only if files were just created, not if they already existed):

1. Read `package.json` or equivalent to understand the project type (React, Node, Python, etc.)
2. Run `git log --oneline -10` to see recent activity
3. Run a quick directory scan: `ls -la src/` or equivalent
4. Check for existing config files that reveal the stack: `tsconfig.json`, `next.config.js`, `expo`, `vite`, etc.

Write a brief **Architecture** section into the newly created `patterns.md` and `MEMORY.md` with:
- Project type and main framework
- Key directories and their purpose
- Entry points (if obvious)

Keep it under 10 lines. The `/update-memory` skill will flesh it out properly on the first `/ship` run.

## Step 5: Write Config

Create `$PROJECT_ROOT/.claude/ship-config.json`:

```json
{
  "version": "1.1.0",
  "initialized": "2026-02-17T10:00:00Z",
  "projectRoot": "/path/to/project",
  "memory": {
    "cc10x": {
      "activeContext": ".claude/cc10x/activeContext.md",
      "patterns": ".claude/cc10x/patterns.md",
      "progress": ".claude/cc10x/progress.md"
    },
    "claudeAutoMemory": "~/.claude/projects/<dir>/memory/MEMORY.md"
  },
  "status": "ready"
}
```

Use relative paths for cc10x (project-level) and absolute for Claude auto-memory (user-level). Set `claudeAutoMemory` to `null` if the directory doesn't exist yet.

## Step 6: Report

Output a clean summary:

```
## Ship Workflow Initialized

**Project:** [name] ([framework])
**Root:** [path]

### Memory Files
| File | Status |
|------|--------|
| `.claude/cc10x/activeContext.md` | Created / Already existed |
| `.claude/cc10x/patterns.md` | Created / Already existed |
| `.claude/cc10x/progress.md` | Created / Already existed |
| `MEMORY.md` (auto-memory) | Created / Already existed / Not available |

### Next Steps
- Run `/ship` to execute the full pipeline on your current changes
- Run `/look [area]` to investigate any part of the codebase
- Run `/review-diff` to review your current diff
```

## Rules

- **Never overwrite existing files.** Only create what's missing.
- **Never delete anything.** Init is additive only.
- **Keep starter content minimal.** Just enough structure for the pipeline to work. The real content comes from `/update-memory`.
- **Idempotent.** Running init twice is safe and fast (skips if config exists, unless `force`).
