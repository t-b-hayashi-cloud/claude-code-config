---
name: verification-loop
description: 機能完成・PR作成前の品質検証ワークフロー。Pythonデータ分析プロジェクト向けの包括的な検証チェックリスト。
---

# 検証ループスキル

Claude Codeセッション向けの包括的な検証システム（Pythonデータ分析プロジェクト向け）。

## 使用タイミング

このスキルを呼び出す:
- 機能または重要なコード変更を完了した後
- PRを作成する前
- 品質ゲートが通過することを確認したい場合
- リファクタリング後

## 検証フェーズ

### フェーズ1: インポート検証

```bash
# パッケージが正しくインストールされているか確認
uv run python -c "import src; print('OK')"

# 依存関係に問題がないか確認
uv run python -m pip check
```

インポートが失敗した場合、停止して続行前に修正。

### フェーズ2: 型チェック

```bash
# mypyで型チェック（設定がある場合）
uv run mypy src/ 2>&1 | head -30

# または pyright
uv run pyright src/ 2>&1 | head -30
```

型エラーをすべて報告。重要なものは続行前に修正。

### フェーズ3: Lintチェック

```bash
# ruffでlint
uv run ruff check . 2>&1 | head -30

# ruffでフォーマット確認
uv run ruff format --check . 2>&1 | head -20
```

### フェーズ4: テストスイート

```bash
# カバレッジ付きでテストを実行
uv run pytest --cov=src --cov-report=term-missing 2>&1 | tail -50
```

報告:
- 合計テスト数: X
- 成功: X
- 失敗: X
- カバレッジ: X%（目標: 最低80%）

### フェーズ5: セキュリティスキャン

```bash
# ハードコードされたシークレットを確認
grep -rn "api_key\s*=\s*['\"]" --include="*.py" . 2>/dev/null | grep -v ".env" | head -10
grep -rn "password\s*=\s*['\"]" --include="*.py" . 2>/dev/null | grep -v "test" | head -10
grep -rn "AIza\|sk-\|ghp_" --include="*.py" . 2>/dev/null | head -10

# 不要なprint文を確認（ログにすべき箇所）
grep -rn "^[[:space:]]*print(" src/ 2>/dev/null | head -10

# 依存関係の脆弱性を確認（任意）
uv run pip-audit 2>&1 | head -20
```

### フェーズ6: 差分レビュー

```bash
# 変更内容を表示
git diff --stat
git diff HEAD --name-only
```

各変更ファイルをレビュー:
- 意図しない変更
- 不足しているエラー処理
- 潜在的なエッジケース
- `data/`, `outputs/`, `model/` へのコミット（gitignore対象なので不要）

### フェーズ7: ノートブック確認（任意）

```bash
# ノートブックに出力が残っていないか確認（nbstripoutで削除）
nbstripout notebook/*.ipynb 2>/dev/null
```

## 出力フォーマット

すべてのフェーズを実行後、検証レポートを作成:

```
検証レポート
==================

インポート:   [成功/失敗]
型チェック:   [成功/失敗] (Xエラー)
Lint:        [成功/失敗] (X警告)
テスト:      [成功/失敗] (X/Y成功、Z%カバレッジ)
セキュリティ: [成功/失敗] (X問題)
差分:        [Xファイル変更]

総合:        PRの準備[完了/未完了]

修正すべき問題:
1. ...
2. ...
```

## 継続モード

長いセッションの場合、主要な変更後に検証を実行:

```markdown
チェックポイントを設定:
- 各関数を完了した後
- src/ に新しいモジュールを追加した後
- 次のタスクに移る前

実行: /verification-loop
```

## フックとの統合

このスキルはPostToolUseフックを補完しますが、より深い検証を提供します。
フックは問題を即座に捕捉し、このスキルは包括的なレビューを提供します。

## プロジェクト固有のチェック（CLAUDE.mdを参照）

- `.gitignore` に `data/`, `outputs/`, `model/`, `.env` が含まれていること
- `pyproject.toml` の依存関係が `uv.lock` と一致していること
- SQLファイルにヘッダーコメントがあること
- 分析完了後に `outputs/` に適切なレポートがあること
