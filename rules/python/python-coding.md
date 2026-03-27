---
paths:
  - "**/*.py"
  - "**/*.pyi"
---
# Pythonコーディングスタイル

## 標準規約

- **PEP 8** の規約に従う
- 全関数シグネチャに**型アノテーション**を付ける
- ファイルサイズ: 通常200〜400行、最大800行
- 関数: 50行未満、深いネスト禁止（4レベル超）

## フォーマット

```bash
ruff format .      # フォーマット
ruff check .       # リント
```

## イミュータビリティ

既存オブジェクトを変更せず、必ず新しいオブジェクトを生成する:

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

## ファイルパス

文字列連結ではなく `pathlib.Path` を使用する:

```python
from pathlib import Path

output_dir = Path("outputs/analysis1_foo")
output_dir.mkdir(parents=True, exist_ok=True)
df.to_csv(output_dir / "analysis1_result.csv", index=False)
```

## 環境変数

```python
from dotenv import load_dotenv
import os

load_dotenv()
project_id = os.environ["GCP_PROJECT_ID"]
```

## エラー処理

- エラーは明示的に処理する。サイレントに無視しない
- 裸の `except:` ではなく具体的な例外型を使用する
- 詳細なコンテキストはサーバーサイドでログ記録し、境界ではユーザーフレンドリーなメッセージを表示する

## Pandas / NumPy イディオム

```python
# 良い例: ベクトル化演算
df["total"] = df["price"] * df["qty"]
result = df.groupby("user_id")["revenue"].sum()

# 悪い例: Pythonループ
for i, row in df.iterrows():
    df.at[i, "total"] = row["price"] * row["qty"]
```

優先順位: ベクトル化 > `apply()` > 明示的ループ

## プロトコル（ダックタイピング）

```python
from typing import Protocol

class DataLoader(Protocol):
    def load(self, path: str) -> "pd.DataFrame": ...
```

## コード品質チェックリスト

作業完了前に確認:
- [ ] 全関数シグネチャに型アノテーションがある
- [ ] `ruff format` と `ruff check` が通過する
- [ ] マジックナンバーがない（定数を使用する）
- [ ] ミューテーションがない（イミュータブルパターンを使用する）
- [ ] 適切なエラー処理がある
