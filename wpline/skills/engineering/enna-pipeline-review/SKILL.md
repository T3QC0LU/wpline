---
name: enna-pipeline-review
description: Review Python code changes against the ENNA Porsche HCP Pipeline lint rules (ruff, pylint, import_check, pycodestyle, pydocstyle) before committing or pushing. Use when writing or modifying .py files in enna-shared-testcases or enna_porsche_core_lib, or when the user says "review before push", "check Pipeline", "lint check", or wants to avoid CI failures.
---

# ENNA Pipeline Review

**Goal**: Every code change MUST pass all 5 Pipeline checks before `git push`.

## Workflow

### Step 0 — Identify files

```bash
git diff --name-only HEAD | grep '\.py$'
```

Use this list as the target. Do NOT ask the user which files to check.

### Step 1 — Static analysis + auto-fix

For each file, apply fixes directly without asking:
- Remove trailing whitespace
- Sort imports alphabetically within each group
- Remove unused imports
- Add missing docstrings (English only)
- Replace `(FileNotFoundError, IOError)` → `OSError`
- Remove dead code after `return`/`break`/`raise`
- Remove module-level `pylint: disable=too-many-*` (they don't work there)
- Ensure try blocks have ≤ 7 statements

### Step 2 — Run nox if available

```bash
# Check if nox is available:
which nox && nox -s ruff import_check pycodestyle pydocstyle pylint -- <files>
```

If nox is not installed, skip this step and rely on static analysis only.

### Step 3 — Final report

Output a summary table and nothing else verbose:

| Session | Status | Notes |
|---------|--------|-------|
| ruff | ✅ / ❌ | brief note |
| import_check | ✅ / ❌ | brief note |
| pycodestyle | ✅ / ❌ | brief note |
| pydocstyle | ✅ / ❌ | brief note |
| pylint | ✅ / ❌ | brief note |

**Changes made**: one-line summary of what was fixed.

Only declare "ready to push" when ALL 5 are ✅.

---

## Rules — run in this order

### 1. ruff  (`ruff.toml` config, line-length=259)

Scan every changed file. Key rules that fire most often:

| Code | Issue | Fix |
|------|-------|-----|
| W291 | Trailing whitespace | Remove spaces at end of lines |
| W293 | Whitespace on blank line | Remove all whitespace from blank lines |
| E302 | Expected 2 blank lines before function/class | Add blank lines |
| F401 | Unused import | Remove (ruff ignores F401, but import_check catches it — see step 2) |
| D1xx | Missing docstring | Add docstring to every public function/class/method |
| D205 | Blank line required after summary | Fix docstring format |
| D400 | First line should end with period | Ignored in this project (D400 is in ruff ignore list) |
| B0xx | Bugbear issues | Fix as instructed |
| PLCxxx | pylint compat rules | See pylint section |

**Ruff-ignored rules** (do NOT add `# noqa` or `# ruff: disable` for these — they're project-wide ignored):
`B009 B010 B019 B024 B904 D203 D213 D301 E402 PLR0911 PLR0912 PLR0913 PLR0915 PLR2004 PLW0603 D206 W191 D400 PLW1510 E701 PLR0402 F401 E501 PLW2901 B007`

**Ruff excluded dirs**: `ide_tools`, `enna_porsche_core_lib/docs`, `enna_tc_porsche_hcp/stimulations/ece/tcg`, `enna_tc_porsche_hcp/stimulations/non_productive`, `helper_files`, `enna_tc_porsche_hcp/pipeline_ymls/pipeline_scripts`

---

### 2. import_check

Rules enforced on `from X import Y` statements:
- **No unused imports** — remove any import not referenced in the file body
- **Alphabetically sorted** within each import group — within a block of `from enna_porsche_core_lib.utilities...` lines, sort alphabetically
- **No `# type: ignore`** on import lines unless absolutely necessary

Common mistakes:
```python
# BAD — unused
from enna_porsche_core_lib.utilities.adb.gateway import helper  # never used below

# BAD — unsorted
from enna_porsche_core_lib.utilities.adb.hcp5 import helper
from enna_porsche_core_lib.utilities.adb import helper  # 'adb.helper' should come before 'adb.hcp5.helper'

# GOOD — sorted
from enna_porsche_core_lib.utilities.adb import helper
from enna_porsche_core_lib.utilities.adb.hcp5 import helper as hcp5_helper
```

---

### 3. pycodestyle (`--max-line-length=250`, ignore `W191 E117 E122 E126 E127 E128 E402`)

- Max line length: **250 chars**
- No trailing whitespace (W291/W293)
- Blank lines: 2 before top-level defs, 1 before method defs inside class

---

### 4. pydocstyle (ignore `D203 D213 D301`)

Every `def` and `class` needs a docstring. Format rules:
```python
def my_func(self, arg: str) -> bool:
    """Short one-line summary.   ← ends with period

    :param arg: description of arg
    :return: description of return value
    """
```

- Parameters documented with `:param name: description`
- Return documented with `:return: description`
- Raises documented with `:raises ExceptionType: description`
- No blank line before class docstring (D211 style, not D203)
- Summary on first line (D212 style, not D213)

---

### 5. pylint (`--max-line-length=306`)

Most `too-many-*` rules are ruff-ignored. The rules that still fire:

| Code | Issue | Fix |
|------|-------|-----|
| W0611 | Unused import | Remove import |
| W0612 | Unused variable | Remove or rename to `_` |
| E1101 | Module has no member | Fix or add `# pylint: disable=no-member` with justification |
| R0201 | Method could be a function | Add `@staticmethod` or keep as-is with disable |
| C0301 | Line too long >306 | Split line |
| W0707 | Raise without from | Use `raise X from Y` or `raise X from None` |
| W0718 | Broad exception | Catch specific exceptions |
| R1705 | Unnecessary `elif` after `return` | Remove `elif` |
| W0105 | String statement with no effect | Remove bare string literals |
| C0123 | Use `isinstance()` | Replace `type(x) ==` with `isinstance(x, ...)` |
| **try block** | Too many statements in try (max 7) | Move non-risky code outside the try block |
| **OSError** | `(FileNotFoundError, IOError)` is redundant | Use just `OSError` |
| **Dead code** | Code after `return`/`break`/`raise` | Remove unreachable lines |
| **Chinese chars** | Spell checker fires on CJK | Replace Chinese comments/docstrings with English |

**Invalid pylint disable patterns** (these do NOT work at module level):
```python
# pylint: disable=too-many-branches   ← module-level, INVALID for function-level issues
# pylint: disable=too-many-return-statements  ← same
```
These must go on the `def` line or inside the function, or be removed entirely (since PLR0911/PLR0912 are already ruff-ignored).

---

## Auto-fix checklist (agent applies these directly)

For every `.py` file from `git diff`:

- [ ] No trailing whitespace on any line
- [ ] All imports used — remove any not referenced in file body
- [ ] Imports sorted alphabetically within each group
- [ ] Every `def`/`class` has a docstring in English
- [ ] Docstring params/return documented with `:param`/`:return:`
- [ ] No Chinese characters in docstrings or comments (replace with English)
- [ ] `OSError` instead of `(FileNotFoundError, IOError)`
- [ ] No dead code after `return`/`break`/`raise`
- [ ] Try blocks have ≤ 7 statements
- [ ] No module-level `pylint: disable=too-many-*`
- [ ] Line length ≤ 250 chars (pycodestyle) / ≤ 306 chars (pylint hard limit)

See [REFERENCE.md](REFERENCE.md) for real error patterns from VCSC files.
