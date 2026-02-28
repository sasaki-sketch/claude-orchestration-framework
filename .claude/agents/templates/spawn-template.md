# エージェント起動テンプレート

**目的**: エージェント定義ファイルの確実な活用と稼働追跡を保証する

---

## 技術的制約

Claude Code の Task tool は以下の `subagent_type` のみ使用可能:
- "Bash", "general-purpose", "statusline-setup", "Explore", "Plan", "claude-code-guide"

**カスタムエージェント定義（mutsubi-*.md）を subagent_type で直接指定することは不可能**

→ `subagent_type: "general-purpose"` + プロンプトで定義ファイル参照を指示する

---

## 標準プロンプト構成（必須要素）

### 1. 定義ファイル読み込み指示（必須）

```
まず以下のファイルを読んでください:
- /.claude/agents/mutsubi-{role}.md（エージェント定義）
- /.claude/agents/_common.md（共通設定）
- /learnings/shared-learnings.md（共有学習）
```

### 2. タスク指示（必須）

```
上記を読んだ上で、以下のタスクを実行してください:
[具体的なタスク内容]
```

### 3. 完了報告指示（必須）

```
タスク完了後、以下を報告してください：
- 成果物のファイルパス
- ステータス（完了/部分完了/失敗）
- 定義ファイルを読んだか（Yes/No）

※ 稼働ログへの記録はオーケストレーターが行います。時刻やWave番号は記録不要です。
```

---

## 完全なプロンプト例

### mutsubi-pm 起動例

```
## 必須読み込み
まず以下のファイルを読んでください:
1. /.claude/agents/mutsubi-pm.md（エージェント定義）
2. /.claude/agents/_common.md（共通設定）
3. /learnings/shared-learnings.md（共有学習）

## タスク
上記を読んだ上で、以下のタスクを実行してください:
- Phase 1の進捗をレビューしてください
- 残タスクと次のアクションを特定してください

## 完了報告
タスク完了後、以下を報告してください：
- 成果物のファイルパス
- ステータス（完了/部分完了/失敗）
- 定義ファイルを読んだか（Yes/No）
```

### mutsubi-designer 起動例

```
## 必須読み込み
まず以下のファイルを読んでください:
1. /.claude/agents/mutsubi-designer.md（エージェント定義）
2. /.claude/agents/_common.md（共通設定）
3. /learnings/shared-learnings.md（共有学習）

## タスク
上記を読んだ上で、以下のタスクを実行してください:
- 駐車場申請書の帳票分析を行ってください
- 出力は forms-analysis.md に追記してください

## 完了報告
タスク完了後、以下を報告してください：
- 成果物のファイルパス
- ステータス（完了/部分完了/失敗）
- 定義ファイルを読んだか（Yes/No）

学びがあれば /learnings/shared-learnings.md にも追記してください。
```

---

## プロンプトチェックリスト

エージェント起動前に確認:

```
□ subagent_type は "general-purpose" か？
□ エージェント定義ファイル読み込み指示があるか？
□ _common.md 読み込み指示があるか？
□ shared-learnings.md 参照指示があるか？
□ 具体的なタスク内容が記載されているか？
□ 完了報告指示があるか？（成果物、ステータス、定義参照の報告）
```

※ 稼働ログ記録はオーケストレーターが行う。エージェントには時刻・Wave番号を記録させない。

---

## 禁止事項

| 禁止 | 理由 | 正しい方法 |
|------|------|----------|
| 定義ファイル読み込み指示を省略 | 定義ファイルが参照されない | 定義ファイル読み込み指示を含める |
| subagent_type に "mutsubi-pm" を指定 | 技術的に不可能 | "general-purpose" を使用 |
| shared-learnings.md 参照を省略 | 過去の学びが活用されない | 必ず参照指示を含める |
| エージェントに稼働ログ記録を指示 | 時刻が不正確になる | オーケストレーターが記録する |
| エージェントに時刻を記録させる | 並列実行時に矛盾が発生 | Wave番号で管理する |

---

## モデル選択（コスト最適化）

### 判断基準

| タスク複雑度 | 推奨モデル | 指定方法 |
|-------------|-----------|---------|
| 軽量（ログ更新、定型作業） | Haiku | `"model": "haiku"` |
| 標準（実装、レビュー） | Sonnet | 省略可 |
| 複雑（設計、アーキテクチャ） | Opus | `"model": "opus"` |

### Haiku適用可能なタスク例

- shared-learnings.md への学び追記
- 稼働ログの更新
- 単純なファイル修正（パス修正、タイポ修正）
- ステータス確認のみのタスク

### spawn時のモデル指定

```javascript
{
  "subagent_type": "general-purpose",
  "model": "haiku",  // コスト重視の軽量タスク
  "prompt": "..."
}
```

### プロンプトチェックリスト（モデル選択追加）

```
□ タスクの複雑度を評価したか？
□ 軽量タスクにHaikuを指定したか？
□ 複雑なタスクにのみOpusを使用しているか？
```

**詳細な判断フローとHaikuの限界**: `/docs/model-selection-guide.md` 参照

---

## Task tool 呼び出し例

### 標準（Sonnet）

```javascript
{
  "subagent_type": "general-purpose",
  "prompt": `あなたは mutsubi-pm エージェントです。

## 必須読み込み
まず以下のファイルを読んでください:
1. /.claude/agents/mutsubi-pm.md
2. /.claude/agents/_common.md
3. /learnings/shared-learnings.md

## タスク
[具体的なタスク内容]

## 完了後
/logs/agent-activity.md に稼働記録を追記してください。`,
  "description": "PM: 進捗レビュー"
}
```

### 軽量タスク（Haiku）

```javascript
{
  "subagent_type": "general-purpose",
  "model": "haiku",
  "prompt": `## 必須読み込み
まず以下のファイルを読んでください:
1. /.claude/agents/_common.md

## タスク
/learnings/shared-learnings.md に今日の学びを追記してください:
- [学びの内容]

## 完了報告
- 追記した内容の要約
- ステータス`,
  "description": "学び更新（軽量）"
}
```

---

**作成日**: 2026-02-16
**更新日**: 2026-02-16（モデル選択セクション追加）
