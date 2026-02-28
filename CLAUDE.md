# Claude Code Orchestration Framework

マルチエージェント協調作業のための汎用オーケストレーションフレームワーク。

---

## 概要

このフレームワークは、Claude Codeでマルチエージェント協調作業を行うための標準化されたパターン・テンプレート・ガイドラインを提供します。

### 特徴

- **標準化されたロール定義**: PM, Designer, Engineer, Reviewer等の役割テンプレート
- **Wave実行モデル**: 並列・順次タスク実行の標準パターン
- **コスト最適化**: モデル選択ガイドとコスト管理
- **学習の蓄積**: shared-learnings.mdによるナレッジ共有

---

## Skills（スラッシュコマンド）

| コマンド | 用途 |
|---------|------|
| `/spawn-agent` | エージェント起動（定義ファイル参照付き） |
| `/start-team` | Agent Teams起動（協調作業） |
| `/wave-execute` | Wave実行（並列タスク） |
| `/observability` | 監視ダッシュボード起動/停止 |
| `/cost-track` | コスト記録（セッション終了時） |

---

## 標準ロール

| ロール | 責務 |
|--------|------|
| pm | プロジェクト管理、統合、オーケストレーション |
| designer | 設計、UI/UX、帳票分析 |
| reviewer | 品質保証、レビュー、検証 |
| researcher | 調査、法規制確認、業界知識 |
| engineer | 実装、技術設計、PoC |
| integrator | 統合、整合性確認、第二案作成 |
| checker | 成果物検証、チェックリスト実行 |

**ロール詳細**: `.claude/agents/_roles.md`

---

## ディレクトリ構造

```
claude-orchestration-framework/
├── CLAUDE.md                    # このファイル
├── README.md                    # GitHub用（使い方、submodule説明）
├── .claude/
│   ├── commands/                # スラッシュコマンド定義
│   │   ├── spawn-agent.md
│   │   ├── start-team.md
│   │   ├── wave-execute.md
│   │   ├── observability.md
│   │   └── cost-track.md
│   ├── agents/
│   │   ├── _roles.md            # 標準ロール定義
│   │   ├── _common.md           # 共通設定テンプレート
│   │   ├── verification-checklist.md
│   │   └── templates/
│   │       ├── spawn-template.md
│   │       └── agent-template.md
│   └── orchestration-guide.md   # 統括ガイド
├── docs/
│   └── framework/
│       ├── getting-started.md
│       ├── agent-teams-guide.md
│       ├── cost-management-guide.md
│       ├── task-splitting-guide.md
│       ├── error-retry-patterns.md
│       └── model-selection-guide.md
└── learnings/
    ├── templates/
    │   └── learning-template.md
    └── domains/                  # 技術ドメイン別ナレッジ
        └── appsheet/
```

---

## チームメイト必須参照

チームメイトとして起動された場合:

1. `.claude/agents/_common.md` - 共通設定
2. `learnings/shared-learnings.md` - 共有学習（プロジェクト側）
3. 該当するロール定義ファイル

---

## モデル選択

| タスク複雑度 | モデル | 用途例 |
|-------------|--------|--------|
| 軽量 | Haiku | ログ更新、定型作業、設定変更 |
| 標準 | Sonnet | 実装、レビュー、調査 |
| 複雑 | Opus | 設計、アーキテクチャ、意思決定 |

---

## ガイドライン

| ドキュメント | パス |
|-------------|------|
| オーケストレーション | `.claude/orchestration-guide.md` |
| エージェントロール | `.claude/agents/_roles.md` |
| コスト管理 | `docs/framework/cost-management-guide.md` |
| Agent Teams | `docs/framework/agent-teams-guide.md` |
| タスク分割 | `docs/framework/task-splitting-guide.md` |
| エラーリトライ | `docs/framework/error-retry-patterns.md` |
| 新規プロジェクト立ち上げ | `docs/framework/getting-started.md` |

---

## 使い方

### 1. git submoduleとして追加

```bash
git submodule add https://github.com/sasaki-sketch/claude-orchestration-framework.git .claude/framework
```

### 2. プロジェクトのCLAUDE.mdで参照

```markdown
## フレームワーク

オーケストレーションフレームワーク: `.claude/framework/CLAUDE.md`
```

### 3. プロジェクト固有設定

- `.claude/agents/_common.md` をプロジェクトルートに作成（プロジェクト固有設定）
- `.claude/agents/project/{project}/` にプロジェクト固有エージェント定義を配置

詳細: `docs/framework/getting-started.md`

---

**最終更新**: 2026-02-28（初版）
