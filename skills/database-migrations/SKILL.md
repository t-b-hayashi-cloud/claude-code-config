---
name: database-migrations
description: データベースマイグレーションのベストプラクティス。スキーマ変更、データマイグレーション、ロールバック、ゼロダウンタイムデプロイ。PostgreSQLとPython ORM（Alembic、Django）向け。
---

# データベースマイグレーションパターン

本番システム向けの安全で可逆的なデータベーススキーマ変更。

## 有効化するタイミング

- データベーステーブルを作成または変更するとき
- カラムやインデックスを追加・削除するとき
- データマイグレーション（バックフィル、変換）を実行するとき
- ゼロダウンタイムスキーマ変更を計画するとき

## コア原則

1. **すべての変更はマイグレーションとして** — 本番DBを手動で変更しない
2. **本番はフォワードオンリー** — ロールバックは新しいフォワードマイグレーション
3. **スキーマとデータは分離** — DDLとDMLを同じマイグレーションに混ぜない
4. **マイグレーションはイミュータブル** — デプロイ済みのマイグレーションを編集しない

## マイグレーション安全チェックリスト

マイグレーション適用前に:

- [ ] UPとDOWNが両方ある（または不可逆とマーク）
- [ ] 大きなテーブルでの全テーブルロックなし
- [ ] 新しいカラムはデフォルト値ありまたはNULL許容
- [ ] インデックスはCONCURRENTLYで作成
- [ ] データバックフィルはスキーマ変更と別のマイグレーション
- [ ] 本番サイズのデータでテスト済み
- [ ] ロールバック計画が文書化されている

## PostgreSQL パターン

### カラムの安全な追加

```sql
-- ✅ 良い: NULL許容カラム、ロックなし
ALTER TABLE users ADD COLUMN avatar_url TEXT;

-- ✅ 良い: デフォルト付きカラム（Postgres 11+は即座、書き直しなし）
ALTER TABLE users ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT true;

-- ❌ 悪い: 既存テーブルへのNOT NULLなしのカラム追加
ALTER TABLE users ADD COLUMN role TEXT NOT NULL;
-- テーブルロックと全行書き直しが発生！
```

### ダウンタイムなしのインデックス追加

```sql
-- ❌ 悪い: 大きなテーブルで書き込みをブロック
CREATE INDEX idx_users_email ON users (email);

-- ✅ 良い: 同時書き込みを許可
CREATE INDEX CONCURRENTLY idx_users_email ON users (email);
-- 注意: CONCURRENTLYはトランザクションブロック内で使用不可
```

### カラムのリネーム（ゼロダウンタイム）

直接リネームは危険。Expand-Contractパターンを使用:

```sql
-- ステップ1: 新しいカラムを追加（マイグレーション001）
ALTER TABLE users ADD COLUMN display_name TEXT;

-- ステップ2: データをバックフィル（マイグレーション002）
UPDATE users SET display_name = username WHERE display_name IS NULL;

-- ステップ3: アプリコードを更新（両方のカラムを読み書き）→ デプロイ

-- ステップ4: 古いカラムを削除（マイグレーション003）
ALTER TABLE users DROP COLUMN username;
```

### 大きなデータマイグレーション

```sql
-- ❌ 悪い: 全行を1トランザクションで更新（テーブルロック）
UPDATE users SET normalized_email = LOWER(email);

-- ✅ 良い: バッチ更新
DO $$
DECLARE
  batch_size INT := 10000;
  rows_updated INT;
BEGIN
  LOOP
    UPDATE users
    SET normalized_email = LOWER(email)
    WHERE id IN (
      SELECT id FROM users
      WHERE normalized_email IS NULL
      LIMIT batch_size
      FOR UPDATE SKIP LOCKED
    );
    GET DIAGNOSTICS rows_updated = ROW_COUNT;
    EXIT WHEN rows_updated = 0;
    COMMIT;
  END LOOP;
END $$;
```

## Alembic（Python/SQLAlchemy）

### ワークフロー

```bash
# スキーマ変更からマイグレーションを生成
alembic revision --autogenerate -m "add_user_avatar"

# 保留中のマイグレーションを適用
alembic upgrade head

# 1つ前にロールバック
alembic downgrade -1

# マイグレーション履歴を確認
alembic history
```

### カスタムSQLマイグレーション

```python
# alembic/versions/xxxx_add_email_index.py
from alembic import op

def upgrade():
    # CONCURRENTLYはトランザクション外で実行する必要がある
    op.execute("COMMIT")
    op.execute("CREATE INDEX CONCURRENTLY idx_users_email ON users (email)")
    op.execute("BEGIN")

def downgrade():
    op.execute("COMMIT")
    op.execute("DROP INDEX CONCURRENTLY IF EXISTS idx_users_email")
    op.execute("BEGIN")
```

### データマイグレーション

```python
def upgrade():
    # スキーマ変更
    op.add_column('users', sa.Column('display_name', sa.Text()))

def downgrade():
    op.drop_column('users', 'display_name')
```

## ゼロダウンタイムマイグレーション戦略

Expand-Contractパターン:

```
フェーズ1: EXPAND
  - 新しいカラム/テーブルを追加（NULLまたはデフォルト）
  - アプリを両方に書き込むようにデプロイ
  - 既存データをバックフィル

フェーズ2: MIGRATE
  - アプリを新しいものから読むようにデプロイ
  - データの一貫性を確認

フェーズ3: CONTRACT
  - アプリが新しいもののみ使用
  - 古いカラム/テーブルを別のマイグレーションで削除
```

## アンチパターン

| アンチパターン | 失敗する理由 | より良いアプローチ |
|-------------|------------|----------------|
| 本番DBの手動変更 | 監査証跡なし、再現不可能 | 常にマイグレーションファイルを使用 |
| デプロイ済みマイグレーションの編集 | 環境間のドリフトを引き起こす | 新しいマイグレーションを作成 |
| デフォルトなしのNOT NULL | テーブルをロックし全行書き直し | NULLで追加、バックフィル、制約追加 |
| 大きなテーブルのインラインインデックス | 書き込みをブロック | CONCURRENTLY を使用 |
| スキーマ+データを1つのマイグレーションに | ロールバックが困難 | マイグレーションを分離 |
| コード削除前にカラムを削除 | アプリがクラッシュ | 先にコードを削除、次のデプロイでカラムを削除 |
