---
paths:
  - "**/*.py"
  - "**/*.ipynb"
---
# Data Analysis Guidelines

## 再現性の原則

```python
import numpy as np
import random

SEED = 42
np.random.seed(SEED)
random.seed(SEED)
```

- 生データ（`data/*_raw_*.csv`）は保存後に上書きしない
- 処理済みデータは `interim` / `final` として別ファイルに保存
- すべての前処理をコードで記録（手作業でのデータ修正禁止）

## データバリデーション

```python
import pandas as pd

def validate_df(df: pd.DataFrame, required_cols: list[str]) -> None:
    missing = set(required_cols) - set(df.columns)
    assert not missing, f"Missing columns: {missing}"
    assert df.duplicated().sum() == 0, "Duplicate rows found"
    assert df[required_cols].isnull().sum().sum() == 0, "Null values found"
```

pandera を使う場合:
```python
import pandera as pa

schema = pa.DataFrameSchema({
    "user_id": pa.Column(str, nullable=False),
    "revenue": pa.Column(float, pa.Check.ge(0)),
})
schema.validate(df)
```

## EDAワークフロー

1. `df.shape`, `df.dtypes`, `df.describe()` で概観
2. 欠損値: `df.isnull().sum()` → 対処方針を決める
3. 外れ値: ヒストグラム + IQR で確認
4. ターゲット分布: `df["target"].value_counts(normalize=True)`
5. 特徴量相関: `df.corr()` ヒートマップ

## モデリングワークフロー

```python
from sklearn.model_selection import StratifiedKFold, cross_validate

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=SEED)
scores = cross_validate(model, X, y, cv=cv, scoring=["roc_auc", "f1"])
```

- 必ずCross-Validationでモデル評価
- train/valid/test を明確に分離（leakage防止）
- 過学習監視: train score vs valid score の差を確認

## ハイパーパラメータ最適化（Optuna）

```python
import optuna

def objective(trial: optuna.Trial) -> float:
    params = {
        "n_estimators": trial.suggest_int("n_estimators", 100, 1000),
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.3, log=True),
    }
    score = cross_val_score(model_cls(**params), X, y, cv=5).mean()
    return score

study = optuna.create_study(direction="maximize", sampler=optuna.samplers.TPESampler(seed=SEED))
study.optimize(objective, n_trials=100)
```

## 可視化ベストプラクティス

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(8, 5))
ax.set_title("Revenue by Segment")
ax.set_xlabel("Segment")
ax.set_ylabel("Revenue (JPY)")
plt.tight_layout()

# 出力先はCLAUDE.mdのディレクトリ構造を参照
fig.savefig(output_dir / "analysis1_revenue_by_segment.png", dpi=150)
plt.close(fig)
```

- 図は必ず `outputs/` に保存（インラインのみ禁止）
- 日本語フォント設定が必要な場合は `matplotlib-japanize` を使用

## BigQueryデータ取得

```python
from google.cloud import bigquery

client = bigquery.Client(project=os.environ["GCP_PROJECT_ID"])

query = """
SELECT user_id, SUM(revenue) AS total_revenue
FROM `project.dataset.table`
WHERE DATE(created_at) BETWEEN @start_date AND @end_date
GROUP BY user_id
"""

job_config = bigquery.QueryJobConfig(
    query_parameters=[
        bigquery.ScalarQueryParameter("start_date", "DATE", "2024-01-01"),
        bigquery.ScalarQueryParameter("end_date", "DATE", "2024-03-31"),
    ]
)

df = client.query(query, job_config=job_config).to_dataframe()
```
