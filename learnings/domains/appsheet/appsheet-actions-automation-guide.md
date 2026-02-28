# AppSheet Actions/Automation 設定ガイド

> 最終更新: 2026年2月
> 本ガイドは AppSheet の Actions と Automation (Bots) の設定に関するベストプラクティスをまとめたものです。

---

## 1. Actions概要

Actions は AppSheet アプリ内で特定の操作を実行するための機能です。ボタンクリック、フォーム保存、行選択など様々なイベントに応じて動作を実行できます。

### 1.1 Action Types一覧

| タイプ | 説明 | 主な用途 |
|--------|------|----------|
| **Navigation** | アプリ内の別のビューへ遷移 | 画面遷移、詳細画面表示 |
| **Data-change** | データの追加・更新・削除 | レコード操作、値の設定 |
| **External** | 外部サービスとの連携 | メール送信、電話発信、URL呼び出し |
| **Grouped** | 複数アクションを順序実行 | 複合処理、ワークフロー |

### 1.2 Actionの使い分け

```
ユーザー操作 → シンプルな処理 → Actions
データ変更  → 自動処理が必要  → Automation (Bots)
定期実行   → スケジュール処理 → Automation (Bots)
```

**Actionsが適している場面:**
- ボタン押下で即座に実行したい処理
- ユーザーが明示的にトリガーする操作
- 画面遷移やナビゲーション

**Automationが適している場面:**
- データ変更を検知して自動実行
- 定期的なレポート送信
- バックグラウンド処理

---

## 2. 主要Action設定手順

### 2.1 Navigation Action（画面遷移）

**用途:** 特定のビューや詳細画面への遷移

**設定手順:**
1. `Behavior` > `Actions` に移動
2. `+ New Action` をクリック
3. 基本設定:
   - **Action name:** わかりやすい名前（例: `Navigate_to_Details`）
   - **For a record of this table:** 対象テーブルを選択
   - **Do this:** `App: go to another view within this app` を選択
4. Target設定:
   - **Target:** 遷移先のビューを選択
   - **LINKTOROWKEY:** 特定行に遷移する場合は行キーを指定

**設定例: 関連レコードの詳細画面へ遷移**
```
Do this: App: go to another view within this app
Target: Order_Detail
LINKTOROWKEY: [OrderID]
```

**注意点:**
- Grouped Action内では、Navigation Actionは最後に1つだけ実行される
- 複数のNavigation Actionを順番に実行することはできない

### 2.2 Data Action（データ操作）

**主要なData Action種類:**

| Action | 説明 |
|--------|------|
| `Data: set the value of some columns` | 特定カラムの値を設定 |
| `Data: add a new row to another table` | 別テーブルに行を追加 |
| `Data: delete this row` | 現在の行を削除 |
| `Data: execute an action on a set of rows` | 複数行に対してアクション実行 |

**設定例: ステータス更新**
```
Action name: Mark_as_Complete
For a record of this table: Tasks
Do this: Data: set the value of some columns in this row
Set these columns:
  - Status: "Complete"
  - CompletedDate: TODAY()
  - CompletedBy: USEREMAIL()
```

**設定例: 別テーブルへの行追加**
```
Action name: Create_Log_Entry
Do this: Data: add a new row to another table using values from this row
Table to add to: ActivityLog
Set these columns:
  - LogDate: NOW()
  - Action: "Updated"
  - RecordID: [ID]
  - UserEmail: USEREMAIL()
```

**動的な値の設定:**
- `[_THISROW]` - 現在の行を参照
- `USEREMAIL()` - 現在のユーザーのメール
- `NOW()` / `TODAY()` - 現在の日時/日付
- `UNIQUEID()` - 一意のID生成

### 2.3 External Action（メール送信等）

**主要なExternal Action:**

| Action | 説明 |
|--------|------|
| `External: start an email` | メール作成画面を開く |
| `External: go to a website` | 外部URLを開く |
| `External: start a phone call` | 電話発信 |
| `External: open a file` | ファイルを開く |

**設定例: メール送信**
```
Action name: Send_Email_to_Customer
Do this: External: start an email
To: [CustomerEmail]
Subject: CONCATENATE("ご注文について - ", [OrderID])
Body: <<
お客様

ご注文番号: [OrderID]
ご注文日: [OrderDate]

ご不明な点がございましたらお問い合わせください。
>>
```

