# Claude Code Orchestration Framework - Getting Started

新規プロジェクトでオーケストレーションフレームワークを使用するためのガイド。

---

## 概要

このフレームワークは、Claude Codeを使用したマルチエージェント協調作業を標準化します。

### 主要機能

| 機能 | 説明 |
|------|------|
| エージェントロール | 7つの標準ロール（pm, designer, reviewer等） |
| Wave実行 | 依存関係を考慮した並列タスク実行 |
| Agent Teams | 複数インスタンスの協調作業 |
| 共有学習 | エージェント間のナレッジ共有 |
| コスト管理 | モデル選択とコスト最適化 |

---

## クイックスタート（既存プロジェクト）

### 1. エージェント起動

```
/spawn-agent {project}-pm "タスク説明"
```

### 2. 並列タスク実行

```
/wave-execute "3タスクを並列実行"
```

### 3. チーム協調作業

```
/start-team "多角的レビュー"
```

---

## 新規プロジェクト立ち上げ

### Step 1: プロジェクト設定の編集

`CLAUDE.md` のプロジェクト設定セクションを編集:

```markdown
## プロジェクト設定

| 項目 | 値 |
|------|-----|
| プロジェクト名 | {your-project} |
| クライアント | {クライアント名} |
| 対象業務 | {業務内容} |
| 技術基盤 | {使用技術} |
| 目的 | {プロジェクト目的} |
```

### Step 2: 共通設定の編集

`/.claude/agents/_common.md` のプロジェクト概要セクションを編集:

```markdown
## プロジェクト概要

- **プロジェクト名**: {your-project}
- **クライアント**: {クライアント名}
- **対象業務**: {業務内容}
- **技術基盤**: {使用技術}
- **目的**: {プロジェクト目的}
```

### Step 3: エージェント定義の作成

1. プロジェクトフォルダを作成:
   ```bash
   mkdir -p .claude/agents/project/{your-project}
   ```

2. テンプレートをコピー:
   ```bash
   cp .claude/agents/templates/agent-template.md \
      .claude/agents/project/{your-project}/{your-project}-pm.md
   ```

3. プレースホルダーを編集:
   - `{project}` → 実際のプロジェクト名
   - `{role}` → ロール名（pm, designer等）
   - 責務・タスクをプロジェクト固有に更新

### Step 4: learnings構造の準備

```bash
mkdir -p learnings/project/{your-project}/phase-a
```

### Step 5: 開始

```
/spawn-agent {your-project}-pm "プロジェクト開始"
```

---

## ディレクトリ構造

```
{project}/
├── CLAUDE.md                          ← エントリーポイント
├── .claude/
│   ├── agents/
│   │   ├── _common.md                 ← 共通設定（テンプレート変数付き）
│   │   ├── _roles.md                  ← 標準ロール定義
│   │   ├── templates/
│   │   │   ├── spawn-template.md
│   │   │   └── agent-template.md      ← 新規エージェント作成用
│   │   └── project/
│   │       └── {project}/             ← プロジェクト固有エージェント
│   │           ├── {project}-pm.md
│   │           └── ...
│   ├── commands/                      ← スラッシュコマンド
│   │   ├── spawn-agent.md
│   │   ├── start-team.md
│   │   ├── wave-execute.md
│   │   ├── observability.md
│   │   └── cost-track.md
│   └── orchestration-guide.md         ← オーケストレーションガイド
├── docs/
│   ├── framework/                     ← フレームワークドキュメント
│   │   └── getting-started.md         ← このファイル
│   └── project/
│       └── {project}/                 ← プロジェクト固有ドキュメント
├── learnings/
│   ├── shared-learnings.md            ← 共有学習
│   ├── domains/                       ← 技術ドメイン別ナレッジ
│   │   └── {technology}/
│   └── project/
│       └── {project}/                 ← プロジェクト固有learnings
│           └── phase-a/
└── logs/
    ├── agent-activity.md              ← 稼働ログ
    └── cost-tracking.md               ← コスト追跡
```

---

## 標準ロール

| ロール | 責務 | 典型タスク |
|--------|------|-----------|
| pm | プロジェクト管理 | 進捗レビュー、Wave計画 |
| designer | 設計 | UI/UX設計、帳票設計 |
| reviewer | 品質保証 | レビュー、テスト実行 |
| researcher | 調査 | 業界調査、法規制確認 |
| engineer | 実装 | 技術設計、PoC |
| integrator | 統合 | 成果物統合、調整 |
| checker | 検証 | 最終チェック |

**詳細**: `/.claude/agents/_roles.md`

---

## スラッシュコマンド

| コマンド | 用途 | 例 |
|---------|------|-----|
| `/spawn-agent` | エージェント起動 | `/spawn-agent {project}-pm "進捗確認"` |
| `/start-team` | Agent Teams起動 | `/start-team "並列レビュー"` |
| `/wave-execute` | Wave並列実行 | `/wave-execute "Phase 1タスク"` |
| `/observability` | 監視ダッシュボード | `/observability start` |
| `/cost-track` | コスト記録 | `/cost-track` |

---

## Wave実行パターン

### 基本構造

```
Wave 1（並列）: 依存関係のないタスク
├── Task A
├── Task B
└── Task C

Wave 2（並列）: Wave 1の結果を使用
├── Task D
└── Task E

Wave 3: 統合
└── Task F（全結果を統合）
```

### 典型パターン

| パターン | 構成 |
|---------|------|
| 設計→実装→検証 | designer → engineer → reviewer |
| 並列レビュー | reviewer × 3 → integrator |
| 調査→設計 | researcher → designer |

---

## モデル選択

| タスク複雑度 | モデル | コスト |
|-------------|--------|--------|
| 軽量（ログ、定型） | Haiku | 低 |
| 標準（実装、レビュー） | Sonnet | 中 |
| 複雑（設計、アーキテクチャ） | Opus | 高 |

---

## ベストプラクティス

### エージェント運用

1. **定義ファイル必読**: 起動時に必ずエージェント定義を読み込む
2. **shared-learnings参照**: 過去の学びを活用する
3. **稼働ログ記録**: タスク完了後にログを追記
4. **学びの追記**: 新たな発見はshared-learningsに記録

### Wave実行

1. **依存関係の分析**: 独立タスクを同一Waveに
2. **並列数の制限**: 5-6エージェント以下を推奨
3. **統合Waveの設置**: 複数成果物は必ず統合

### コスト最適化

1. **Haikuの活用**: 軽量タスクにHaikuを使用
2. **定期的な確認**: `/cost`でコスト確認
3. **コンテキストクリア**: タスク切り替え時に`/clear`

---

## トラブルシューティング

### エージェントが定義を読まない

**原因**: プロンプトに読み込み指示がない

**解決**: spawn-agent.mdのテンプレートに従い、必須読み込みを明記

### Wave実行でレート制限

**原因**: 並列エージェント数が多すぎる

**解決**: 並列数を4-5に削減、または複数Waveに分割

### 学びが蓄積されない

**原因**: タスク完了後の追記を忘れている

**解決**: PMがWave完了時に学びの追記を確認

---

## 関連ドキュメント

| ドキュメント | パス |
|-------------|------|
| オーケストレーションガイド | `/.claude/orchestration-guide.md` |
| ロール定義 | `/.claude/agents/_roles.md` |
| コスト管理 | `/docs/cost-management-guide.md` |
| Agent Teams | `/docs/agent-teams-guide.md` |
| タスク分割 | `/docs/task-splitting-guide.md` |

---

**最終更新**: 2026-02-28
