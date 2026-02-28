---
name: spawn-agent
description: Spawn a project agent with proper role definition. Use when starting subagents for tasks.
disable-model-invocation: false
allowed-tools: Task, Read
argument-hint: [role-name] [task-description]
---

# エージェント起動スキル

指定されたロールのエージェントを適切な定義ファイル参照付きで起動します。

## 使用方法

```
/spawn-agent {project}-pm "Phase 1の進捗をレビュー"
/spawn-agent {project}-designer "帳票分析"
/spawn-agent {project}-reviewer "PRDのセキュリティレビュー"
```

## 標準ロール

| ロール | 責務 |
|--------|------|
| pm | プロジェクト管理、統合 |
| designer | 設計、UI/UX |
| reviewer | 品質保証、レビュー |
| researcher | 調査、法規制確認 |
| engineer | 実装、技術設計 |
| integrator | 統合、整合性確認 |
| checker | 成果物検証 |

**詳細**: `/.claude/agents/_roles.md`

## 実行テンプレート

エージェント起動時、以下のプロンプト構成を使用:

```
## 必須読み込み
まず以下のファイルを読んでください:
1. /.claude/agents/project/{project}/{project}-{role}.md（エージェント定義）
2. /.claude/agents/_common.md（共通設定）
3. /learnings/shared-learnings.md（共有学習）

## タスク
上記を読んだ上で、以下のタスクを実行してください:
{task-description}

## 完了報告
タスク完了後、以下を報告してください：
- 成果物のファイルパス
- ステータス（完了/部分完了/失敗）
- 定義ファイルを読んだか（Yes/No）
```

## モデル選択

| タスク複雑度 | モデル |
|-------------|--------|
| 軽量（ログ更新、定型作業） | haiku |
| 標準（実装、レビュー） | sonnet（デフォルト） |
| 複雑（設計、アーキテクチャ） | opus |

## 禁止事項

- 定義ファイル読み込み指示の省略
- shared-learnings.md参照の省略
- subagent_typeに"{project}-*"を直接指定（技術的に不可）

## 関連ドキュメント

- 詳細テンプレート: `/.claude/agents/templates/spawn-template.md`
- モデル選択ガイド: `/docs/model-selection-guide.md`
- ロール定義: `/.claude/agents/_roles.md`