**注意:** External Actionはユーザーのデバイス機能を利用するため、Grouped Action内では最後に1つだけ実行されます。

### 2.4 Grouped Action（複数アクション）

**用途:** 複数の処理を順序立てて実行

**設定手順:**
1. 個別のアクションを先に作成
2. `+ New Action` で新規作成
3. `Do this:` で `Grouped: execute a sequence of actions` を選択
4. `Actions` リストに実行したいアクションを順番に追加

**設定例: 承認処理**
```
Action name: Approve_Request
Actions (in order):
  1. Set_Status_Approved    # ステータスを「承認済」に変更
  2. Set_ApprovedDate       # 承認日を設定
  3. Create_Notification    # 通知レコードを作成
  4. Navigate_to_List       # 一覧画面に戻る（最後に実行）
```

**重要な制約:**
- Navigation/External Actionは1つだけ、かつ最後に実行される
- 各ステップでバリデーションルールが適用される
- データ変更アクションのみ複数実行可能

---

## 3. Automation (Bots)

Bots は AppSheet の自動化機能で、**Event（イベント）** と **Process（プロセス）** の2つの要素で構成されます。

### 3.1 Event設定（トリガー）

**イベントタイプ:**

| タイプ | 説明 | 使用例 |
|--------|------|--------|
| **Data Change** | データの追加・更新・削除 | レコード作成時にメール送信 |
| **Schedule** | 定期的なスケジュール | 日次レポート送信 |
| **External** | 外部からのトリガー | Google Sheetsの変更検知 |

**Data Change Event設定:**
```
Event name: New_Order_Created
Event type: Data Change
Table: Orders
Data change type:
  - Adds only（追加のみ）
  - Updates only（更新のみ）
  - Deletes only（削除のみ）
  - All changes（すべて）
Condition: [Status] = "New"
```

**Before/After値の活用:**

更新イベントでは、変更前後の値にアクセスできます:
- `[_THISROW_BEFORE].[ColumnName]` - 変更前の値
- `[_THISROW_AFTER].[ColumnName]` - 変更後の値

**条件式の例:**
```
# ステータスが「保留」から「承認済」に変わった時のみ発火
AND(
  [_THISROW_BEFORE].[Status] = "保留",
  [_THISROW_AFTER].[Status] = "承認済"
)
```

### 3.2 Process設定（処理）

**Processの構成要素:**
- **Step（ステップ）:** 処理の単位
- **Task（タスク）:** 具体的な処理内容
- **Branch（分岐）:** 条件による処理分岐

**主要なTask種類:**

| Task | 説明 |
|------|------|
| Send an email | メール送信 |
| Send a notification | Push通知送信 |
| Send an SMS | SMS送信 |
| Create a new file | ファイル生成（PDF等） |
| Call a webhook | Webhook呼び出し |
| Run a data action | データアクション実行 |

**Process設定例: 承認依頼メール送信**
```
Process name: Send_Approval_Request

Step 1: Send_Email
  Task type: Send an email
  To: [ManagerEmail]
  Subject: CONCATENATE("承認依頼: ", [RequestTitle])
  Body: <<
    以下の申請が提出されました。

    申請者: [RequesterName]
    申請日: [RequestDate]
    内容: [Description]

    AppSheetで確認してください。
  >>

Step 2: Update_Status
  Task type: Run a data action
  Action: Set_Status_Pending
```

### 3.3 Schedule設定（定期実行）

**スケジュールイベント設定:**
```
Event type: Schedule
Schedule type:
  - Every day
  - Every week
  - Every month
  - Custom
Time: 09:00 (UTC)
Timezone: Asia/Tokyo
```

**重要な注意点:**

1. **実行タイムアウト:** 5分以内に完了する必要がある
2. **正時を避ける:** 多くのBotが正時に実行されるため、数分ずらすことを推奨
3. **有料プランが必要:** スケジュール実行は有料プランでのみ動作
4. **デプロイ必須:** Botはアプリをデプロイしないと実行されない

**推奨設定:**
```
# 正時ではなく、数分ずらす
Bad:  09:00
Good: 09:03 または 08:57
```

---

## 4. 通知設定

### 4.1 メール通知

**Automation経由のメール送信:**

