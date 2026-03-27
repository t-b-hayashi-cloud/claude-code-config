# セキュリティガイドライン

## シークレット管理

- ソースコードにシークレットを直接書かない
- 環境変数または `.env`（gitignore済み）を必ず使用する
- 起動時に必要なシークレットの存在を検証する
- 漏洩した可能性のあるシークレットはローテーションする

Python 例:
```python
import os
from dotenv import load_dotenv

load_dotenv()
api_key = os.environ["API_KEY"]  # Raises KeyError if missing
```

## コミット前チェック

コミット前に必ず確認:
- [ ] ハードコードされたシークレットなし（APIキー・パスワード・トークン）
- [ ] `.env` が gitignore されている（`.env.example` は絶対にコミットしない）
- [ ] SQLクエリはパラメータ化クエリを使用（f文字列補間は禁止）
- [ ] エラーメッセージに機密データが含まれていない

## セキュリティスキャン

```bash
bandit -r src/    # Python static security analysis
```

## セキュリティ対応プロトコル

セキュリティ問題を発見したら:
1. 即座に作業を停止する
2. **security-reviewer** エージェントを使用する
3. CRITICALな問題を修正してから作業を再開する
4. 漏洩したシークレットをローテーションする
