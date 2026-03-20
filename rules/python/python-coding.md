---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Python Coding Style

## Standards

- Follow **PEP 8** conventions
- Use **type annotations** on all function signatures
- File size: 200–400 lines typical, 800 max
- Functions: <50 lines, no deep nesting (>4 levels)

## Formatting

```bash
ruff format .      # フォーマット
ruff check .       # リント
```

## Immutability

ALWAYS create new objects, NEVER mutate existing ones:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Config:
    n_trials: int
    seed: int

from typing import NamedTuple

class Point(NamedTuple):
    x: float
    y: float
```

## File Paths

Use `pathlib.Path` (not string concatenation):

```python
from pathlib import Path

output_dir = Path("outputs/analysis1_foo")
output_dir.mkdir(parents=True, exist_ok=True)
df.to_csv(output_dir / "analysis1_result.csv", index=False)
```

## Environment Variables

```python
from dotenv import load_dotenv
import os

load_dotenv()
project_id = os.environ["GCP_PROJECT_ID"]
```

## Error Handling

- Handle errors explicitly; never silently swallow
- Use specific exception types over bare `except:`
- Log detailed context server-side, user-friendly messages at boundaries

## Pandas / NumPy Idioms

```python
# GOOD: vectorized operations
df["total"] = df["price"] * df["qty"]
result = df.groupby("user_id")["revenue"].sum()

# BAD: Python loops
for i, row in df.iterrows():
    df.at[i, "total"] = row["price"] * row["qty"]
```

Priority: vectorized > `apply()` > explicit loop

## Protocols (Duck Typing)

```python
from typing import Protocol

class DataLoader(Protocol):
    def load(self, path: str) -> "pd.DataFrame": ...
```

## Code Quality Checklist

Before marking work complete:
- [ ] Type annotations on all function signatures
- [ ] `ruff format` and `ruff check` pass
- [ ] No hardcoded magic numbers (use constants)
- [ ] No mutation (immutable patterns used)
- [ ] Proper error handling
