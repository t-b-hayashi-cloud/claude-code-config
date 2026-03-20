---
name: security-review
description: 認証の追加、外部APIの呼び出し、シークレットの操作、BigQueryクエリの作成時にこのスキルを使用します。Pythonデータ分析プロジェクト向けの包括的なセキュリティチェックリストとパターンを提供します。
---

# セキュリティレビュースキル（Python データ分析）

## 有効化するタイミング

- 新しいAPIキーや認証情報を使用するコードを書くとき
- BigQueryクエリを作成するとき
- ファイルパスをユーザー入力から構築するとき
- 外部サービス（GCP、OpenAI、Slack等）と統合するとき
- `.env` ファイルや設定ファイルを変更するとき

## セキュリティチェックリスト

### 1. シークレット管理（クリティカル）

```python
# ❌ 絶対にしないこと
api_key = "AIzaSyXXXXXXXXX"         # ハードコード
project_id = "my-gcp-project"      # ハードコード
PASSWORD = "admin123"               # ハードコード

# ✅ 常にすること
import os
from dotenv import load_dotenv

load_dotenv()  # .env ファイルから読み込み

api_key = os.getenv("GOOGLE_API_KEY")
if not api_key:
    raise ValueError("GOOGLE_API_KEY が .env ファイルに設定されていません")
```

**検証ステップ:**
- [ ] `.env` ファイルが `.gitignore` に含まれている
- [ ] `.env.example` に必要な変数が全てリストされている
- [ ] コードに文字列リテラルのシークレットがない
- [ ] `git log -p | grep -i "key\|token\|secret"` で履歴を確認

### 2. BigQueryクエリのセキュリティ（クリティカル）

```python
# ❌ 悪い: ユーザー入力をSQLに直接埋め込む
query = f"SELECT * FROM `{user_table}` WHERE id = {user_id}"

# ✅ 良い: ホワイトリスト検証とパラメータ化クエリ
from google.cloud import bigquery

ALLOWED_TABLES = {"orders", "users", "events"}

def safe_query(table_name: str, user_id: str) -> pd.DataFrame:
    if table_name not in ALLOWED_TABLES:
        raise ValueError(f"無効なテーブル: {table_name}")

    client = bigquery.Client()
    query = """
        SELECT * FROM `project.dataset.{table}`
        WHERE user_id = @user_id
    """.format(table=table_name)  # ホワイトリスト済みのみ

    job_config = bigquery.QueryJobConfig(
        query_parameters=[
            bigquery.ScalarQueryParameter("user_id", "STRING", user_id)
        ]
    )
    return client.query(query, job_config=job_config).to_dataframe()
```

### 3. パストラバーサル防止（高）

```python
# ❌ 悪い: ユーザー入力からパスを構築
def load_data(filename: str) -> pd.DataFrame:
    return pd.read_csv(f"data/{filename}")  # パストラバーサルのリスク

# ✅ 良い: パスを検証
from pathlib import Path

def load_data(filename: str, base_dir: Path = Path("data")) -> pd.DataFrame:
    base = base_dir.resolve()
    target = (base / filename).resolve()

    # ベースディレクトリ外へのアクセスを防ぐ
    if not str(target).startswith(str(base)):
        raise ValueError(f"無効なファイルパス: {filename}")

    # 拡張子チェック
    if target.suffix not in {".csv", ".parquet", ".json"}:
        raise ValueError(f"許可されていないファイル形式: {target.suffix}")

    return pd.read_csv(target)
```

### 4. 外部APIの安全な使用（中）

```python
# ✅ タイムアウトとエラーハンドリングを含める
import requests

def call_external_api(endpoint: str, data: dict) -> dict:
    api_key = os.environ["API_KEY"]  # 環境変数から取得
    headers = {"Authorization": f"Bearer {api_key}"}

    try:
        response = requests.post(
            endpoint,
            json=data,
            headers=headers,
            timeout=30,  # タイムアウトを設定
        )
        response.raise_for_status()  # エラーレスポンスを例外に変換
        return response.json()
    except requests.exceptions.Timeout:
        raise RuntimeError("APIリクエストがタイムアウトしました")
    except requests.exceptions.HTTPError as e:
        raise RuntimeError(f"APIエラー: {e}")
```

### 5. ログへの機密データ漏洩防止（中）

```python
# ❌ 悪い: 機密データをログに記録
import logging
logger = logging.getLogger(__name__)

def process_user(user_id: str, credit_card: str):
    logger.info(f"ユーザー {user_id} のカード {credit_card} を処理中")  # 危険！

# ✅ 良い: 機密データはマスク
def process_user(user_id: str, credit_card: str):
    masked_card = f"****-****-****-{credit_card[-4:]}"
    logger.info(f"ユーザー {user_id} の支払い処理中 ({masked_card})")
```

## 依存関係セキュリティ

```bash
# 脆弱性をチェック（定期的に実行）
uv run pip-audit

# 依存関係を最新に保つ
uv lock --upgrade
```

## デプロイ前セキュリティチェックリスト

- [ ] **シークレット**: コードにハードコードされたシークレットがない
- [ ] **環境変数**: すべてのシークレットが `.env` に移動されている
- [ ] **gitignore**: `.env`, `*.key`, `credentials.json` が除外されている
- [ ] **SQLインジェクション**: BigQueryクエリがパラメータ化されている
- [ ] **パストラバーサル**: ファイルパスが検証されている
- [ ] **依存関係**: `uv run pip-audit` がクリーン
- [ ] **ログ**: 機密データがログに含まれていない

## セキュリティ問題発見時の対応

1. **即座に停止** - 該当シークレットをGCP/サービスコンソールでローテーション
2. **git履歴確認** - `git log -p | grep -i "key\|secret"` でコミット済みシークレットを探す
3. **git履歴クリーン** - 必要に応じてBFG Repo Cleanerを使用
4. **`.env.example` 更新** - 必要な環境変数を文書化

---

詳細なセキュリティ分析には `security-reviewer` エージェントを使用してください。
