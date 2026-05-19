> 🔍 Generated in **learn** mode by [read-repo](https://github.com/T3QC0LU/read-repo). Re-run with "analyze this repo for someone taking over" to switch modes.

# wpline — READREPO

## Tech Stack

| | |
|---|---|
| Platform | GitHub Copilot CLI agent (local) |
| Languages | Markdown (6), Bash (1), JSON (1) |
| Installer | `npx skills@latest` protocol |

## Features

- Pre-push Python lint review across 5 Pipeline sessions (ruff, import_check, pycodestyle, pydocstyle, pylint)
- Auto-detects changed files via `git diff`
- Auto-fixes issues directly, no prompting
- Outputs ✅/❌ result table + change summary
- One-command install via `npx` or manual symlink script

## Architecture

**Pattern**: Prompt-as-config — Markdown files are the instructions, the agent is the runtime.

```
User triggers keyword
  → Agent reads SKILL.md
  → git diff → changed .py files
  → Static analysis + nox (if available)
  → Auto-fix
  → ✅/❌ report
```

## Folder Map

```
/.claude-plugin/plugin.json   npx skills@latest entry point
/skills/engineering/
  enna-pipeline-review/
    SKILL.md                  Agent operation manual (core)
    REFERENCE.md              Real error case library — add to this as you encounter new patterns
/scripts/link-skills.sh       Manual symlink installer (Mac/Linux)
/docs/adr/                    Architecture decision records
/CONTEXT.md                   Domain language glossary
/CLAUDE.md                    Skills organisation rules
```

## Installation

```bash
# Recommended
npx skills@latest add T3QC0LU/wpline

# Manual (Mac/Linux)
git clone https://github.com/T3QC0LU/wpline.git
bash wpline/scripts/link-skills.sh
```
