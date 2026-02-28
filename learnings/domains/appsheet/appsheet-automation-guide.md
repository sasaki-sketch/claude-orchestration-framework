# AppSheet Automation (Bot) ベストプラクティスガイド

> 作成日: 2026-02-16
> 調査者: mutsubi-researcher

## 目次

1. [Event設定のベストプラクティス](#1-event設定のベストプラクティス)
2. [Process (Step) 設定のベストプラクティス](#2-process-step-設定のベストプラクティス)
3. [よくある間違い・トラブルシューティング](#3-よくある間違いトラブルシューティング)
4. [実践的なセットアップ手順](#4-実践的なセットアップ手順)
5. [設定チェックリスト](#5-設定チェックリスト)

---

## 1. Event設定のベストプラクティス

### 1.1 Data change type の使い分け

| Data change type | 発火タイミング | 推奨ユースケース | 注意点 |
|------------------|---------------|-----------------|--------|
| **Adds only** | 新規レコード追加時のみ | 申請受付通知、初期値設定 | 最もシンプル。無限ループの心配なし |
| **Updates only** | 既存レコード更新時のみ | ステータス変更検知、承認処理 | **Condition必須**。無限ループリスク高 |
| **Deletes only** | レコード削除時のみ | 削除通知、関連データのクリーンアップ | 使用頻度低 |
| **（指定なし）** | すべての変更で発火 | 広範囲な監視（非推奨） | 発火頻度が高すぎる |

### 1.2 Condition式の書き方

#### パターン1: Adds onlyの場合（Conditionは通常不要）

```
# 新規申請のみ処理する場合
Data change type: Adds only
Condition: （空白でOK）

# 特定の申請種別のみ処理する場合
Data change type: Adds only
Condition: [申請種別]="契約"
```

#### パターン2: Updates onlyの場合（Condition必須）

```
# ステータスが「承認済み」に変更された時のみ
Condition: AND(
  [_THISROW_BEFORE].[ステータス] <> "承認済み",
  [ステータス] = "承認済み"
)

# 区画番号が変更された時のみ（移動申請）
Condition: [_THISROW_BEFORE].[区画番号] <> [区画番号]
```

**重要**: `[_THISROW_BEFORE]` を使わないと、更新のたびにBotが発火して無限ループになる危険性がある。

#### パターン3: 複数条件の組み合わせ

```
# 契約申請が承認された時のみ
Condition: AND(
  [_THISROW_BEFORE].[ステータス] <> "承認済み",
  [ステータス] = "承認済み",
  [申請種別] = "契約"
)
```

### 1.3 Bypass Security Filters の使いどころ

| 設定 | 動作 | 使用場面 | 注意点 |
|------|------|----------|--------|
| **無効（OFF）** | セキュリティフィルタが適用される | ユーザー操作トリガのBot | 通常はこちら |
| **有効（ON）** | すべてのデータにアクセス可能 | スケジュールBot、管理者処理 | 権限管理に注意 |

**スケジュールBotの注意点**:
- スケジュール実行時、`USEREMAIL()` は `NULL` になる
- セキュリティフィルタが `[メールアドレス]=USEREMAIL()` の場合、Bypass Security Filters を有効化しないとデータにアクセスできない

---

## 2. Process (Step) 設定のベストプラクティス

### 2.1 Send email の設定

#### 基本設定

| 項目 | 設定方法 | 例 |
|------|----------|-----|
| **To** | 列参照またはメールアドレス | `<<[メールアドレス]>>` または `example@example.com` |
| **Subject** | 件名（列参照可） | `【睦備建設】申請を受け付けました（申請ID: <<[申請ID]>>）` |
| **Body** | 本文（列参照可） | テンプレート参照 |

#### Bodyテンプレート例

```
<<[氏名]>> 様

ご申請を受け付けました。

【申請内容】
申請ID: <<[申請ID]>>
申請種別: <<[申請種別]>>
申請日: <<[申請日]>>

承認完了後、改めてご連絡いたします。

---
睦備建設 管理部
```

#### よくあるエラーと対処法

| エラー | 原因 | 対処法 |
|--------|------|--------|
| メールが届かない | Prototypeモード | 有料プランに移行 |
| 列参照が空白 | 列名の記述ミス | `<<[列名]>>` の列名を確認 |
| 特殊文字エラー | メールアドレスの `-` や `+` | Expression Assistantで式入力 |

### 2.2 Set column values の設定

#### 基本設定

```
Action: Data: set the values of some columns in this row
Set these columns:
  - ステータス: "処理中"
  - 更新日時: NOW()
  - 更新者: USEREMAIL()
```

#### 複数列の更新

1つのStepで複数の列を更新できる。効率的な設計のため、関連する列はまとめて更新する。

```
# 承認時の更新例
Set these columns:
  - ステータス: "承認済み"
  - 承認日: TODAY()
  - 承認者: USEREMAIL()
  - 備考: CONCATENATE([備考], "\n承認日時: ", NOW())
```

### 2.3 複数ステップの連鎖

#### 設計原則

1. **順序依存性を明確化**: Step 1の結果をStep 2が参照する場合、順序を守る
2. **エラーハンドリング**: 重要なStepは "Requires confirmation" を有効化
3. **処理時間**: 合計2分以内に収める（タイムアウト対策）

#### 実装例: 承認処理（3ステップ）

```
Process: 承認処理
  Step 1: Set column values in this row
    Set these columns:
      - ステータス: "承認済み"
      - 承認日: TODAY()

  Step 2: Call a webhook
    URL: {{N8N_WEBHOOK_URL}}/create-contract
    Body: {"申請ID": "<<[申請ID]>>", "区画ID": "<<[区画ID]>>"}

  Step 3: Send an email
    To: <<[メールアドレス]>>
    Subject: 【睦備建設】申請が承認されました
    Body: （承認通知テンプレート）
```

### 2.4 Send a notification の設定

#### 基本設定

| 項目 | 設定方法 | 例 |
|------|----------|-----|
| **Table name** | 対象テーブル | `Applications` |
| **To** | 受信者のAppSheetアカウント | `<<[メールアドレス]>>` |
| **Title** | 通知タイトル | `新規申請: <<[申請種別]>>` |
| **Body** | 通知本文 | `申請ID: <<[申請ID]>>` |
| **DeepLink** | アプリ内リンク | `LINKTOFILTEREDVIEW("申請詳細", [申請ID] = [_THISROW].[申請ID])` |

#### メリット

- プッシュ通知で即時性が高い
- DeepLinkで直接該当レコードに遷移可能
- Email より設定項目が少ない

#### 制約

- Prototypeではアプリオーナーのみに通知
- デプロイ後は指定された受信者に通知

---

## 3. よくある間違い・トラブルシューティング

### 3.1 Bot が発火しない原因

| 原因 | 確認方法 | 対処法 |
|------|----------|--------|
| **1. データソースに直接変更** | データをAppSheet以外で変更したか | AppSheet経由でデータを変更（例外: AppSheet database events） |
| **2. Condition式が常にFALSE** | Audit History で "Condition = FALSE" | 式を見直す、テストデータで確認 |
| **3. Event Type の設定ミス** | Adds only なのに Update操作 | Event Typeと操作を一致させる |
| **4. 有料プラン未契約** | Prototypeモードか | 有料プランに移行（Schedule、Email等） |
| **5. Security Filter制約** | Bypass Security Filtersが無効 | スケジュールBotでは有効化 |

### 3.2 無限ループの回避

#### 発生パターン

```
# 危険な設定例（無限ループ）
Event Type: Updates only
Condition: [ステータス] = "処理中"  ← _THISROW_BEFOREがない
Process:
  Step 1: Set column values
    Set these columns:
      - 更新日時: NOW()  ← これが更新されると再びBotが発火
```

#### 正しい設定

```
# 安全な設定例
Event Type: Updates only
Condition: AND(
  [_THISROW_BEFORE].[ステータス] <> "処理中",  ← 変更前後を比較
  [ステータス] = "処理中"
)
Process:
  Step 1: Set column values
    Set these columns:
      - 更新日時: NOW()  ← ステータスが変わらなければ発火しない
```

#### 無限ループ防止のチェックリスト

- [ ] `Updates only` 使用時に `[_THISROW_BEFORE]` を使っているか
- [ ] Condition式が「値が変更された時のみTRUE」になるか
- [ ] Set column values で更新する列がConditionに影響しないか

### 3.3 タイムアウト対策

| 原因 | 制限 | 対処法 |
|------|------|--------|
| 処理時間超過 | Add/Update/Delete + Bot実行 = 2分以内 | 処理をシンプルに、ループ処理はGAS |
| 複雑な計算 | AppSheetは大量データ処理が苦手 | GASで処理、結果をAppSheetに書き戻し |
| 同時実行競合 | 毎時00分に複数Botが集中 | 実行時刻を数分ずらす |

### 3.4 デバッグ方法

#### Step 1: Audit History確認

1. **Monitor** → **Audit History** を開く
2. **Filter by bot** でトラブルのあるBotを選択
3. 実行ログを確認
   - **Result: Success** → 正常実行
   - **Result: Failed** → エラー原因を確認

#### Step 2: ログの読み方

```
# 成功例
Result: Success
Bot: 申請受付通知
Triggered by: データ変更（新規追加）
Condition: TRUE
Steps executed: 1/1

# 失敗例（Condition = FALSE）
Result: Not triggered
Bot: 承認処理
Triggered by: データ変更（更新）
Condition: FALSE  ← Conditionが満たされなかった
Steps executed: 0/1

# 失敗例（エラー）
Result: Failed
Bot: 承認処理
Error: Timeout after 120 seconds
Steps executed: 1/2（Step 1は成功、Step 2でタイムアウト）
```

#### Step 3: テストデータで検証

1. テスト用のレコードを作成
2. 意図した操作（追加/更新/削除）を実行
3. Audit Historyで結果を確認
4. Conditionが意図通りに評価されているか確認

---

## 4. 実践的なセットアップ手順

### 4.1 申請受付通知Bot（新規申請時にメール送信）

#### Step 1: Bot作成

1. **Automation** → **Bots** → **+ New Bot**
2. **Bot name**: `申請受付通知`
3. **Save**

#### Step 2: Event設定

1. **Event** セクション → **Configure event**
2. 設定項目:
   - **Event Type**: Data change
   - **Table**: `Applications`
   - **Data change type**: `Adds only`
   - **Condition**: （空白）
3. **Save**

#### Step 3: Process設定

1. **Process** セクション → **Configure process**
2. **Add a step** → **Send an email**
3. 設定項目:
   - **To**: `<<[メールアドレス]>>`
   - **Subject**: `【睦備建設】申請を受け付けました`
   - **Body**:
     ```
     <<[氏名]>> 様

     ご申請を受け付けました。

     【申請内容】
     申請ID: <<[申請ID]>>
     申請種別: <<[申請種別]>>
     申請日: <<[申請日]>>

     承認完了後、改めてご連絡いたします。

     ---
     睦備建設 管理部
     ```
4. **Save**

#### Step 4: テスト

1. テスト用の申請を作成
2. Audit Historyで実行ログを確認
3. メールが届いているか確認

---

### 4.2 承認処理Bot（ステータス変更時にデータ更新）

#### Step 1: Bot作成

1. **Automation** → **Bots** → **+ New Bot**
2. **Bot name**: `承認時契約レコード作成`
3. **Save**

#### Step 2: Event設定

1. **Event** セクション → **Configure event**
2. 設定項目:
   - **Event Type**: Data change
   - **Table**: `Applications`
   - **Data change type**: `Updates only`
   - **Condition**:
     ```
     AND(
       [_THISROW_BEFORE].[ステータス] <> "承認済み",
       [ステータス] = "承認済み",
       [申請種別] = "契約"
     )
     ```
3. **Save**

#### Step 3: Process設定（複数ステップ）

**Step 1: Set column values**

1. **Add a step** → **Run a data action** → **Set the values of some columns in this row**
2. 設定項目:
   - **Set these columns**:
     - `承認日`: `TODAY()`
     - `承認者`: `USEREMAIL()`
3. **Save**

**Step 2: Send an email**

1. **Add a step** → **Send an email**
2. 設定項目:
   - **To**: `<<[メールアドレス]>>`
   - **Subject**: `【睦備建設】申請が承認されました`
   - **Body**:
     ```
     <<[氏名]>> 様

     ご申請が承認されました。

     【申請内容】
     申請ID: <<[申請ID]>>
     承認日: <<[承認日]>>

     契約手続きを開始いたします。

     ---
     睦備建設 管理部
     ```
3. **Save**

#### Step 4: テスト

1. テスト用の申請を作成（ステータス: "申請中"）
2. ステータスを "承認済み" に更新
3. Audit Historyで実行ログを確認
4. 承認日・承認者が更新されているか確認
5. メールが届いているか確認

---

## 5. 設定チェックリスト

### 5.1 Event設定チェック

- [ ] **Event Type** が適切か（Data change / Schedule）
- [ ] **Table** が正しいか
- [ ] **Data change type** が適切か（Adds only / Updates only / Deletes only）
- [ ] **Condition** が設定されているか（Updates only の場合は必須）
- [ ] **Condition** に `[_THISROW_BEFORE]` を使っているか（Updates only の場合）
- [ ] **Bypass Security Filters** が適切か（スケジュールBotでは有効化）

### 5.2 Process設定チェック

- [ ] **Step の順序** が正しいか
- [ ] **Send email** の To / Subject / Body が適切か
- [ ] **Set column values** の列名と値が正しいか
- [ ] **列参照** が `<<[列名]>>` 形式で正しいか
- [ ] **処理時間** が2分以内に収まるか

### 5.3 テスト実施チェック

- [ ] テストデータで実行して Audit History を確認したか
- [ ] **Result: Success** になっているか
- [ ] **Condition** が意図通りに評価されているか
- [ ] メール / 通知が届いているか
- [ ] データが正しく更新されているか

### 5.4 本番デプロイ前チェック

- [ ] 有料プランに契約しているか（Email / Schedule 使用時）
- [ ] Bypass Security Filters の設定が適切か
- [ ] エラーハンドリングが実装されているか
- [ ] ログ監視体制が整っているか

---

## 6. 推奨設定パターン

### パターン1: 単純通知（Adds only）

```
用途: 新規申請時のメール通知
Event Type: Data change
Data change type: Adds only
Condition: （空白）
Process: Send an email

メリット: シンプル、無限ループの心配なし
```

### パターン2: ステータス変更検知（Updates only）

```
用途: 承認時の処理
Event Type: Data change
Data change type: Updates only
Condition: AND([_THISROW_BEFORE].[ステータス] <> "承認済み", [ステータス] = "承認済み")
Process: Set column values → Send an email

注意点: Condition必須、無限ループに注意
```

### パターン3: 定期実行（Schedule）

```
用途: 毎日の未処理申請リマインダー
Event Type: Schedule
Schedule: Daily at 09:00
Condition: [ステータス] = "申請中"
Process: Send an email

注意点: Bypass Security Filters 有効化、有料プラン必須
```

---

## 7. 参考資料

### 7.1 公式ドキュメント

- [bot: 基本情報 - AppSheet ヘルプ](https://support.google.com/appsheet/answer/11432969?hl=ja)
- [bot のトラブルシューティング - AppSheet ヘルプ](https://support.google.com/appsheet/answer/11547467?hl=ja)
- [Events: The Essentials - AppSheet Help](https://support.google.com/appsheet/answer/11445188?hl=en)
- [Bots: The Essentials - AppSheet Help](https://support.google.com/appsheet/answer/11432969?hl=en)
- [Troubleshoot bots - AppSheet Help](https://support.google.com/appsheet/answer/11547467?hl=en)
- [更新前後の列の値にアクセスする - AppSheet ヘルプ](https://support.google.com/appsheet/answer/11547057?hl=ja)

### 7.2 実践ガイド

- [AppSheet Automationの使い方（1） 6つのTaskでできること](https://appsheet-apps.jp/appsheet/automation-review-01/)
- [AppSheet Automationの使い方（6）「Send a notification」で通知](https://appsheet-apps.jp/appsheet/automation-review-06/)
- [AppSheet Automation で自動化する方法 | Google Cloud 公式ブログ](https://cloud.google.com/blog/ja/products/no-code-development/automation-bots-with-appsheet-and-no-code?hl=ja)
- [AppSheetでタスク管理アプリ（第7回）アクションで列の値を更新する](https://appsheet-apps.jp/appsheet/mytask-07/)
- [AppSheetステップアップ|複数アクションを順番に自動実行しよう](https://note.com/fuku_create/n/n2021a9cf03bf)

### 7.3 コミュニティ

- [Google Developer forums - AppSheet Q&A](https://discuss.google.dev/c/appsheet/)
- [AppSheet Creator Community](https://community.appsheet.com/)

---

## プロジェクトへの示唆

### mutsubi-dx D-1 PoCへの適用

#### 推奨Bot構成

1. **申請受付通知Bot** (Adds only)
   - 新規申請時にメール送信
   - リスク: 低、実装: 簡単

2. **承認処理Bot** (Updates only)
   - ステータス変更検知
   - Condition必須、無限ループ注意
   - n8n webhookとの連携

3. **定期リマインダーBot** (Schedule)
   - 毎日の未処理申請確認
   - 有料プラン必須、Phase A-2で実装

#### 重要な設計判断

- **Phase A-1**: Adds only のみ実装（リスク最小化）
- **Phase A-2**: Updates only 実装（Condition式の検証必須）
- **n8n連携**: Call a webhook でデータ連携

---

最終更新: 2026-02-16
更新者: mutsubi-researcher
