# wpline — Domain Context

A single-skill repo that enforces the ENNA Porsche HCP Pipeline lint rules before every `git push`.

## Language

**Pipeline**: The CI/CD lint pipeline run by nox that executes ruff, import_check, pycodestyle, pydocstyle, and pylint on every `.py` file.

**Lint session**: One of the five nox sessions — `ruff`, `import_check`, `pycodestyle`, `pydocstyle`, `pylint` — that must all pass before pushing.

**Pre-push review**: Running all five lint sessions locally on every modified `.py` file before `git push`.
_Avoid_: "check", "validation" (use "pre-push review" or "lint session")

## Scope

This skill applies only to `.py` files in:
- `enna-shared-testcases`
- `enna_porsche_core_lib`
