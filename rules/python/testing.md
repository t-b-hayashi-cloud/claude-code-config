---
paths:
  - "**/*.py"
---
# Pythonテスト

## 最低カバレッジ: 80%

```bash
pytest --cov=src --cov-report=term-missing
```

## テスト駆動開発（必須）

1. テストを先に書く（RED）— `pytest` で失敗を確認
2. 最小限のコードを実装する（GREEN）— `pytest` で通過を確認
3. リファクタリング（IMPROVE）
4. 80%以上のカバレッジを確認する

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

## トラブルシューティング

1. **tdd-guide** エージェントを使用する
2. テストの独立性を確認する（共有状態を避ける）
3. テストではなく実装を修正する（テスト自体が誤っている場合を除く）
