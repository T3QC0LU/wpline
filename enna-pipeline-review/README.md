# wpline

My GitHub Copilot CLI skill for enforcing the ENNA Porsche HCP Pipeline lint rules — so every `.py` push passes CI on the first try.

## Quickstart (30-second setup)

**Option 1 — npx (recommended):**

```bash
npx skills@latest add T3QC0LU/wpline
```

Pick `enna-pipeline-review` and select which agent to install it on.

**Option 2 — manual link script:**

```bash
git clone https://github.com/T3QC0LU/wpline.git ~/wpline/enna-pipeline-review
cd ~/wpline/enna-pipeline-review
bash scripts/link-skills.sh
```

This symlinks the skill into `~/.agents/skills/` so the Copilot CLI agent picks it up automatically.

## Usage

Once installed, trigger the skill by saying any of:

- `review before push`
- `check Pipeline`
- `lint check`
- or when editing `.py` files in `enna-shared-testcases` or `enna_porsche_core_lib`

## Skills

### Engineering

- **[enna-pipeline-review](./skills/engineering/enna-pipeline-review/SKILL.md)** — Review Python code changes against the ENNA Porsche HCP Pipeline lint rules (ruff, pylint, import_check, pycodestyle, pydocstyle) before committing or pushing.

## Why This Exists

The ENNA Porsche HCP Pipeline runs 5 lint sessions on every push. Failing them after push wastes CI time and blocks the team. This skill runs the same checks locally — in the right order — so you catch issues before they hit the pipeline.
