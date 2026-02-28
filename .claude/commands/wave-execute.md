---
name: wave-execute
description: Execute tasks in Wave model with parallel subagents. Use when running multiple independent tasks.
disable-model-invocation: false
allowed-tools: Task, TaskCreate, TaskUpdate, TaskList
argument-hint: [wave-description]
---

# Wave実行モデル スキル

並列実行可能なタスクをWaveとしてグループ化し、効率的に実行します。

## 使用方法

```
/wave-execute "Phase 1の3タスクを並列実行"
/wave-execute "6エージェントでドキュメントレビュー"
```

## Wave実行の基本構造

```
Wave N: 並列タスク群（依存関係なし）
├── Task A
├── Task B
└── Task C

Wave N+1: Wave N完了後のタスク
└── Task D（A, B, Cの結果を統合）
```

## 実行手順

### 1. タスク分析

タスクを依存関係で分類:

```
□ 独立タスク → 同一Waveで並列実行
□ 依存タスク → 後続Waveで実行
```

### 2. Wave構成

```javascript
// Wave 1: 並列実行（最大5-6エージェント推奨）
[
  { role: "{project}-designer", task: "帳票分析", model: "sonnet" },
  { role: "{project}-researcher", task: "法規制調査", model: "sonnet" },
  { role: "{project}-engineer", task: "技術調査", model: "sonnet" }
]

// Wave 2: 統合（Wave 1完了後）
[
  { role: "{project}-integrator", task: "結果統合", model: "opus" }
]
```

### 3. 起動

Task toolを使用して並列起動:

```
subagent_type: "general-purpose"
model: "sonnet" または "haiku"（軽量タスク）
prompt: [spawn-templateに従う]
```

## モデル選択ガイド

| タスク | モデル | コスト比 |
|--------|--------|---------|
| ログ更新、定型作業 | haiku | 1x |
| 実装、レビュー | sonnet | 5x |
| 設計、アーキテクチャ | opus | 15x |

## 並列実行の制約

| 項目 | 推奨値 |
|------|--------|
| 最大並列数 | 5-6エージェント |
| レート制限 | 6エージェント超でリスク増 |
| タイムアウト | 10分以上のタスクは分割 |

## エラーハンドリング

### レート制限発生時

```
1. 失敗したタスクを記録
2. 次のWaveで再実行
3. または手動で対応
```

### タスク失敗時

```
1. 失敗原因を確認
2. 依存タスクをブロック
3. 修正後に再実行
```

## コスト最適化

- 軽量タスクにHaikuを積極活用
- 並列数を3-4に抑えてコスト削減
- `/cost`で定期的に確認

## Wave実行チェックリスト

```
□ タスクの依存関係を分析したか
□ 適切なモデルを選択したか
□ 並列数は5-6以下か
□ エラーハンドリングを考慮したか
□ 完了後に結果を統合したか
```

## 関連ドキュメント

- オーケストレーション: `/.claude/orchestration-guide.md`
- モデル選択: `/docs/model-selection-guide.md`
- コスト管理: `/docs/cost-management-guide.md`