```
Task type: Send an email
Email type: Custom template

To: [CustomerEmail]
CC: [ManagerEmail]  # オプション
Subject: CONCATENATE("【重要】", [Title])
Body: <<
  <h2>お知らせ</h2>
  <p>[Message]</p>
  <p>詳細は以下のリンクからご確認ください。</p>
>>
Attachment: [AttachmentFile]  # オプション
```

**埋め込みアプリビューの送信:**
```
Task type: Send embedded app view email
View: Order_Detail_View
For row: [_THISROW]
```
→ アプリのビューをメール内に埋め込んで送信可能

**メール送信のベストプラクティス:**
- 送信先はテーブルのカラム参照を使用（例: `[CustomerEmail]`）
- 動的な宛先には式を使用（例: `IFS([Type]="A", [EmailA], [Type]="B", [EmailB])`）
- HTMLフォーマットでリッチなメールを作成可能

### 4.2 Push通知

**設定手順:**
1. `Automation` > `Bots` で新規Bot作成
2. Task typeで `Send a notification` を選択
3. 通知設定:
   - **To:** 通知先ユーザー（メールアドレス）
   - **Title:** 通知タイトル
   - **Body:** 通知本文

**設定例:**
```
Task type: Send a notification
To: [AssigneeEmail]
Title: "新しいタスクが割り当てられました"
Body: CONCATENATE("タスク: ", [TaskName], " 期限: ", [DueDate])
Deep link: LINKTOROW([ID], "Task_Detail")
```

**Push通知の要件:**
- ユーザーがアプリをインストールしている必要がある
- ブランドアプリの場合は追加設定が必要
- 通知許可がデバイスで有効になっている必要がある

### 4.3 SMS通知

**設定:**
```
Task type: Send an SMS
To: [PhoneNumber]
Body: CONCATENATE("確認コード: ", [VerificationCode])
```

**SMS送信の制限:**
- Twilio連携が必要
- 文字数制限あり（160文字推奨）
- 有料プラン必須
- 国際SMS送信は追加設定が必要

---

## 5. ベストプラクティス

### Do's（推奨事項）

**設計・命名**
- Action/Bot名は目的が明確にわかる命名にする
  - Good: `Approve_Purchase_Request`, `Send_Daily_Report`
  - Bad: `Action1`, `NewBot`
- 関連するActionsはプレフィックスで整理する
  - 例: `Order_Create`, `Order_Update`, `Order_Delete`

**パフォーマンス**
- Grouped Actionは必要最小限のアクションで構成する
- 条件式（Only if this condition is true）で不要な実行を防ぐ
- スケジュールBotは正時を避けて設定する（例: 09:03）
- 大量データ処理は複数のBotに分割する

**セキュリティ**
- メール送信先は必ずバリデーションする
- 機密データを含むActionには適切な条件を設定
- ユーザー権限に応じてActionの表示/非表示を制御

**テスト・デバッグ**
- 本番デプロイ前に十分にテストする
- Bot実行ログを定期的に確認する
- エラー時の通知設定を行う

### Don'ts（避けるべきこと）

**設計**
- Grouped Action内に複数のNavigation/External Actionを入れない
  - → 最後の1つしか実行されない
- 無限ループを作らない
  - 例: Bot AがBot Bをトリガーし、Bot BがBot Aをトリガー
- 1つのBotに過度な処理を詰め込まない

**パフォーマンス**
- スケジュールBotを正時（XX:00）に設定しない
  - → タイムアウトの原因になる
- 5分以上かかる処理をBotに設定しない
  - → 自動的にタイムアウトする
- 不要なBotを有効のまま放置しない

**データ整合性**
- バリデーションなしでデータ変更アクションを作成しない
- Before/After値を考慮せずに更新トリガーを設定しない
  - → 意図しない発火の原因

**よくあるエラーと対処:**

| エラー | 原因 | 対処 |
|--------|------|------|
| Bot実行されない | デプロイされていない | アプリをデプロイする |
| スケジュールBot動作しない | 無料プラン | 有料プランにアップグレード |
| タイムアウト | 処理が5分超過 | 処理を分割する |
| 複数Navigation無視 | Grouped Action制約 | Navigation は最後に1つだけ |

---

## 7. mutsubi-dx固有の実装例

本セクションはmutsubi-dx（睦備建設・駐車場管理PoC）プロジェクトの実装例を示します。PRD v1.3のセクション10.5で定義された承認時の自動処理を実装します。

### 7.1 契約承認時の自動処理（PRD 10.5.1）

