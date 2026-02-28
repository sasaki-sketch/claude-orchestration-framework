---
name: observability
description: Start or stop the observability dashboard for monitoring agent activity. Use when debugging or analyzing agent behavior.
disable-model-invocation: true
allowed-tools: Bash
argument-hint: [start|stop|status]
---

# オブザーバビリティ スキル

エージェント活動を監視するダッシュボードの起動・停止を行います。

## 使用方法

```
/observability start   # ダッシュボード起動
/observability stop    # ダッシュボード停止
/observability status  # 状態確認
```

## 起動手順

### ダッシュボード起動

```bash
./scripts/start-observability.sh
```

起動後、以下でアクセス:
- **ダッシュボード**: http://localhost:5173
- **APIサーバー**: http://localhost:4000

### ダッシュボード停止

```bash
./scripts/stop-observability.sh
```

## 監視できる情報

| 情報 | 説明 |
|------|------|
| ツール使用 | どのツールがいつ呼ばれたか |
| エージェント活動 | subagentの起動・完了 |
| エラー | ツール実行エラー |
| タイムライン | 時系列での活動可視化 |

## フック設定

`/.claude/settings.json`で12種類のフックが設定済み:

| フック | タイミング |
|--------|----------|
| PreToolUse | ツール実行前 |
| PostToolUse | ツール実行後 |
| Notification | 通知発生時 |
| Stop | セッション終了時 |

## トラブルシューティング

### ダッシュボードが起動しない

```bash
# ポート確認
lsof -i :4000
lsof -i :5173

# プロセス強制終了
pkill -f "bun run"
```

### フックが動作しない

```bash
# uv確認
uv --version

# 依存関係確認
cd .claude/hooks && uv sync
```

## 前提条件

| ツール | 用途 |
|--------|------|
| uv | Pythonパッケージ管理（フック実行） |
| bun | サーバー・クライアント実行 |

### インストール（未設定の場合）

```bash
# uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# bun
curl -fsSL https://bun.sh/install | bash
```

## ディレクトリ構造

```
.claude/
├── hooks/
│   ├── hook_callback.py    # フックスクリプト
│   └── pyproject.toml
└── settings.json           # フック設定

scripts/
├── start-observability.sh  # 起動スクリプト
└── stop-observability.sh   # 停止スクリプト
```

## 関連ドキュメント

- セットアップガイド: `/docs/observability-setup.md`
- 設定ファイル: `/.claude/settings.json`
