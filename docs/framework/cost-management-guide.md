# mutsubi-dx コスト管理ガイド

**作成日**: 2026-02-16
**目的**: Claude Codeのトークン使用量を最適化し、コスト効率の良いエージェント運用を実現

---

## コスト構造の理解

### トークン消費の内訳

| 消費元 | 入力トークン | 出力トークン |
|--------|-------------|-------------|
| システムプロンプト | 多 | - |
| CLAUDE.md読み込み | 多 | - |
| ファイル読み込み | 多 | - |
| MCP/ツール定義 | 中 | - |
| 会話履歴 | 累積 | - |
| Claude応答 | - | 多 |
| Extended Thinking | - | 多 |

### コスト目安

| 指標 | 値 |
|------|-----|
| 平均コスト | $6/開発者/日 |
| 90%のユーザー | $12/日以下 |
| 月間目安（Sonnet） | $100-200/開発者 |

### サブエージェントのコスト

> "Agent teams use approximately 7x more tokens than standard sessions"

サブエージェント（Task tool）は**独自のコンテキストウィンドウ**を持つため、トークン消費が大幅に増加します。

---

## 必須コマンド

### /cost - セッションコスト確認

```
> /cost

Total cost:            $0.55
Total duration (API):  6m 19.7s
Total duration (wall): 6h 33m 10.2s
Total code changes:    0 lines added, 0 lines removed
```

**使用タイミング**:
- セッション開始時（ベースライン確認）
- 重要なタスク完了後
- セッション終了前

### /stats - 使用パターン確認

Pro/Maxユーザー向けの使用状況確認コマンド。

**使用タイミング**:
- 日次の使用パターン把握
- リミット到達予防

### /clear - コンテキストクリア

```
> /clear
```

**使用タイミング**:
- タスク切り替え時（異なるドメインの作業へ移行）
- コンテキストが肥大化したと感じた時
- `/rename` でセッション名を付けてから実行推奨

### /compact - コンテキスト圧縮

```
> /compact Focus on code samples and API usage
```

**使用タイミング**:
- 長時間セッションの中間
- コンテキストウィンドウ上限に近づいた時

**カスタム圧縮指示の例**:
- `Focus on test output and code changes`
- `Preserve API endpoints and data models`
- `Keep error messages and stack traces`

---

## コスト削減戦略

### 1. モデル使い分け

| タスク複雑度 | 推奨モデル | コスト比 |
|------------|-----------|---------|
| 単純（定型作業、軽量タスク） | Haiku | 1x |
| 中程度（実装、レビュー） | Sonnet | 5x |
| 複雑（設計、アーキテクチャ） | Opus | 15x |

詳細は `/docs/model-selection-guide.md` を参照

### 2. コンテキスト管理

| 戦略 | 効果 |
|------|------|
| `/clear` でタスク間をクリア | 不要な履歴の排除 |
| `/compact` で圧縮 | 重要情報の保持 |
| 不要なMCPサーバー無効化 | ツール定義の削減 |

### 3. プロンプト最適化

| 良い例 | 悪い例 |
|--------|--------|
| 「auth.tsのlogin関数に入力検証を追加」 | 「コードベースを改善して」 |
| 「TC-01〜TC-05のテストを実行」 | 「全部テストして」 |

### 4. サブエージェント最適化

| 戦略 | 効果 |
|------|------|
| 軽量タスクにHaiku使用 | コスト30-50%削減 |
| チームサイズを最小限に | 並列消費の抑制 |
| 明確なspawnプロンプト | 無駄な探索の削減 |

---

## 日次運用チェックリスト

### セッション開始時

```
□ /cost でベースライン確認
□ 前回セッションの続きか？ → /resume 検討
□ 新規タスクか？ → /clear でクリーンスタート
```

### タスク完了時

```
□ /cost で消費確認
□ 次のタスクと関連あり？
  - Yes → 継続
  - No → /clear
```

### セッション終了時

```
□ /cost で最終コスト確認
□ /rename でセッションに名前付け
□ 重要な学びがあれば shared-learnings.md に追記
```

---

## アラート閾値

### 即時対応

| 状況 | アクション |
|------|-----------|
| セッションコスト $5超過 | `/compact` 実行 |
| サブエージェント3回連続失敗 | 手動対応に切り替え |
| リミット警告表示 | 優先タスクのみ実行 |

### 予防対応

| 状況 | アクション |
|------|-----------|
| 並列5エージェント以上 | 3-4に抑制 |
| 1タスクに10分以上 | タスク分割を検討 |
| Extended Thinking多用 | 予算設定を検討 |

---

## Extended Thinking予算管理

### Extended Thinkingとは

Claude Opus 4.6で利用可能な深い推論機能。複雑な問題に対してより正確な回答を生成するが、**思考トークンは出力トークンとして課金**される。

### デフォルト設定

- **デフォルト予算**: 31,999トークン（有効時）
- **課金**: 出力トークンとして計算

### 予算調整方法

**1. /config で無効化**

```
> /config
→ Extended Thinking を OFF に設定
```

**2. 環境変数で予算制限**

```bash
export MAX_THINKING_TOKENS=8000  # 予算を8,000トークンに制限
```

**3. /model でエフォートレベル調整（Opus 4.6）**

```
> /model
→ effort level を low/medium/high で選択
```

### 推奨設定

| タスク種別 | 推奨設定 |
|-----------|---------|
| 軽量タスク | Extended Thinking OFF |
| 標準タスク | デフォルト or 8,000トークン |
| 複雑な推論 | 31,999トークン（フル） |
| アーキテクチャ設計 | 50,000-100,000トークン |

### コスト影響

Extended Thinking有効時のコスト増加目安:
- **低予算（8K）**: 約20-30%増
- **デフォルト（32K）**: 約50-100%増
- **高予算（100K）**: 約200-300%増

---

## 参考リンク

- [Manage costs effectively - Claude Code Docs](https://code.claude.com/docs/en/costs)
- [Understanding usage limits - Claude Help Center](https://support.claude.com/en/articles/11647753-understanding-usage-and-length-limits)

---

**最終更新**: 2026-02-16（Extended Thinkingセクション追加）
