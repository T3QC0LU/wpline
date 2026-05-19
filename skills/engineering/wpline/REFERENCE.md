# ENNA Pipeline Review — Patterns from VCSC files

## Real errors seen and fixed in this project

### ruff W291 — trailing whitespace
```python
# BAD (spaces after comma on blank-ish lines)
        params = [
            "off",
            "on",
        ]  
#              ^ trailing space

# GOOD
        params = [
            "off",
            "on",
        ]
```

### import_check — unused import
```python
# BAD — imported but never referenced
from enna.core import exceptions                         # W0611
from enna_porsche_core_lib.utilities.adb.gateway import helper  # never called

# GOOD — remove it entirely
```

### import_check — unsorted imports
```python
# BAD — hcp5.helper before adb.helper (hcp5 > adb alphabetically? No: 'adb.h' < 'adb.hcp5')
from enna_porsche_core_lib.utilities.adb.hcp5 import helper as hcp5_helper
from enna_porsche_core_lib.utilities.adb import helper as adb_helper

# GOOD
from enna_porsche_core_lib.utilities.adb import helper as adb_helper
from enna_porsche_core_lib.utilities.adb.hcp5 import helper as hcp5_helper
```

### pylint — try block too long (max 7 statements)
```python
# BAD
try:
    f = open(path)          # 1
    line1 = f.readline()   # 2
    line2 = f.readline()   # 3
    line3 = f.readline()   # 4
    f.close()              # 5
    result = process(line1, line2, line3)  # 6
    validate(result)       # 7
    store(result)          # 8  ← too many!
except OSError:
    ...

# GOOD — read outside, process outside
try:
    with open(path) as f:
        content = f.read()
except OSError:
    return False
result = process(content)
validate(result)
store(result)
```

### pylint — OSError redundancy
```python
# BAD
except (FileNotFoundError, IOError) as e:

# GOOD — FileNotFoundError and IOError are both subclasses of OSError
except OSError as e:
```

### pylint — dead code after return in loop
```python
# BAD
for keyword in keywords:
    if keyword in content:
        return True         # ← early return
    return False            # ← DEAD: only runs if keyword IN content + return True above
                            #         actually runs on first iteration always!

# GOOD — collect missing, return after loop
missing = [kw for kw in keywords if kw not in content]
return len(missing) == 0
```

### pylint — invalid module-level disable
```python
# BAD — put at top of file, doesn't suppress function-level warnings
# pylint: disable=too-many-branches
# pylint: disable=too-many-return-statements

# GOOD option 1: remove entirely (PLR0911/PLR0912 are ruff-ignored project-wide)
# GOOD option 2: put inline on the def line if you truly need it
def my_func():  # pylint: disable=too-many-branches
    ...
```

### pydocstyle — Chinese in docstring
```python
# BAD — spell checker fires on CJK characters
def _open_connect_services_toggle(self) -> None:
    """打开连接服务开关."""

# GOOD
def _open_connect_services_toggle(self) -> None:
    """Open the ConnectServices privacy toggle."""
```

### pydocstyle — missing param/return docs
```python
# BAD
def set_timeout(self, seconds: int) -> bool:
    """Set the timeout."""

# GOOD
def set_timeout(self, seconds: int) -> bool:
    """Set the timeout.

    :param seconds: timeout duration in seconds
    :return: True if timeout was set successfully
    """
```

## Import group ordering convention

```python
# stdlib
import os
import time

# third-party
import pytest

# enna core
from enna.core import ...
from enna_porsche_core_lib.utilities.adb import helper as adb_helper
from enna_porsche_core_lib.utilities.adb.hcp5 import helper as hcp5_helper
from enna_porsche_core_lib.utilities.contexts import launcher
from enna_porsche_core_lib.utilities.menu_navigation import exceptions as menu_nav_exceptions

# project-local
from enna_tc_porsche_hcp.stimulations.cn.vcsc import ...
```

## nox local commands (if nox is installed and venv set up)

```bash
# Single session on specific file:
nox -s ruff -- enna_tc_porsche_hcp/stimulations/cn/vcsc/hcp3_notify_sl_job.py

# All lint sessions (skip coverage):
nox -s ruff import_check pycodestyle pydocstyle pylint

# Quick ruff check without nox:
ruff check enna_tc_porsche_hcp/stimulations/cn/vcsc/
```
