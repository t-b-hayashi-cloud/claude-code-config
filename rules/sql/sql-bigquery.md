---
paths:
  - "**/*.sql"
---
# SQL / BigQuery Guidelines

## コスト最適化（最重要）

```sql
-- BAD: パーティションフィルタなし（フルスキャン）
SELECT * FROM `project.dataset.events`
WHERE user_id = '123'

-- GOOD: パーティションフィルタあり
SELECT user_id, event_type, created_at
FROM `project.dataset.events`
WHERE DATE(created_at) BETWEEN '2024-01-01' AND '2024-03-31'
  AND user_id = '123'
```

- `SELECT *` 禁止 — 必要なカラムのみ指定
- パーティション列（`DATE(created_at)` 等）は必ず WHERE 句に含める
- 大きなテーブルは `LIMIT` をつけて開発・テスト

## パラメータ化クエリ（SQLインジェクション防止）

```python
# BAD: f文字列でクエリ生成
query = f"SELECT * FROM table WHERE user_id = '{user_id}'"

# GOOD: QueryJobConfig でパラメータ化
job_config = bigquery.QueryJobConfig(
    query_parameters=[
        bigquery.ScalarQueryParameter("user_id", "STRING", user_id),
    ]
)
client.query(query, job_config=job_config)
```

## CTE（共通テーブル式）

複雑なクエリはCTEで分割して可読性を上げる:

```sql
WITH
  active_users AS (
    SELECT DISTINCT user_id
    FROM `project.dataset.events`
    WHERE DATE(created_at) >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
  ),
  user_revenue AS (
    SELECT user_id, SUM(amount) AS total_revenue
    FROM `project.dataset.orders`
    GROUP BY user_id
  )
SELECT
  a.user_id,
  COALESCE(r.total_revenue, 0) AS total_revenue
FROM active_users a
LEFT JOIN user_revenue r USING (user_id)
```

## NULL ハンドリング

```sql
-- NULL を含む集計
SELECT
  user_id,
  COALESCE(SUM(revenue), 0) AS total_revenue,
  COUNT(DISTINCT order_id) AS order_count,
  AVG(NULLIF(discount_rate, 0)) AS avg_nonzero_discount
FROM orders
GROUP BY user_id
```

## データ型選択

| 用途 | 型 |
|---|---|
| 日付のみ | `DATE` |
| タイムスタンプ | `TIMESTAMP` |
| 金額（整数円） | `INT64` |
| 金額（小数点あり） | `NUMERIC` |
| フラグ（0/1） | `BOOL` |

## ファイルメタデータ（先頭コメント必須）

```sql
-- purpose: Analysis3 用ユーザーセグメント集計
-- created: 2024-01-15
-- output: data/analysis3_final_segments.csv
-- author: t-b-hayashi
```

## パーティション & クラスタリング（テーブル作成時）

```sql
CREATE TABLE `project.dataset.user_events`
PARTITION BY DATE(created_at)
CLUSTER BY user_id, event_type
AS SELECT ...
```
