# /ship

A pre-commit workflow for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that reviews, investigates, fixes, documents, and commits your code — in one command.

```
/ship
```

That's it. Five steps, fully automated, one slash command.

---

## The Idea

Every commit should be reviewed, understood, and documented. But doing all of that manually is slow and breaks your flow. `/ship` chains five focused skills together so you get a senior-engineer-quality commit pipeline without thinking about it.

```
  Review        Look         Fix         Memory       Commit
    |             |            |            |            |
 find issues → understand → fix safely → learn from → document
               the code                  the diff     everything
```

Each step feeds the next. The review finds issues. The investigation gives the fixer context so it doesn't break things. The memory update captures what you learned. The commit describes everything that happened.

No step is wasted.

---

## What Each Skill Does

| Skill | Purpose | Writes Code? |
|-------|---------|:---:|
| `/review-diff` | Five-stage code review (spec, correctness, security, quality, performance). Confidence-scored findings. Only flags issues it's 80%+ sure about. | No |
| `/look` | Senior architect investigation. Traces data flows, maps dependencies, assesses blast radius. Tells you what would break before you touch anything. | No |
| `/fix-review` | Fixes review findings one at a time. Checks blast radius before each fix. Reverts immediately if tests break. No hardcoded patches. | Yes |
| `/update-memory` | Reads the diff and extracts durable knowledge — architecture decisions, gotchas, patterns — into project memory files so future sessions start smarter. | Memory files only |
| `/big-commit` | Creates a comprehensive commit message with categorized changes, technical details, and clear rationale. Asks for approval before committing. | Git only |

Every skill also works standalone. Use `/review-diff` without the full pipeline. Use `/look streaming provider` to investigate a specific area. Mix and match.

---

## The Pipeline

```
/ship
```

1. **Review** — Scans your diff. If everything is clean, skips straight to memory + commit.
2. **Look** — If issues were found, investigates the flagged areas deeply before anyone touches code.
3. **Fix** — Fixes each finding with full context. One at a time. Verify after each. Revert on failure.
4. **Memory** — Captures what changed and why into `.claude/cc10x/` and `MEMORY.md`.
5. **Commit** — Builds a detailed commit message, shows it to you, waits for your OK.

### Skip Steps

```
/ship skip-look     # review → fix → memory → commit (skip investigation)
/ship skip-fix      # review → memory → commit (review only, no auto-fix)
/ship skip-memory   # review → look → fix → commit (skip memory update)
```

### When the Pipeline Stops

- **Critical issues that can't be auto-fixed**: Pipeline stops, tells you what needs manual work.
- **Fix broke tests**: Reverts the fix, stops, explains what happened.
- **Clean review**: Skips look + fix entirely, goes straight to memory + commit.

---

## Install

### Claude Code

```
/plugin install smartari1/ship-workflow
```

Done. You now have all six commands available:

```
/ship-workflow:ship
/ship-workflow:review-diff
/ship-workflow:look
/ship-workflow:fix-review
/ship-workflow:update-memory
/ship-workflow:big-commit
```

### Manual Setup (No Plugin Manager)

If you prefer to add the skills directly to your project:

1. Clone this repo:
   ```bash
   git clone https://github.com/smartari1/ship-workflow.git
   ```

2. Copy the skills into your project's `.claude/skills/` directory:
   ```bash
   cp -r ship-workflow/skills/* your-project/.claude/skills/
   ```

3. That's it. The skills are available as `/ship`, `/review-diff`, `/look`, etc.

### Global Install (All Projects)

To make the skills available in every project on your machine:

```bash
cp -r ship-workflow/skills/* ~/.claude/skills/
```

---

## Memory Files

The `/update-memory` and `/ship` pipeline look for memory files in two locations:

- **`.claude/cc10x/`** — Project-level memory (activeContext.md, patterns.md, progress.md)
- **`~/.claude/projects/*/memory/MEMORY.md`** — Claude Code's auto-memory

If these files don't exist, `/update-memory` creates them. If they do exist, it surgically updates them — removing outdated entries, adding new patterns, keeping everything concise.

### What Gets Saved

- Architecture decisions and their rationale
- Patterns that would take 10+ minutes to rediscover from code
- Gotchas that caused bugs or confusion
- Structural changes (new directories, provider hierarchies, registries)

### What Doesn't Get Saved

- Test counts (they change constantly)
- Line-by-line change descriptions (that's git log)
- Implementation details obvious from reading the code

---

## Design Principles

**One thing per skill.** Each skill does exactly one job. No skill tries to be clever about doing two things.

**Order matters.** Review before fix (know what's wrong). Look before fix (understand the code). Memory after fix (capture the final state). Commit last (document everything).

**Fail safe.** Fix breaks tests? Revert immediately. Finding too risky? Skip it and tell the user. Never push without asking.

**Context flows forward.** Each step's output is in the conversation when the next step runs. The fixer knows what the reviewer found AND what the investigator learned about blast radius. No information lost between steps.

**Confidence over volume.** The reviewer only flags issues it's 80%+ confident about. No vague "you might want to consider..." feedback. Every finding has a severity, a file:line, and a concrete fix suggestion.

---

## License

MIT
