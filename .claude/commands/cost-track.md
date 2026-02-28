---
name: cost-track
description: Record session cost to tracking log. Use at end of session or after major tasks.
disable-model-invocation: true
allowed-tools: Bash, Read, Edit
argument-hint: [session-name]
---

# コスト記録スキル

セッションのコストを `/logs/cost-tracking.md` に記録します。

## 使用方法

```
/cost-track "Phase 5 Skills移行"
/cost-track "PRDレビュー"
```

## 実行手順

### 1. コスト確認

```
/cost
```

出力例:
```
Total cost:            $0.55
Total duration (API):  6m 19.7s
Total duration (wall): 6h 33m 10.2s
Total code changes:    0 lines added, 0 lines removed
```

### 2. ログに追記

`/logs/cost-tracking.md` に以下を追記:

```markdown
### [日付] [セッション名]

| 指標 | 値 |
|------|-----|
| Total cost | $X.XX |
| API duration | Xm Xs |
| Wall duration | Xh Xm |
| Lines added | X |
| Lines removed | X |

**主なタスク**:
- [タスク1]
- [タスク2]
```

## 記録タイミング

- セッション終了時（必須）
- 大きなタスク完了後（推奨）
- コストが$5を超えた時（警告）

## コスト目安

| 状況 | アクション |
|------|-----------|
| < $3 | 正常 |
| $3-5 | 注意（/compact検討） |
| $5-10 | 警告（原因分析） |
| > $10 | 要改善（レビュー） |

## 関連ドキュメント

- コスト管理ガイド: `/docs/cost-management-guide.md`
- 記録ログ: `/logs/cost-tracking.md`
