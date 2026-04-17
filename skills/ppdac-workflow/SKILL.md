---
name: ppdac-workflow
description: PPDACサイクルに基づくデータ分析ワークフロー。どのフェーズからでも開始可能で、既存の状態を自動検出し最適な開始地点を提案します。各フェーズのエージェント配置・品質ゲート・docs/連携・コミット戦略を定義します。
---

# PPDACワークフロー

## 使い方

このスキルは**どのフェーズからでも開始可能**です。実行時に以下の手順で進めます：

### ステップ1: 現在の状態を検出

以下のファイル・ディレクトリの存在を確認します：
- `docs/analysis_plan/` 内の分析計画ファイル
- SQLクエリファイル（`queries/`, `sql/` など）
- 分析コード（`notebooks/`, `scripts/`, `src/` など）
- `docs/analysis_result/` 内の分析レポート

### ステップ2: ユーザーに開始フェーズを確認

検出した状態に基づき、以下の選択肢を提示します：

| 状態 | 推奨開始フェーズ | 理由 |
|------|----------------|------|
| 何もない | **Phase 1 (Problem)** から | 新規分析の開始 |
| 計画あり | **Phase 3 (Data)** から | データ取得・クエリ作成 |
| データあり | **Phase 4 (Analysis)** から | 分析コード作成・実行 |
| 分析コードあり | **Phase 5 (Conclusion)** から | レポート作成 |
| レポートあり | **レビュー・改善** | 既存分析の見直し |

ユーザーが明示的に「Phase Xから始める」と指定した場合はその通りに進めます。

### ステップ3: 選択したフェーズから実行

必要な前提条件が不足している場合は警告し、代替案を提示します。

---

## フロー概要

```
[Problem] ← 新規分析の場合はここから
  CLAUDE.md・過去分析を読み込み、現状の問題点を把握する
       ↓ 品質ゲート: adversary（Phase 1批評）
       
**[Branch Creation]** ← 必須ステップ
  git checkout -b analysis/<分析名>
       ↓
       
[Plan] ← 計画がない場合はここから
  analysis-planner（人間と協働）→ docs/analysis_plan/<分析名>.md 保存
       ↓ 品質ゲート: adversary（Phase 2批評、最大2回再検証）
       ↓ 人間承認 → analysis-planner 修正 → 再批評
       
[Data] ← 計画がある場合はここから開始可能
  architect → SQLクエリ作成
       ↓ 品質ゲート: チーム形成（sql-reviewer + security-reviewer）
       ↓ クエリ実行・データ取得
       ↓ チーム解散
       
[Analysis / Code] ← データがある場合はここから開始可能
  **分析計画を読む（必須）** → architect → tdd-guide でコード作成
       ↓ 品質ゲート: チーム形成（code-reviewer + python-reviewer + security-reviewer）
       ↓ 人間が分析実施
       ↓ チーム解散
       
[Conclusion] ← 分析が完了している場合はここから開始可能
  **分析計画を読む（必須）** → analysis-reporter → docs/analysis_result/<分析名>.md 保存
       ↓ 品質ゲート: adversary（Phase 5批評、最大2回再検証）
       ↓ 人間承認 → analysis-reporter 修正 → 再批評

[横断] 計画変更 → commit + analysis-planner（docs/analysis_plan/ 更新）
```

---

## フェーズ詳細

### Phase 1: Problem

**目的**: プロジェクトの目的と現状を把握し、今回解くべき問いを特定する

| 項目 | 内容 |
|------|------|
| 前提条件 | なし（常に開始可能） |
| 使用エージェント | `adversary`（Phase 1批評） |
| 人間関与度 | 高（判断・優先度決定） |
| docs/連携 | 読み取りのみ |
| コミット | なし |
| スキップ可否 | ✅ 可（既に問いが明確な場合） |

**実施内容**:

1. `CLAUDE.md` を読み込み、プロジェクトの目的・ビジネス課題を把握する
2. `docs/analysis_result/` の既存レポートを確認し、過去の分析で明らかになったこと・残課題を把握する
3. `docs/analysis_plan/` の既存計画を確認し、進行中・完了済みの分析を把握する
4. 上記をもとに「現時点での問題点・未解明の問い」を整理し、ユーザーに提示する
5. **品質ゲート**: `adversary` で問いの明確性・ビジネス価値を批評

**成果物**:
- 現状の問題点・未解明の問いの整理（テキストサマリー）
- 今回の分析で何を明らかにすべきかの合意
- adversaryによる批評結果（テキスト出力）

