---
name: ppdac-workflow
description: PPDACサイクルに基づくデータ分析ワークフロー。各フェーズのエージェント配置・品質ゲート・docs/連携・コミット戦略を定義します。新しい分析開始時や進め方を確認したいときに参照してください。
---

# PPDACワークフロー

## フロー概要

```
[Problem]
  CLAUDE.md・過去分析を読み込み、現状の問題点を把握する
[Plan]
  analysis-planner（人間と協働）→ docs/analysis_plan/<分析名>.md 保存
[Data]
  architect → SQLクエリ作成
       ↓ 品質ゲート: sql-reviewer + security-reviewer（並行）
       ↓ クエリ実行・データ取得
[Analysis / Code]
  architect → tdd-guide でコード作成
       ↓ 品質ゲート: code-reviewer + python-reviewer + security-reviewer（並行）
       ↓ 人間が分析実施
[Conclusion]
  analysis-reporter → docs/analysis_result/<分析名>.md 保存

[横断] 計画変更 → commit + analysis-planner（docs/analysis_plan/ 更新）
```

---

## フェーズ詳細

### Phase 1: Problem

**目的**: プロジェクトの目的と現状を把握し、今回解くべき問いを特定する

| 項目 | 内容 |
|------|------|
| 使用エージェント | なし（人間主導） |
| 人間関与度 | 高（判断・優先度決定） |
| docs/連携 | 読み取りのみ |
| コミット | なし |

**実施内容**:

1. `CLAUDE.md` を読み込み、プロジェクトの目的・ビジネス課題を把握する
2. `docs/analysis_result/` の既存レポートを確認し、過去の分析で明らかになったこと・残課題を把握する
3. `docs/analysis_plan/` の既存計画を確認し、進行中・完了済みの分析を把握する
4. 上記をもとに「現時点での問題点・未解明の問い」を整理し、ユーザーに提示する

**成果物**:
- 現状の問題点・未解明の問いの整理（テキストサマリー）
- 今回の分析で何を明らかにすべきかの合意

---

### Phase 2: Plan

**目的**: 分析の問いを明確化し、仮説とデータ要件を定義する

| 項目 | 内容 |
|------|------|
| 使用エージェント | `analysis-planner` |
| 人間関与度 | 高（協働作業） |
| docs/連携 | `analysis-planner` が `docs/analysis_plan/<分析名>.md` を直接保存 |
| コミット | `docs: create analysis plan for [分析名]` |

**実施内容**:

Phase 1 で整理したコンテキストを `analysis-planner` に渡して計画を策定する:

```
# analysis-planner に渡すコンテキスト
- プロジェクト目的（CLAUDE.md より）
- 過去の分析サマリー（docs/analysis_result/ より）
- 現時点で未解決の問い・課題（Phase 1 の成果）
- 今回の分析で何を明らかにしたいか
```

**成果物**:
- 分析目的・仮説（H1, H2, ...）・想定結果
- 必要なデータ・期間・粒度の定義
- `docs/analysis_plan/<分析名>.md`

---

### Phase 3: Data

**目的**: 仮説検証に必要なデータを取得する

| 項目 | 内容 |
|------|------|
| 使用エージェント | `architect`（クエリ作成前）, `sql-reviewer`（クエリ作成後）, `security-reviewer`（クエリ作成後） |
| 人間関与度 | 中（クエリ確認・実行） |
| docs/連携 | 計画変更があれば `analysis-planner` が `docs/analysis_plan/<分析名>.md` を直接更新 |
| コミット | `feat: add data extraction query for [分析名]` |

**品質ゲート**:
```
Task 1: sql-reviewer → コスト・正確性・セキュリティ
Task 2: security-reviewer → クレデンシャル・アクセス権限
```

---

### Phase 4: Analysis / Code

**目的**: データを分析し、仮説を検証するコードを作成する

| 項目 | 内容 |
|------|------|
| 使用エージェント | `architect`（コード作成前）, `tdd-guide`（コード作成中）, `code-reviewer`（コード作成後）, `python-reviewer`（コード作成後）, `security-reviewer`（コード作成後） |
| 人間関与度 | 高（分析の解釈・判断） |
| docs/連携 | 計画変更があれば `analysis-planner` が `docs/analysis_plan/<分析名>.md` を直接更新 |
| コミット | `feat: implement analysis code for [分析名]` |

**品質ゲート**:
```
Task 1: code-reviewer → 汎用品質・セキュリティ
Task 2: python-reviewer → Python固有のイディオム・Pandas
Task 3: security-reviewer → データアクセス・シークレット
```

---

### Phase 5: Conclusion

**目的**: 分析結果を集約し、意思決定可能なレポートを作成する

| 項目 | 内容 |
|------|------|
| 使用エージェント | `analysis-reporter`, `refactor-cleaner`（オプション） |
| 人間関与度 | 中（レポート内容の確認・承認） |
| docs/連携 | `analysis-reporter` が `docs/analysis_result/<分析名>.md` を直接保存 |
| コミット | `docs: add analysis report for [分析名]` |

**手順**:
1. `analysis-reporter` が `docs/analysis_plan/<分析名>.md` から分析計画を取得し、結果と突き合わせる
2. 分析レポートテンプレートに沿って Conclusion を構造化
3. `analysis-reporter` が `docs/analysis_result/<分析名>.md` として直接保存

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

## docs/連携まとめ

| ファイル | 作成タイミング | 担当エージェント |
|---------|-------------|---------------|
| `docs/analysis_plan/<分析名>.md` | Planフェーズ完了時 | `analysis-planner` |
| `docs/analysis_result/<分析名>.md` | Conclusionフェーズ完了時 | `analysis-reporter` |

計画変更が生じた場合は `analysis-planner` が既存の `docs/analysis_plan/<分析名>.md` を更新する。
