# Claude Orchestration Framework

Claude Codeでマルチエージェント協調作業を行うための汎用オーケストレーションフレームワーク。

## 特徴

- **標準化されたロール定義**: PM, Designer, Engineer, Reviewer等の役割テンプレート
- **Wave実行モデル**: 並列・順次タスク実行の標準パターン
- **コスト最適化**: モデル選択ガイドとコスト管理
- **学習の蓄積**: shared-learnings.mdによるナレッジ共有
- **スラッシュコマンド**: `/spawn-agent`, `/start-team`, `/wave-execute`等

## インストール

### git submoduleとして追加（推奨）

```bash
cd your-project
git submodule add https://github.com/sasaki-sketch/claude-orchestration-framework.git .claude/framework
git commit -m "Add claude-orchestration-framework as submodule"
```

### 既存リポジトリのクローン時

```bash
git clone --recurse-submodules <your-repo>

# または既存クローン後
git submodule update --init --recursive
```

### submodule更新

```bash
cd .claude/framework
git pull origin main
cd ../..
git add .claude/framework
git commit -m "Update orchestration framework"
```

## プロジェクト設定

### 1. プロジェクトのCLAUDE.mdで参照

```markdown
# My Project

## フレームワーク

オーケストレーションフレームワーク: `.claude/framework/CLAUDE.md`

## プロジェクト設定

| 項目 | 値 |
|------|-----|
| プロジェクト名 | my-project |
| 技術基盤 | ... |
```

### 2. プロジェクト固有設定を追加

```
your-project/
├── CLAUDE.md                      # プロジェクトエントリーポイント
├── .claude/
│   ├── framework/                 # ← このsubmodule
│   ├── agents/
│   │   ├── _common.md             # プロジェクト固有設定
│   │   └── project/
│   │       └── myproject/         # プロジェクト固有エージェント
│   │           ├── myproject-pm.md
│   │           └── myproject-designer.md
│   └── commands/                  # frameworkからシンボリックリンク
└── learnings/
    └── shared-learnings.md        # プロジェクト固有学習
```

### 3. コマンドの有効化

```bash
# シンボリックリンクで有効化（オプション）
ln -s .claude/framework/.claude/commands .claude/commands
```

または、プロジェクトの `.claude/commands/` にコピーして使用。

## クイックスタート

### エージェント起動

```
/spawn-agent myproject-pm "タスク説明"
```

### Wave実行

```
/wave-execute
```

### Agent Teams起動

```
/start-team
```

## ドキュメント

- [Getting Started](docs/framework/getting-started.md) - 新規プロジェクト立ち上げ
- [Orchestration Guide](.claude/orchestration-guide.md) - 統括ガイド
- [Agent Roles](.claude/agents/_roles.md) - ロール定義
- [Cost Management](docs/framework/cost-management-guide.md) - コスト管理

## ディレクトリ構造

```
claude-orchestration-framework/
├── CLAUDE.md                    # エントリーポイント
├── README.md                    # このファイル
├── .claude/
│   ├── commands/                # スラッシュコマンド
│   ├── agents/                  # ロール・テンプレート
│   └── orchestration-guide.md   # 統括ガイド
├── docs/framework/              # ガイドライン
└── learnings/                   # ナレッジテンプレート
```

## ライセンス

MIT License - see [LICENSE](LICENSE)

## 作成者

Molvia Inc.