---

### ブランチ作成（必須ステップ）

**タイミング**: Phase 1 完了後、Phase 2 開始前

**ブランチ名の形式**:
```
analysis/<分析名>
例: analysis/user-churn-202604
    analysis/ad-effectiveness-q1
```

**作成コマンド**:
```bash
git checkout -b analysis/<分析名>
git branch --show-current  # ブランチ確認
```

**重要事項**:
- すべての分析作業（Phase 2〜5）はこのブランチで実施
- Phase 5 完了・adversary承認後、main にマージ
- このステップはスキップ不可
- ブランチ名には分析対象と期間を含めると管理しやすい

---

### Phase 2: Plan

**目的**: 分析の問いを明確化し、仮説とデータ要件を定義する

| 項目 | 内容 |
|------|------|
| 前提条件 | 分析の目的・問いが明確になっている |
| 使用エージェント | `analysis-planner`, `adversary`（品質ゲート） |
| 人間関与度 | 高（協働作業） |
| docs/連携 | `analysis-planner` が `docs/analysis_plan/<分析名>.md` を直接保存、`adversary` が `docs/analysis_plan/<分析名>_adversary_review.md` を保存 |
| コミット | `docs: create analysis plan for [分析名]` |
| スキップ可否 | ✅ 可（既に計画が存在する場合） |

**実施内容**:

Phase 1 で整理したコンテキストを `analysis-planner` に渡して計画を策定する:

```
# analysis-planner に渡すコンテキスト
- プロジェクト目的（CLAUDE.md より）
- 過去の分析サマリー（docs/analysis_result/ より）
- 現時点で未解決の問い・課題（Phase 1 の成果）
- 今回の分析で何を明らかにしたいか
```

**品質ゲート**:
1. `analysis-planner` が `docs/analysis_plan/<分析名>.md` を保存
2. `adversary` が Phase 2 批評を実施（仮説の検証可能性、因果推論、バイアス等）
3. CRITICAL/HIGH問題がある場合: `analysis-planner` が修正 → `adversary` 再批評（最大2回）
4. 🟢 承認後、Phase 3へ進行

**成果物**:
- 分析目的・仮説（H1, H2, ...）・想定結果
- 必要なデータ・期間・粒度の定義
- `docs/analysis_plan/<分析名>.md`
- `docs/analysis_plan/<分析名>_adversary_review.md`（批評結果）

---

### Phase 3: Data

**目的**: 仮説検証に必要なデータを取得する

| 項目 | 内容 |
|------|------|
| 前提条件 | `docs/analysis_plan/<分析名>.md` が存在（データ要件が定義されている） |
| 使用エージェント | `architect`（クエリ作成前）, `sql-reviewer`（クエリ作成後）, `security-reviewer`（クエリ作成後） |
| 人間関与度 | 中（クエリ確認・実行） |
| docs/連携 | 計画変更があれば `analysis-planner` が `docs/analysis_plan/<分析名>.md` を直接更新 |
| コミット | `feat: add data extraction query for [分析名]` |
| スキップ可否 | ✅ 可（既にデータが取得済みの場合） |

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
| 前提条件 | 分析対象のデータが存在（CSVファイル、DBアクセス等） |
| 使用エージェント | `architect`（コード作成前）, `tdd-guide`（コード作成中）, `code-reviewer`（コード作成後）, `python-reviewer`（コード作成後）, `security-reviewer`（コード作成後） |
| 人間関与度 | 高（分析の解釈・判断） |
| docs/連携 | 計画変更があれば `analysis-planner` が `docs/analysis_plan/<分析名>.md` を直接更新 |
| コミット | `feat: implement analysis code for [分析名]` |
| スキップ可否 | ✅ 可（既に分析が完了している場合） |

**分析計画の参照（必須）**:

分析コード作成時は必ず以下を実施:

1. **計画ファイルを読み込む**: `docs/analysis_plan/<分析名>.md`
2. **目的・仮説を把握**: H1, H2, ... それぞれの検証内容を理解
3. **データ要件を確認**: 対象期間・粒度・除外条件等
4. **想定結果を理解**: 計画時に想定していた結果の方向性

