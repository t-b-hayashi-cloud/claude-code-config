---
name: postgres-patterns
description: PostgreSQL/BigQueryデータベースパターンのクイックリファレンス。クエリ最適化、スキーマ設計、インデックス設計、セキュリティ。詳細なレビューには database-reviewer エージェントを使用。
---

# PostgreSQL / BigQuery パターン

クイックリファレンス。詳細なガイダンスについては `database-reviewer` エージェントを使用してください。

## 起動タイミング

- SQLクエリやマイグレーションを作成するとき
- データベーススキーマを設計するとき
- 遅いクエリをトラブルシューティングするとき
- BigQueryのコストを最適化するとき

## BigQuery クイックリファレンス

### コスト最適化

| アンチパターン | 問題 | 解決策 |
|--------------|------|--------|
| `SELECT *` | 不要なカラムを全スキャン | 必要なカラムのみ指定 |
| パーティションフィルタなし | テーブル全体をスキャン | `WHERE DATE(ts) >= '2024-01-01'` |
| 重複したサブクエリ | 同じデータを複数回読み込み | CTEでキャッシュ |

```sql
-- ✅ コスト効率の良いクエリ
SELECT
    user_id,
    DATE(event_timestamp) AS event_date,
    COUNT(*) AS event_count
FROM `project.dataset.events`
WHERE DATE(event_timestamp) BETWEEN @start_date AND @end_date  -- パーティションフィルタ
  AND event_name IN ('purchase', 'add_to_cart')
GROUP BY 1, 2;
```

### パーティション & クラスタ

```sql
-- パーティション化されたテーブルを作成
CREATE TABLE `project.dataset.events`
PARTITION BY DATE(event_timestamp)
CLUSTER BY user_id, event_name
AS SELECT ...;
```

### CTE（WITH句）の活用

```sql
WITH
daily_revenue AS (
    SELECT
        DATE(created_at) AS date,
        SUM(amount) AS revenue
    FROM `project.dataset.orders`
    WHERE DATE(created_at) >= @start_date
    GROUP BY 1
),
daily_orders AS (
    SELECT
        DATE(created_at) AS date,
        COUNT(*) AS order_count
    FROM `project.dataset.orders`
    WHERE DATE(created_at) >= @start_date
    GROUP BY 1
)
SELECT
    r.date,
    r.revenue,
    o.order_count,
    SAFE_DIVIDE(r.revenue, o.order_count) AS avg_order_value
FROM daily_revenue r
JOIN daily_orders o USING (date)
ORDER BY r.date;
```

## PostgreSQL クイックリファレンス

### インデックスチートシート

| クエリパターン | インデックスタイプ | 例 |
|--------------|------------|---------|
| `WHERE col = value` | B-tree（デフォルト） | `CREATE INDEX idx ON t (col)` |
| `WHERE a = x AND b > y` | 複合 | `CREATE INDEX idx ON t (a, b)` |
| `WHERE jsonb @> '{}'` | GIN | `CREATE INDEX idx ON t USING gin (col)` |
| 時系列範囲 | BRIN | `CREATE INDEX idx ON t USING brin (col)` |

### データ型クイックリファレンス

| 用途 | 正しいタイプ | 避けるべき |
|----------|-------------|-------|
| ID | `bigint` | `int`、ランダムUUID |
| 文字列 | `text` | `varchar(255)` |
| タイムスタンプ | `timestamptz` | `timestamp` |
| 金額 | `numeric(10,2)` | `float` |

### よく使うパターン

**カーソルベースのページネーション（O(1)）:**
```sql
SELECT * FROM products WHERE id > @last_id ORDER BY id LIMIT 20;
-- OFFSETはO(n)なので避ける
```

**UPSERT:**
```sql
INSERT INTO settings (user_id, key, value)
VALUES (@user_id, @key, @value)
ON CONFLICT (user_id, key)
DO UPDATE SET value = EXCLUDED.value, updated_at = now();
```

## SQLファイル規約（CLAUDE.mdを参照）

```sql
-- purpose: [クエリの目的を1文で説明]
-- created: YYYY-MM-DD
-- output: data/[出力ファイル名].csv
-- table: project.dataset.table

SELECT ...
```

## 関連

- Agent: `database-reviewer` - 完全なデータベースレビューワークフロー
- Skill: `database-migrations` - マイグレーションパターン