**概要:** 駐車場契約申請が承認された際、以下の処理を自動実行します:
1. Contractsテーブルに契約レコードを追加
2. ParkingSpacesの契約状況を「空き」から「契約中」に更新
3. 申請者へ承認通知メールを送信

#### Event設定

```
Event name: Contract_Application_Approved
Event type: Data Change
Table: Applications
Data change type: Updates only

Condition:
AND(
  [_THISROW_BEFORE].[ステータス] = "申請中",
  [_THISROW_AFTER].[ステータス] = "承認済み",
  [_THISROW_AFTER].[申請種別] = "契約"
)
```

**ポイント:**
- `_THISROW_BEFORE`と`_THISROW_AFTER`を使用して、ステータス変更を検知
- 申請種別が「契約」の場合のみ実行

#### Process Step 1: Contractsテーブルへの行追加

```
Step name: Create_Contract_Record
Task type: Run a data action
Action: Add_Contract

Data Action定義:
  Action name: Add_Contract
  Do this: Data: add a new row to another table using values from this row
  Table to add to: Contracts

  Set these columns:
    契約ID: UNIQUEID()
    物件番号: [_THISROW].[物件番号]
    部屋番号: [_THISROW].[部屋番号]
    区画ID: [_THISROW].[区画ID]
    施設種別: [_THISROW].[施設種別]
    契約者姓: [_THISROW].[姓]
    契約者名: [_THISROW].[名]
    契約開始日: [_THISROW].[契約開始日]
    契約ステータス: IF([_THISROW].[契約開始日] > TODAY(), "予約中", "契約中")
    申請ID: [_THISROW].[申請ID]
    月額利用料: LOOKUP([_THISROW].[区画ID], "ParkingSpaces", "区画ID", "月額利用料")
    保証金: LOOKUP([_THISROW].[区画ID], "ParkingSpaces", "区画ID", "保証金")
    作成日時: NOW()
```

**ポイント:**
- `LOOKUP()`で区画マスタから料金情報を取得
- 契約開始日が未来の場合は「予約中」、当日以降は「契約中」に設定

#### Process Step 2: ParkingSpacesの契約状況更新

```
Step name: Update_ParkingSpace_Status
Task type: Run a data action
Action: Set_Space_Occupied

Data Action定義:
  Action name: Set_Space_Occupied
  Do this: Data: execute an action on a set of rows
  Referenced table: ParkingSpaces
  Referenced rows: SELECT(ParkingSpaces[区画ID], [区画ID] = [_THISROW].[区画ID])
  Action to execute: Set_Status_Contracted

  Set_Status_Contracted定義:
    Do this: Data: set the value of some columns in this row
    Set these columns:
      契約状況: "契約中"
      契約部屋番号: [_THISROW].[部屋番号]
```

**ポイント:**
- `execute an action on a set of rows`で関連する区画レコードを更新
- 区画IDで対象レコードを特定

#### Process Step 3: 承認通知メール送信

```
Step name: Send_Approval_Email
Task type: Send an email

To: [_THISROW].[メールアドレス]
Subject: "【駐車場契約】承認のお知らせ"

Body: <<
[_THISROW].[姓] [_THISROW].[名] 様

駐車場契約申請が承認されました。

■ 契約内容
- 施設種別: [_THISROW].[施設種別]
- 区画番号: [_THISROW].[区画番号]
- 契約開始日: [_THISROW].[契約開始日]

契約書類は別途郵送いたします。
ご不明な点がございましたら、管理事務所までお問い合わせください。

睦備建設 管理部
>>
```

**実装時の注意:**
- Grouped Actionで3つのステップを順序実行
- エラー時は管理者に通知するエラーハンドリング追加を推奨
- テスト時はメール送信先を管理者アドレスに変更して検証

---

### 7.2 解約承認時の自動処理（PRD 10.5.2）

**概要:** 駐車場解約申請が承認された際の処理:
1. Contractsテーブルの契約終了日・ステータス更新
2. ParkingSpacesの契約状況を「契約中」から「空き」に更新
3. 保証金返金フラグ設定（総務課へのアラート）

#### Event設定

```
Event name: Cancellation_Application_Approved
Event type: Data Change
Table: Applications
Data change type: Updates only

Condition:
AND(
  [_THISROW_BEFORE].[ステータス] = "申請中",
  [_THISROW_AFTER].[ステータス] = "承認済み",
  [_THISROW_AFTER].[申請種別] = "解約"
)
```