**計画から逸脱する場合**:
- 安易に逸脱しない
- 逸脱が必要な場合は `analysis-planner` で計画を更新してから進む
- 更新理由を明記する

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
| 前提条件 | 分析結果（グラフ、統計量、考察）が得られている |
| 使用エージェント | `analysis-reporter`, `adversary`（品質ゲート）, `refactor-cleaner`（オプション） |
| 人間関与度 | 中（レポート内容の確認・承認） |
| docs/連携 | `analysis-reporter` が `docs/analysis_result/<分析名>.md` を直接保存、`adversary` が `docs/analysis_result/<分析名>_adversary_review.md` を保存 |
| コミット | `docs: add analysis report for [分析名]` |
| スキップ可否 | ❌ 不可（分析の最終成果物） |

**Phase 5 開始前の必須準備**:

1. **分析計画を必ず読む**: 
   - `docs/analysis_plan/<分析名>.md` から目的・仮説（H1, H2, ...）を取得
2. **仮説との対応づけ**:
   - 各仮説（H1, H2, ...）に対する結果を明確に記述
   - 支持/棄却/判断保留を明示
3. **計画との整合性確認**:
   - 分析手法・対象期間・対象範囲が計画と一致するか確認
   - 乖離がある場合は理由を説明

**重要**: 分析レポートは「計画に対する回答」として構造化する。計画を無視した独立したレポートにしない。

**手順**:
1. `analysis-reporter` が `docs/analysis_plan/<分析名>.md` から分析計画を取得し、結果と突き合わせる
2. 分析レポートテンプレートに沿って Conclusion を構造化
3. `analysis-reporter` が `docs/analysis_result/<分析名>.md` として直接保存
4. **品質ゲート**: `adversary` が Phase 5 批評を実施（解釈の妥当性、根拠の強さ、提案の実行可能性等）
5. CRITICAL/HIGH問題がある場合: `analysis-reporter` が修正 → `adversary` 再批評（最大2回）
6. 🟢 承認後、コミット

**成果物**:
- `docs/analysis_result/<分析名>.md`
- `docs/analysis_result/<分析名>_adversary_review.md`（批評結果）

---

## チーム形成による品質ゲートの運用

**レビュワーは必ずチーム形成して同時起動**:

品質ゲート開始時にチームを形成し、レビュー完了後に解散します。

```python
# Phase 3 の品質ゲート
TeamCreate("phase3-review")
Agent(sql-reviewer, team="phase3-review")
Agent(security-reviewer, team="phase3-review")
# レビュー完了後
TeamDelete("phase3-review")

# Phase 4 の品質ゲート
TeamCreate("phase4-review")
Agent(code-reviewer, team="phase4-review")
Agent(python-reviewer, team="phase4-review")
Agent(security-reviewer, team="phase4-review")
# レビュー完了後
TeamDelete("phase4-review")

# Phase 1/2/5 の品質ゲート（チーム不要、単独起動）
Agent(adversary)  # レッドチーム批評
```

### adversary の品質ゲート基準

**承認基準**:
- 🟢 **承認**: MEDIUM以下のみ → 次フェーズへ進行
- 🟡 **条件付き承認**: HIGH問題あり（CRITICAL問題なし） → 人間判断または修正後進行
- 🔴 **要再検討**: CRITICAL問題あり → 修正必須

**再批評フロー**:
1. `adversary` が初回批評
2. CRITICAL/HIGH問題がある場合: 担当エージェントが修正
3. `adversary` が1回目再批評
4. まだCRITICAL/HIGH問題がある場合: 再度修正
5. `adversary` が2回目再批評（最終）
6. 3回目以降は人間が最終判断

各レビュワーからCRITICAL/HIGHが出た場合は修正してから次フェーズへ進む。

## チームのライフサイクル管理

### チーム形成のタイミング

**Phase 3/4 の品質ゲート開始時**:
1. レビュー用チーム名を決定（例: `phase3-review`, `phase4-review`）
2. `TeamCreate` でチーム作成
3. レビュワーエージェントを同時起動
4. 各レビュワーが並行してタスク実施

### チーム解散のタイミング

**すべてのレビュワー完了・品質ゲート通過後**:
1. レビュワーの結果を確認
2. CRITICAL/HIGH問題があれば修正
3. 承認後 `TeamDelete` でチームを解散
4. 次フェーズへ進行

**例: Phase 3 のライフサイクル**:
```
Phase 3 開始 
  → TeamCreate("phase3-review") 
  → sql-reviewer + security-reviewer 起動
  → レビュー完了・問題修正
  → 品質ゲート承認
  → TeamDelete("phase3-review") 
  → Phase 4 へ
```

**Phase 1/2/5 はチーム不要**: adversary が単独で実施するため、チーム形成・解散は不要。

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
