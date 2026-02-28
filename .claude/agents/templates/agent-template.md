---
name: {project}-{role}
description: |
  {ロールの説明}
  Use this agent when:
  - {ユースケース1}
  - {ユースケース2}
  <example>「{タスク例1}」</example>
  <example>「{タスク例2}」</example>
model: sonnet
tools:
  - Read
  - Write
  - Edit
  - Glob
---

# {project}-{role} エージェント

**役割**: {ロールの説明}

## 起動時の必須アクション

1. **読み込み必須ファイル**
   - `/.claude/agents/_common.md` - 共通設定
   - `/learnings/shared-learnings.md` - 共有学習

2. **タスク完了後の必須アクション**
   - `/logs/agent-activity.md` に稼働記録を追記
   - 学びがあれば `/learnings/shared-learnings.md` に追記

## 責務

1. **{責務1}**: {説明}
2. **{責務2}**: {説明}
3. **{責務3}**: {説明}

## 典型タスク

- {タスク1}
- {タスク2}
- {タスク3}

## 連携エージェント

| エージェント | 連携内容 |
|-------------|---------|
| {project}-pm | {連携内容} |
| {project}-{role2} | {連携内容} |

## 出力フォーマット

```markdown
## {成果物タイトル}: YYYY-MM-DD

### サマリー
- {項目1}: {内容}
- {項目2}: {内容}

### 詳細
{詳細内容}

### 次のアクション
1. {アクション1}
```

## 参照

- `_common.md` - 共通設定
- `_roles.md` - 標準ロール定義
- `shared-learnings.md` - 必須読み込み

---

<!--
テンプレート使用方法:
1. このファイルを /.claude/agents/project/{project}/ にコピー
2. ファイル名を {project}-{role}.md に変更
3. {project}, {role}, その他プレースホルダーを置換
4. プロジェクト固有の責務・タスクを追記
-->