#### Process Step 1: Contractsテーブルの契約終了

```
Step name: End_Contract
Task type: Run a data action
Action: Update_Contract_End

Data Action定義:
  Action name: Update_Contract_End
  Do this: Data: execute an action on a set of rows
  Referenced table: Contracts
  Referenced rows: SELECT(
    Contracts[契約ID],
    AND(
      [部屋番号] = [_THISROW].[部屋番号],
      [区画ID] = [_THISROW].[解約区画ID],
      [契約ステータス] = "契約中"
    )
  )
  Action to execute: Set_Contract_Ended

  Set_Contract_Ended定義:
    Do this: Data: set the value of some columns in this row
    Set these columns:
      契約終了日: [_THISROW].[解約日]
      契約ステータス: "解約済み"
      更新日時: NOW()
```

**ポイント:**
- 現在「契約中」のレコードを検索して更新
- 複数契約がある場合でも、部屋番号と区画IDで特定

#### Process Step 2: ParkingSpacesの空き状態への更新

```
Step name: Update_ParkingSpace_Available
Task type: Run a data action
Action: Set_Space_Available

Data Action定義:
  Action name: Set_Space_Available
  Do this: Data: execute an action on a set of rows
  Referenced table: ParkingSpaces
  Referenced rows: SELECT(ParkingSpaces[区画ID], [区画ID] = [_THISROW].[解約区画ID])
  Action to execute: Set_Status_Available

  Set_Status_Available定義:
    Do this: Data: set the value of some columns in this row
    Set these columns:
      契約状況: "空き"
      契約部屋番号: ""
```

#### Process Step 3: 保証金返金アラート（総務課向け）

```
Step name: Send_Refund_Alert
Task type: Send an email

To: "accounting@molvia.co.jp"  # 総務課メールアドレス
Subject: CONCATENATE("【保証金返金】", [_THISROW].[部屋番号], " ", [_THISROW].[姓], [_THISROW].[名])

Body: <<
総務課 御中

駐車場解約が承認されました。保証金の返金処理をお願いいたします。

■ 解約情報
- 申請ID: [_THISROW].[申請ID]
- 部屋番号: [_THISROW].[部屋番号]
- 契約者名: [_THISROW].[姓] [_THISROW].[名]
- 解約日: [_THISROW].[解約日]
- 区画番号: [_THISROW].[解約区画番号]

■ 返金口座情報
- 金融機関名: [_THISROW].[口座_金融機関名]
- 支店名: [_THISROW].[口座_支店名]
- 口座種別: [_THISROW].[口座_種別]
- 口座番号: [_THISROW].[口座_番号]
- 名義人: [_THISROW].[口座_名義人]（フリガナ: [_THISROW].[口座_名義人フリガナ]）

※ 口座情報が未入力の場合は契約者へ別途確認してください。

AppSheetで申請詳細を確認: [アプリURL]

管理部
>>
```

**条件分岐の追加（オプション）:**
口座情報が登録されていない場合とされている場合で通知内容を変える:

```
Step name: Branch_Refund_Alert
Branch type: IF

Condition: ISBLANK([_THISROW].[口座_金融機関名])

True branch: Send_Refund_Alert_No_Account
  Body: "口座情報が未登録のため、契約者へ確認してください。"

False branch: Send_Refund_Alert_With_Account
  Body: "上記口座情報で返金処理を実施してください。"
```

---

### 7.3 移動承認時の自動処理（PRD 10.5.3）

**概要:** 駐車場移動申請（区画変更）の承認時処理:
1. 旧契約を終了（Contracts更新）
2. 新契約を作成（Contracts追加）
3. ParkingSpacesを2件更新（旧区画「空き」、新区画「契約中」）

#### Event設定

```
Event name: Move_Application_Approved
Event type: Data Change
Table: Applications
Data change type: Updates only

Condition:
AND(
  [_THISROW_BEFORE].[ステータス] = "申請中",
  [_THISROW_AFTER].[ステータス] = "承認済み",
  [_THISROW_AFTER].[申請種別] = "移動"
)
```

#### Process Step 1: 旧契約終了

