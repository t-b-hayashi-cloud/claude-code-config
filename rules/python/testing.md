---
paths:
  - "**/*.py"
---
# Python Testing

## Minimum Coverage: 80%

```bash
pytest --cov=src --cov-report=term-missing
```

## Test-Driven Development (mandatory)

1. Write test first (RED) — `pytest` で失敗を確認
2. Implement minimal code (GREEN) — `pytest` で通過を確認
3. Refactor (IMPROVE)
4. Verify 80%+ coverage

## pytest 構成

```python
import pytest
import pandas as pd

@pytest.mark.unit
def test_calculate_revenue():
    result = calculate_revenue(price=100, qty=3)
    assert result == 300

@pytest.mark.integration
def test_bigquery_load(tmp_path):
    # BigQueryモックを使用
    ...
```

## DataFrameテストパターン

```python
import pandas as pd
from pandas.testing import assert_frame_equal

def test_aggregate():
    input_df = pd.DataFrame({"user_id": ["a", "a"], "revenue": [100, 200]})
    expected = pd.DataFrame({"user_id": ["a"], "revenue": [300]})
    result = aggregate_by_user(input_df)
    assert_frame_equal(result.reset_index(drop=True), expected)
```

## BigQueryモック

```python
from unittest.mock import MagicMock, patch

def test_fetch_data():
    mock_df = pd.DataFrame({"user_id": ["u1"], "revenue": [500.0]})
    with patch("google.cloud.bigquery.Client") as MockClient:
        MockClient.return_value.query.return_value.to_dataframe.return_value = mock_df
        result = fetch_data()
    assert len(result) == 1
```

## tmp_path の使用

```python
def test_save_csv(tmp_path):
    df = pd.DataFrame({"a": [1, 2]})
    path = tmp_path / "output.csv"
    save_csv(df, path)
    loaded = pd.read_csv(path)
    assert len(loaded) == 2
```

## Troubleshooting

1. Use **tdd-guide** agent
2. Check test isolation (avoid shared state)
3. Fix implementation, not tests (unless tests are wrong)
