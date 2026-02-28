---
name: start-team
description: Start an Agent Teams session for collaborative work. Use when multiple agents need to discuss and coordinate.
disable-model-invocation: true
argument-hint: [team-purpose]
---

# Agent Teams 起動スキル

複数のClaude Codeインスタンスが協調作業を行うAgent Teamsセッションを開始します。

## 使用方法

```
/start-team "PRDの多角的レビュー"
/start-team "認証方式の比較調査"
```

## 起動手順

### 1. チーム作成

自然言語でリクエスト:

```
チームを作成してください:
- チームメイト1: [役割1]
- チームメイト2: [役割2]
- チームメイト3: [役割3]

各チームメイトは以下を参照してください:
- /.claude/agents/_common.md
- /learnings/shared-learnings.md
```

### 2. キーボード操作

| 操作 | キー |
|------|------|
| チームメイト選択 | `Shift+Up/Down` |
| タスクリスト表示 | `Ctrl+T` |
| Delegateモード | `Shift+Tab` |

### 3. チーム終了

```
チームメイトをシャットダウンして、チームをクリーンアップしてください
```

## Subagentsとの使い分け

| 状況 | 推奨 |
|------|------|
| 独立した並列タスク | Subagents（Wave方式） |
| 相互議論・協調が必要 | Agent Teams |
| コスト重視 | Subagents |
| 品質・深い分析重視 | Agent Teams |

## 活用シナリオ

### 並列レビュー
```
チーム構成:
├── Lead: {project}-pm
├── Teammate 1: セキュリティレビュー
├── Teammate 2: 設計レビュー
└── Teammate 3: 法規制レビュー
```

### 競合仮説調査
```
チーム構成:
├── Lead: {project}-pm
├── Teammate 1: 仮説A調査
├── Teammate 2: 仮説B調査
└── Teammate 3: 仮説C調査
```

## ベストプラクティス

- チームサイズ: 3-4人（最大5人）
- Delegateモードでリードのコンテキスト消費を抑制
- 複雑なタスクではPlan承認を要求

## 制限事項

| 制限 | 説明 |
|------|------|
| セッション再開 | `/resume`でチームメイト復元不可 |
| 1セッション1チーム | 同時複数チーム不可 |
| ネストなし | チームメイトは子チーム作成不可 |

## 振り返りチェックリスト

セッション終了時:

```
□ チームサイズは適切だったか
□ 直接通信は有効活用されたか
□ Subagentsで代替可能だったか
□ 学びを shared-learnings.md に記録したか
```

## 関連ドキュメント

- 詳細ガイド: `/docs/agent-teams-guide.md`
- コスト管理: `/docs/cost-management-guide.md`