```
Step name: End_Old_Contract
Task type: Run a data action
Action: Update_Contract_End_Move

Data Action定義:
  Action name: Update_Contract_End_Move
  Do this: Data: execute an action on a set of rows
  Referenced table: Contracts
  Referenced rows: SELECT(
    Contracts[契約ID],
    AND(
      [部屋番号] = [_THISROW].[部屋番号],
      [区画ID] = [_THISROW].[解約区画ID],
      [契約ステータス] = "契約中"
    )
  )
  Action to execute: Set_Contract_Ended_Move

  Set_Contract_Ended_Move定義:
    Do this: Data: set the value of some columns in this row
    Set these columns:
      契約終了日: [_THISROW].[解約日（移動）]
      契約ステータス: "解約済み"
      備考: CONCATENATE("移動により解約（新区画: ", [_THISROW].[区画番号], "）")
      更新日時: NOW()
```

#### Process Step 2: 新契約作成

```
Step name: Create_New_Contract
Task type: Run a data action
Action: Add_New_Contract_Move

Data Action定義:
  Action name: Add_New_Contract_Move
  Do this: Data: add a new row to another table using values from this row
  Table to add to: Contracts

  Set these columns:
    契約ID: UNIQUEID()
    物件番号: [_THISROW].[物件番号]
    部屋番号: [_THISROW].[部屋番号]
    区画ID: [_THISROW].[区画ID]
    施設種別: [_THISROW].[施設種別]
    契約者姓: [_THISROW].[姓]
    契約者名: [_THISROW].[名]
    契約開始日: [_THISROW].[契約日（移動）]
    契約ステータス: IF([_THISROW].[契約日（移動）] > TODAY(), "予約中", "契約中")
    申請ID: [_THISROW].[申請ID]
    月額利用料: LOOKUP([_THISROW].[区画ID], "ParkingSpaces", "区画ID", "月額利用料")
    保証金: LOOKUP([_THISROW].[区画ID], "ParkingSpaces", "区画ID", "保証金")
    備考: CONCATENATE("移動元: ", [_THISROW].[解約区画番号])
    作成日時: NOW()
```

#### Process Step 3: ParkingSpaces更新（旧区画を空きに）

```
Step name: Update_Old_Space_Available
Task type: Run a data action
Action: Set_Old_Space_Available

Data Action定義:
  Action name: Set_Old_Space_Available
  Do this: Data: execute an action on a set of rows
  Referenced table: ParkingSpaces
  Referenced rows: SELECT(ParkingSpaces[区画ID], [区画ID] = [_THISROW].[解約区画ID])
  Action to execute: Set_Status_Available_Move

  Set_Status_Available_Move定義:
    Do this: Data: set the value of some columns in this row
    Set these columns:
      契約状況: "空き"
      契約部屋番号: ""
```

#### Process Step 4: ParkingSpaces更新（新区画を契約中に）

```
Step name: Update_New_Space_Occupied
Task type: Run a data action
Action: Set_New_Space_Occupied

Data Action定義:
  Action name: Set_New_Space_Occupied
  Do this: Data: execute an action on a set of rows
  Referenced table: ParkingSpaces
  Referenced rows: SELECT(ParkingSpaces[区画ID], [区画ID] = [_THISROW].[区画ID])
  Action to execute: Set_Status_Contracted_Move

  Set_Status_Contracted_Move定義:
    Do this: Data: set the value of some columns in this row
    Set these columns:
      契約状況: "契約中"
      契約部屋番号: [_THISROW].[部屋番号]
```

#### Process Step 5: 通知メール送信

```
Step name: Send_Move_Approval_Email
Task type: Send an email

To: [_THISROW].[メールアドレス]
Subject: "【駐車場移動】承認のお知らせ"

Body: <<
[_THISROW].[姓] [_THISROW].[名] 様

駐車場移動申請が承認されました。

■ 移動内容
- 旧区画: [_THISROW].[解約区画番号]（解約日: [_THISROW].[解約日（移動）]）
- 新区画: [_THISROW].[区画番号]（契約開始日: [_THISROW].[契約日（移動）]）
- 施設種別: [_THISROW].[施設種別]

リモコン・鍵の交換手続きについては、管理事務所よりご連絡いたします。

睦備建設 管理部
>>
```

**Grouped Actionでの構成:**
```
Action name: Process_Move_Approval
Actions (in order):
  1. End_Old_Contract
  2. Create_New_Contract
  3. Update_Old_Space_Available
  4. Update_New_Space_Occupied
  5. Send_Move_Approval_Email
```

