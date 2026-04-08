---
paths:
  - "**/*.ipynb"
---
# Jupyterノートブックガイドライン

## セル構成の原則

ノートブックは基本的に以下のセクション順に構成する（ただし、分析内容に応じて変更可能）:

```
1. セットアップ・インポート
2. 定数・設定
3. データ読み込み
4. 探索的データ分析（EDA）
5. 特徴量エンジニアリング・前処理
6. モデリング・分析
7. 結果・可視化
8. まとめ
```

## 再現性の確保

- **Restart & Run All** で最初から最後まで再現できること
- セルの実行順序に依存する副作用を持ち込まない
- 乱数シードは定数として先頭セルで設定する

```python
# セル1: Setup
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from pathlib import Path

SEED = 42
np.random.seed(SEED)

# ディレクトリ設定はCLAUDE.mdのディレクトリ構造を参照
OUTPUT_DIR = Path("outputs/analysis1_foo")
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)
```

## マジックナンバー禁止

```python
# BAD
df = df[df["age"] >= 20]
top_n = df.head(100)

# GOOD
MIN_AGE = 20
TOP_N = 100
df = df[df["age"] >= MIN_AGE]
top_n = df.head(TOP_N)
```

## 長い処理は src/ に抽出

セルに50行以上の処理が集中する場合は `src/` にモジュールとして抽出（CLAUDE.mdのディレクトリ構造を参照）:

```python
# BAD: 長い前処理をセルに直書き

# GOOD: src/ に関数として定義して呼び出す
from src.mypackage.preprocessing import clean_transactions
df_clean = clean_transactions(df_raw)
```

## 出力管理

- 図は必ず `OUTPUT_DIR` に保存してから表示
- `plt.show()` の前に `fig.savefig(...)` を呼ぶ
- 大きなDataFrameは `.head(20)` で表示（全行出力禁止）

```python
fig, ax = plt.subplots()
ax.hist(df["revenue"], bins=50)
fig.savefig(OUTPUT_DIR / "analysis1_revenue_dist.png", dpi=150)
plt.show()
plt.close(fig)
```

## コミット前チェック

- [ ] Restart & Run All で全セル通過
- [ ] 不要な出力（中間print）を削除
- [ ] セル出力をクリアしてからコミット（大容量画像出力を避ける）
