---
name: tdd-workflow
description: 新機能の作成、バグ修正、コードのリファクタリング時にこのスキルを使用します。ユニット・統合テストを含む80%以上のカバレッジでテスト駆動開発を強制します。Pythonプロジェクト（pytest）向け。
---

# テスト駆動開発ワークフロー（Python/pytest）

このスキルは、すべてのコード開発が包括的なテストカバレッジを備えたTDDの原則に従うことを保証します。

## 有効化するタイミング

- 新機能や関数の作成
- バグや問題の修正
- 既存コードのリファクタリング
- 新しいユーティリティの作成

## コア原則

### 1. コードの前にテスト
常にテストを最初に書き、次にテストに合格するコードを実装します。

### 2. カバレッジ要件
- 最低80%のカバレッジ（ユニット + 統合）
- すべてのエッジケースをカバー
- エラーシナリオのテスト

### 3. テストタイプ

#### ユニットテスト（必須）
```python
import pytest
import pandas as pd
import numpy as np

# テスト対象モジュールをインポート（CLAUDE.mdのsrc/構造に従う）
from src.mypackage.utils import normalize_column

class TestNormalizeColumn:
    def test_basic_normalization(self):
        series = pd.Series([0.0, 1.0, 2.0])
        result = normalize_column(series)
        assert result.min() == pytest.approx(0.0)
        assert result.max() == pytest.approx(1.0)

    def test_constant_series_raises(self):
        """定数列は正規化できないのでエラー"""
        series = pd.Series([1.0, 1.0, 1.0])
        with pytest.raises(ValueError, match="定数列"):
            normalize_column(series)

    def test_nan_handling(self):
        series = pd.Series([1.0, None, 3.0])
        result = normalize_column(series)
        assert result.isna().sum() == 1
```

#### 統合テスト（必須）
```python
from pathlib import Path

# テスト対象モジュールをインポート（CLAUDE.mdのsrc/構造に従う）
from src.mypackage.pipeline import run_preprocessing

@pytest.fixture
def sample_csv(tmp_path):
    df = pd.DataFrame({
        "user_id": [1, 2, 3],
        "amount": [100.0, 200.0, 300.0]
    })
    path = tmp_path / "input.csv"
    df.to_csv(path, index=False)
    return path

def test_pipeline_produces_output(sample_csv, tmp_path):
    output_dir = tmp_path / "output"
    run_preprocessing(input_path=sample_csv, output_dir=output_dir)
    assert (output_dir / "processed.csv").exists()
```

## TDDワークフローステップ

### ステップ1：最初にテストを書く（RED）
```python
# tests/test_new_feature.py を作成
def test_my_new_function():
    result = my_new_function(input_data)
    assert result == expected_output
```

### ステップ2：テストを実行（失敗することを確認）
```bash
uv run pytest tests/test_new_feature.py -v
# FAILED - まだ実装していない
```

### ステップ3：最小限の実装を書く（GREEN）
```python
# src/ に実装（CLAUDE.mdのディレクトリ構造に従う）
def my_new_function(data):
    # 最小限の実装
    return processed_data
```

### ステップ4：テストを実行（合格することを確認）
```bash
uv run pytest tests/test_new_feature.py -v
# PASSED
```

### ステップ5：リファクタリング（IMPROVE）
```bash
uv run pytest tests/ -v  # すべてのテストが通ることを確認しながらリファクタ
```

### ステップ6：カバレッジを確認
```bash
uv run pytest --cov=src --cov-report=term-missing
# Stmts  Miss  Cover
# 最低80%を達成していることを確認
```

## 外部サービスのモック

### BigQueryをモック
```python
from unittest.mock import patch, MagicMock

@patch("src.mypackage.data.BigQueryClient")
def test_fetch_data(mock_bq_class):
    mock_instance = mock_bq_class.return_value
    mock_instance.query.return_value = pd.DataFrame({
        "user_id": [1, 2],
        "amount": [100.0, 200.0]
    })
    result = fetch_data("SELECT * FROM table")
    assert len(result) == 2
    mock_instance.query.assert_called_once()
```

### ファイルI/Oをモック
```python
def test_save_results(tmp_path):
    """tmp_pathフィクスチャで一時ディレクトリを使用"""
    output_path = tmp_path / "results.csv"
    save_results(df, output_path)
    assert output_path.exists()
    saved = pd.read_csv(output_path)
    assert len(saved) == len(df)
```

## pytest設定

```toml
# pyproject.toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "--cov=src --cov-report=term-missing"

[tool.coverage.run]
omit = ["tests/*", "*/conftest.py"]

[tool.coverage.report]
fail_under = 80
```

## テスト品質チェックリスト

テストを完了としてマークする前に:
- [ ] すべての公開関数にユニットテストがある
- [ ] エッジケースがカバーされている（NaN、空DataFrame、負の値）
- [ ] エラーパスがテストされている
- [ ] 外部依存関係（BigQuery、ファイルI/O）にモックが使用されている
- [ ] テストが独立している（共有状態なし）
- [ ] カバレッジが80%以上

## 避けるべきアンチパターン

### ❌ 実装の詳細をテスト
```python
# 内部状態をテストしない
assert model._n_estimators == 100
```

### ✅ 動作をテスト
```python
# 結果の性質をテスト
predictions = model.predict(X_test)
assert len(predictions) == len(X_test)
assert predictions.dtype == np.float64
```

---

**覚えておいてください**: テストは `src/` 内のコードにのみ必要です。ノートブック（`notebook/`）のテストは任意です。