**重要な注意:**
- 移動処理は複数テーブルへの更新が含まれるため、トランザクション整合性に注意
- 各ステップのエラーハンドリングを実装
- ロールバック処理は手動対応が必要（AppSheetには自動ロールバックなし）

---

### 7.4 管理人向け通知Bot（承認不要）

**概要:** 新規申請が登録された際、管理人へリアルタイム通知を送信する。承認処理は不要で、情報共有のみを目的とする。

#### Event設定

```
Event name: New_Application_Submitted
Event type: Data Change
Table: Applications
Data change type: Adds only

Condition:
# すべての新規申請が対象（条件なし）
TRUE
```

#### Process Step: 管理人へのメール通知

```
Step name: Notify_Manager
Task type: Send an email

To: "manager@molvia.co.jp"  # 管理人メールアドレス
CC: "admin@molvia.co.jp"    # 管理部もCC
Subject: CONCATENATE("【新規申請】", [_THISROW].[申請種別], " - ", [_THISROW].[部屋番号])

Body: <<
管理人 様

駐車場の新規申請が登録されました。

■ 申請情報
- 申請ID: [_THISROW].[申請ID]
- 申請日: [_THISROW].[申請日]
- 申請種別: [_THISROW].[申請種別]
- 申請者: [_THISROW].[部屋番号] [_THISROW].[姓] [_THISROW].[名]
- 電話番号: [_THISROW].[電話番号]

IF([_THISROW].[申請種別] = "契約",
  CONCATENATE(
    "■ 契約内容", CHAR(10),
    "- 施設種別: ", [_THISROW].[施設種別], CHAR(10),
    "- 希望区画: ", [_THISROW].[区画番号], CHAR(10),
    "- 契約開始日: ", [_THISROW].[契約開始日]
  ),
  IF([_THISROW].[申請種別] = "解約",
    CONCATENATE(
      "■ 解約内容", CHAR(10),
      "- 解約区画: ", [_THISROW].[解約区画番号], CHAR(10),
      "- 解約日: ", [_THISROW].[解約日], CHAR(10),
      "- 解約理由: ", [_THISROW].[解約理由]
    ),
    IF([_THISROW].[申請種別] = "移動",
      CONCATENATE(
        "■ 移動内容", CHAR(10),
        "- 旧区画: ", [_THISROW].[解約区画番号], CHAR(10),
        "- 新区画: ", [_THISROW].[区画番号], CHAR(10),
        "- 移動日: ", [_THISROW].[契約日（移動）]
      ),
      CONCATENATE(
        "■ 車庫証明内容", CHAR(10),
        "- 対象区画: ", [_THISROW].[車庫証明_区画ID], CHAR(10),
        "- 所有者名: ", [_THISROW].[車庫証明_自動車所有者名]
      )
    )
  )
)

AppSheetで詳細を確認: [アプリURL]

管理部
>>
```

**ポイント:**
- `Adds only`で新規追加のみトリガー
- 申請種別に応じて通知内容を動的に変更（IF文のネスト）
- 承認処理は別途管理者が手動で実施

**簡易版（固定文言）:**
複雑なIF文を避ける場合:

```
Body: <<
管理人 様

駐車場の新規申請が登録されました。

- 申請ID: [_THISROW].[申請ID]
- 申請種別: [_THISROW].[申請種別]
- 申請者: [_THISROW].[部屋番号] [_THISROW].[姓] [_THISROW].[名]

詳細はAppSheetでご確認ください。

管理部
>>
```

---

### 7.5 実装時の共通注意事項

#### データ整合性の確保

1. **Before/After値の活用:**
   - ステータス変更検知には必ず`_THISROW_BEFORE`と`_THISROW_AFTER`を使用
   - 複数回の更新でBotが重複発火しないよう条件を厳密に設定

2. **SELECT式の精度:**
   - 複数レコードが返る可能性がある場合は条件を追加
   - 例: 部屋番号と区画IDと契約ステータスの3条件でContractsを特定

3. **エラーハンドリング:**
   - 各ProcessにエラーNotificationを追加
   - 管理者へのアラート設定

#### パフォーマンス最適化

1. **Grouped Actionの分割:**
   - 1つのBotで5ステップ以上の処理は避ける
   - 関連性の低い処理は別Botに分割

2. **参照データの最小化:**
   - LOOKUP()の多用を避ける
   - Virtual Columnで事前計算

#### テストプロセス

