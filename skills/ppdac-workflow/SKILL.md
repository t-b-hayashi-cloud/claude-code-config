---
name: ppdac-workflow
description: PPDACサイクルに基づくデータ分析ワークフロー。各フェーズのエージェント配置・品質ゲート・Notion連携・コミット戦略を定義します。新しい分析開始時や進め方を確認したいときに参照してください。
---

# PPDACワークフロー

## フロー概要

```
[Problem & Plan]
  analysis-planner（人間と協働）
       ↓ doc-updater → Notion記載（Analysis plan）
[Data]
  architect（オプション）→ SQLクエリ作成
       ↓ 品質ゲート: sql-reviewer + security-reviewer（並行）
       ↓ クエリ実行・データ取得
[Analysis / Code]
  architect（オプション）→ tdd-guide でコード作成
       ↓ 品質ゲート: code-reviewer + python-reviewer + security-reviewer（並行）
       ↓ 人間が分析実施
[Conclusion]
  analysis-reporter → doc-updater → Notion記載（Analysis report）
  refactor-cleaner（コードリファクタ）

[横断] 計画変更 → commit + doc-updater（Notion更新）
```

---

## フェーズ詳細

### Phase 1: Problem & Plan

**目的**: 分析の問いを明確化し、仮説とデータ要件を定義する

| 項目 | 内容 |
|------|------|
| 使用エージェント | `analysis-planner` |
| 人間関与度 | 高（協働作業） |
| Notion連携 | `doc-updater` が Analysis plan を作成 |
| コミット | `docs: create analysis plan for [分析名]` |

**成果物**:
- 分析目的・仮説（H1, H2, ...）・想定結果
- 必要なデータ・期間・粒度の定義
- Notionの Analysis plan ページ

---

### Phase 2: Data

**目的**: 仮説検証に必要なデータを取得する

| 項目 | 内容 |
|------|------|
| 使用エージェント | `architect`（オプション）, `sql-reviewer`, `security-reviewer` |
| 人間関与度 | 中（クエリ確認・実行） |
| Notion連携 | 計画変更があれば `doc-updater` が Analysis plan を更新 |
| コミット | `feat: add data extraction query for [分析名]` |

**品質ゲート（必ず並行起動）**:
```
Task 1: sql-reviewer → コスト・正確性・セキュリティ
Task 2: security-reviewer → クレデンシャル・アクセス権限
```

---

### Phase 3: Analysis / Code

**目的**: データを分析し、仮説を検証するコードを作成する

| 項目 | 内容 |
|------|------|
| 使用エージェント | `architect`（オプション）, `tdd-guide`, `code-reviewer`, `python-reviewer`, `security-reviewer` |
| 人間関与度 | 高（分析の解釈・判断） |
| Notion連携 | 計画変更があれば `doc-updater` が Analysis plan を更新 |
| コミット | `feat: implement analysis code for [分析名]` |

**品質ゲート（必ず並行起動）**:
```
Task 1: code-reviewer → 汎用品質・セキュリティ
Task 2: python-reviewer → Python固有のイディオム・Pandas
Task 3: security-reviewer → データアクセス・シークレット
```

---

### Phase 4: Conclusion

**目的**: 分析結果を集約し、意思決定可能なレポートを作成する

| 項目 | 内容 |
|------|------|
| 使用エージェント | `analysis-reporter`, `doc-updater`, `refactor-cleaner`（オプション） |
| 人間関与度 | 中（レポート内容の確認・承認） |
| Notion連携 | `doc-updater` が Analysis report を作成 |
| コミット | `docs: add analysis report for [分析名]` |

**手順**:
1. `analysis-reporter` が Notion から分析計画を取得し、結果と突き合わせる
2. Notion Analysis Report テンプレートに沿って Conclusion を構造化
3. `doc-updater` が Notion に Analysis report を記載
4. （オプション）`refactor-cleaner` でコードの整理

---

## 品質ゲートの運用

**レビュワーは必ず並行起動する**（Task tool で同時呼び出し）:

```python
# GOOD: 並行起動
Task(sql-reviewer), Task(security-reviewer)  # 同時

# BAD: 直列起動
Task(sql-reviewer) → 完了後 → Task(security-reviewer)
```

各レビュワーからCRITICAL/HIGHが出た場合は修正してから次フェーズへ進む。

---

## コミット戦略

| タイミング | コミットメッセージ |
|-----------|-----------------|
| 計画確定時 | `docs: create analysis plan for [分析名]` |
| データ取得完了時 | `feat: add data extraction query for [分析名]` |
| 分析コード完成時 | `feat: implement analysis code for [分析名]` |
| 計画変更時 | `docs: update analysis plan - [変更内容]` |
| 分析完了時 | `docs: add analysis report for [分析名]` |

---

## Notion連携まとめ

| テンプレート | 作成タイミング | 担当エージェント |
|------------|-------------|---------------|
| Analysis plan | Planフェーズ完了時 | analysis-planner → doc-updater |
| Analysis report | Conclusionフェーズ完了時 | analysis-reporter → doc-updater |

計画変更が生じた場合は `doc-updater` が既存の Analysis plan ページを更新する。