1. **ステージング環境でのテスト:**
   - 本番データのコピーでテスト実施
   - メール送信先を管理者アドレスに変更

2. **テストケース:**
   - 正常系: ステータス「申請中」→「承認済み」
   - 異常系: すでに「承認済み」のレコードを再更新
   - 境界値: 契約開始日が過去/未来の場合

3. **ロールバック手順の準備:**
   - Botエラー時の手動復旧手順を文書化
   - データベーススナップショットのバックアップ

#### デプロイチェックリスト

- [ ] すべてのAction定義を作成
- [ ] Event条件式を検証
- [ ] Process Stepを正しい順序で構成
- [ ] メール送信先を本番アドレスに変更
- [ ] エラー通知先を設定
- [ ] ステージング環境で全ケーステスト完了
- [ ] データバックアップ取得
- [ ] ロールバック手順を文書化
- [ ] アプリをデプロイ

---

## 6. 参照リンク

### 公式ドキュメント
- [Actions: The Essentials](https://support.google.com/appsheet/answer/10107706?hl=en) - Actions の基本
- [Bots: The Essentials](https://support.google.com/appsheet/answer/11432969?hl=en) - Bots の基本
- [Events: The Essentials](https://support.google.com/appsheet/answer/11445188?hl=en) - イベント設定
- [Tasks: The Essentials](https://support.google.com/appsheet/answer/11445301?hl=en) - タスク設定

### 通知関連
- [Send an email from an automation](https://support.google.com/appsheet/answer/11447614?hl=en) - メール送信
- [Send a notification from an automation](https://support.google.com/appsheet/answer/11513089?hl=en) - Push通知
- [Send embedded app view email](https://support.google.com/appsheet/answer/11511240?hl=en) - 埋め込みビューメール

### 高度な設定
- [Run actions based on view events](https://support.google.com/appsheet/answer/10108214?hl=en) - ビューイベント
- [Access column values before and after an update](https://support.google.com/appsheet/answer/11547057?hl=en) - Before/After値
- [Understand bot scheduling and retry](https://support.google.com/appsheet/answer/11547468?hl=en) - スケジュールとリトライ
- [Set input values dynamically in data-change actions](https://support.google.com/appsheet/answer/13779963?hl=en) - 動的値設定

### トラブルシューティング
- [Troubleshoot bots](https://support.google.com/appsheet/answer/11547467?hl=en) - Botのトラブルシューティング
- [Example automations](https://support.google.com/appsheet/answer/11917747?hl=en) - 自動化の例

### 学習リソース
- [Google Cloud - AppSheet Automation](https://cloud.google.com/appsheet/automation) - 概要
- [Automation bots with AppSheet and no-code](https://workspace.google.com/blog/developers-practitioners/automation-bots-with-appsheet-and-no-code) - Google Workspace Blog
- [Quick start: Build your first automation](https://support.google.com/appsheet/answer/14871588?hl=en) - クイックスタート

---

## 付録: クイックリファレンス

### よく使う式

```
# 現在のユーザー
USEREMAIL()
USERNAME()

# 日時
NOW()
TODAY()
TIMENOW()

# 一意ID
UNIQUEID()

# 条件分岐
IF([Condition], TrueValue, FalseValue)
IFS([Cond1], Val1, [Cond2], Val2, TRUE, Default)

# 文字列結合
CONCATENATE("Text", [Column], "More text")

# 行参照
[_THISROW]
[_THISROW_BEFORE]  # Automation内で更新前の値
[_THISROW_AFTER]   # Automation内で更新後の値

# リンク生成
LINKTOROW([KeyColumn], "ViewName")
LINKTOAPP()
```

### Action設定チェックリスト

- [ ] 適切なAction名を設定した
- [ ] 対象テーブルを正しく選択した
- [ ] Action typeを適切に選択した
- [ ] 条件（Only if this condition is true）を設定した
- [ ] 必要に応じて確認ダイアログを設定した
- [ ] テスト環境で動作確認した

### Bot設定チェックリスト

- [ ] 適切なBot名を設定した
- [ ] Event typeを正しく選択した（Data Change / Schedule）
- [ ] Condition式を適切に設定した
- [ ] Process/Taskを正しく構成した
- [ ] エラーハンドリングを考慮した
- [ ] スケジュールBotの場合、正時を避けた
- [ ] テスト環境で動作確認した
- [ ] 本番デプロイした
