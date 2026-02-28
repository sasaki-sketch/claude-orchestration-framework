# AppSheet 実装Tips

> 作成日: 2026-02-16

## このファイルの役割

**目的**: AppSheet実装で得た学びを集約し、再利用可能なナレッジを構築

**対象読者**: AppSheet実装を行うエージェント・開発者

**このファイルを参照するタイミング**:
- AppSheet実装作業を開始する前
- 設定方法がわからない時
- よく使う数式を探す時

**関連ファイル**:
- 問題解決時: `/learnings/appsheet-troubleshooting.md`
- Views詳細: `/learnings/appsheet-views-guide.md`
- D-1実装: `/context/d1-poc-implementation-guide.md`

---

## 1. Data設定

### 1.1 列タイプ（Type）設定

| # | Tips | 重要度 |
|---|------|--------|
| D-1 | 列タイプ（Type）はShow_If設定より先に変更する | 高 |
| D-2 | Enum（単一選択）とEnumList（複数選択）を区別する | 高 |
| D-3 | Date型にはMinimum設定がない。Valid Ifで比較式を使う | 高 |
| D-4 | 日付バリデーション: `[_THIS] >= TODAY() + 30` | 中 |
| D-5 | Image folder pathは空欄でOK（自動でフォルダ作成される） | 低 |
| D-6 | _ComputedName列は実際の列名（姓、名）に合わせて修正する | 中 |

### 1.2 SHOW設定

| # | Tips | 重要度 |
|---|------|--------|
| D-7 | SHOW設定は全テーブルで確認する（内部ID、システム列は非表示に） | 高 |
| D-8 | **SHOW?=NoにするとView OptionsのDropdownに列が表示されない** | 高 |
| D-9 | 不要な列（登録日など）は削除してシンプルに保つ | 中 |

### 1.4 Show型Virtual Column（フォームヘッダー・説明文）

**Show型とは**: データ自体を格納せず、フォームやビューのレイアウト・スタイルを制御する特殊なデータ型

| # | Tips | 重要度 |
|---|------|--------|
| D-12 | **Show型のApp formulaは必ず `""` （空文字）にする** | 高 |
| D-13 | 実際の表示内容はContentフィールドに記述する | 高 |
| D-14 | Show型は仮想列（Virtual Column）またはスプレッドシート空白列で実装 | 高 |
| D-15 | デフォルトではForm viewのみ表示。Detail viewで表示するには「Include Show columns in detail views」を有効化 | 中 |

**Show型の6つのCategory（Type Details）**:

| Category | 用途 | 表示方法 |
|----------|------|----------|
| **Page_Header** | ページ区切り（複数ページフォーム作成） | Simple / Page Count / Tabs |
| **Section_Header** | セクション見出し | テキスト表示 |
| **Text** | 説明文・注意事項 | テキスト表示 |
| **URL** | リンク | HYPERLINK関数で埋め込み可 |
| **Image** | 画像 | 公開URL参照（Google Drive不可） |
| **Video** | 動画 | MP4 or YouTube埋め込み |

**設定手順**:
1. `Data` → 対象テーブル → `+ Add virtual column`
2. `Column name`（例: `フォームヘッダー`）
3. `Type` → `Show`
4. `Type Details` → `Category` → `Page_Header` / `Section_Header` / `Text` 等を選択
5. `App formula` → `""` （必ず空文字）
6. `Content` → 表示したいテキスト（改行は直接入力）
7. `Save`

**改行を含むテキストの記述方法**:
- Contentフィールド内で直接改行可能（Enterキーで改行）
- 動的な改行: App formulaで `CONCATENATE()` や `CHAR(10)` を使用（ただしShow型では基本的にContentに静的テキストを記述）

**ベストプラクティス**:
- フォーム先頭の注意事項: `Page_Header` + Contentに「管理組合理事長 殿\n※ 本申請は区分所有者ご本人のみ申請可能です。」
- セクション区切り: `Section_Header` + Content「車両情報」等
- 説明文: `Text` + Content「解約日は申請日の30日後以降を指定してください」
- Format rulesと組み合わせて条件付き表示（CONTEXT関数でForm viewのみ警告表示等）

---

## 7. Show型Virtual Columnの設定（詳細ガイド）

### 7.1 重要ポイント

| # | ポイント | 詳細 |
|---|---------|------|
| S-1 | **App formulaは必ず `""` （空文字）** | Show型ではApp formulaは無視される。空文字設定で「表示制御のみ」を明示 |
| S-2 | **実際の表示内容はContentに記述** | Contentフィールドにテキストを入力（改行は直接入力可能、クォート不要） |
| S-3 | **Categoryの選択が重要** | Page_Header / Section_Header / Text など、用途に応じて適切なCategoryを選択 |

### 7.2 Categoryの種類と使い分け

| Category | 用途 | 設定例 |
|----------|------|--------|
| **Page_Header** | ページ区切り・フォームヘッダー | 「管理組合理事長 殿」など宛先表示 |
| **Section_Header** | セクション見出し | 「車両情報」「口座情報」などの区切り |
| **Text** | 説明文・注意事項 | 「※ 本申請は区分所有者ご本人のみ申請可能です。」 |

### 7.3 正しい設定例

**フォームヘッダーの設定**:

| 項目 | 設定値 |
|------|--------|
| Column name | フォームヘッダー |
| Type | Show |
| Category | Page_Header |
| App formula | "" |
| Content | 管理組合理事長 殿（改行）※ 本申請は区分所有者ご本人のみ申請可能です。 |

**注意**: Contentフィールド内で改行する場合は、直接Enterキーで改行できます。

### 7.4 よくある間違い

| 間違い | 正しい方法 |
|--------|----------|
| App formulaに表示テキストを入れる | App formulaは `""` のみ、Contentに表示テキスト |
| Contentに式を書く（例: `CONCATENATE(...)`） | 静的テキストはContentに直接、動的テキストが必要な場合のみApp formulaを使用 |
| Categoryを設定し忘れる | 必ず用途に応じたCategoryを選択 |

### 7.5 Text Viewとの違い

| 項目 | Show型Virtual Column | Text View |
|------|---------------------|-----------|
| 用途 | フォーム内の説明文・ヘッダー | 単独の説明画面 |
| 設定場所 | Data → Virtual Column | UX → Views → Text |
| Content設定 | Data設定のContent | View OptionsのContent |
| 表示場所 | フォーム内（他の項目と一緒） | 独立した画面 |

**使い分け**:
- **Show型**: フォーム内に注意書きや区切りを表示したい場合
- **Text View**: お問い合わせ先など、独立した情報ページを作成する場合

### 1.3 Ref（参照）設定

| # | Tips | 重要度 |
|---|------|--------|
| D-10 | Valid If式でSELECTを使って参照先をフィルタ可能 | 中 |
| D-11 | Invalid value errorで日本語エラーメッセージを設定 | 中 |

---

## 2. Views設定

### 2.1 System Generated Views vs 新規View

| # | Tips | 重要度 |
|---|------|--------|
| V-1 | **System Generated ViewsにはPosition設定がない。新規View作成が必要** | 高 |
| V-2 | View作成時は「Create a new view」ボタンから新規作成 | 中 |
| V-3 | Display nameは空欄でOK（View nameがそのまま使われる） | 低 |

### 2.2 Position設定

| Position | 表示場所 | 用途 |
|----------|----------|------|
| `first` | プライマリナビゲーションの最初 | アプリ起動時の初期画面 |
| `menu` | メニュー（ハンバーガーメニュー内） | 補助的なView |
| `ref` | 参照時のみ表示 | ドリルダウン、編集画面 |

### 2.5 Dashboard View制限事項

| # | Tips | 重要度 |
|---|------|--------|
| V-7 | **Form型ViewはDashboardに埋め込めない** | 高 |
| V-8 | Dashboardには deck / table / gallery / detail などを埋め込む | 中 |
| V-9 | 新規申請へはメニューナビゲーションでアクセスさせる | 中 |

### 2.3 Row filter（Slice経由）

| # | Tips | 重要度 |
|---|------|--------|
| V-4 | **Row filterはView直接設定ではなく Slice で設定する** | 高 |
| V-5 | Data → Slices でフィルタ条件を定義 | 高 |
| V-6 | Viewの「For this data」でSliceを選択してフィルタを適用 | 中 |

**Slice設定手順**:
1. `Data` → `Slices` → `+ New Slice`
2. Slice Name（例: `申請履歴_自分のみ`）
3. Source Table（例: `Applications`）
4. Row filter condition（例: `[メールアドレス]=USEREMAIL()`）
5. `Save`
6. `UX` → `Views` → 対象View → `For this data` でSliceを選択

### 2.4 View Options設定

| 設定項目 | 説明 | 注意点 |
|----------|------|--------|
| Primary header | カードのメインタイトル | SHOW?=Noの列は表示されない |
| Secondary header | サブタイトル | SHOW?=Noの列は表示されない |
| Summary column | 要約表示 | SHOW?=Noの列は表示されない |

---

## 3. Actions設定

| # | Tips | 重要度 |
|---|------|--------|
| A-1 | **Position = Hide で非表示にする**（推奨方法） | 高 |
| A-2 | Only if this condition is true = FALSE でも非表示にできる | 中 |
| A-3 | **LINKTOFORMは列名・値ペアが必須**: `LINKTOFORM("View名", "列名", 値)` | 高 |
| A-4 | `LINKTOFORM("View名")` だけではエラー | 高 |

### 3.1 Action非表示の設定方法

**推奨**: `Position` → `Hide` を選択

**代替**: `Behavior` → `Only if this condition is true` → `FALSE`

### 3.2 LINKTOFORM構文

```
# 正しい構文（列名・値ペア必須）
LINKTOFORM("新規申請", "区画ID", [区画ID], "施設種別", [施設種別])

# エラーになる構文
LINKTOFORM("新規申請")  ← 列名・値ペアがないのでエラー
```

**効果**: フォームに遷移時、指定した列の値が自動入力（プリフィル）される

---

## 4. Automation（Bot）設定

> **参照**: 詳細は `appsheet-automation-guide.md` を参照

### 4.1 Event設定のベストプラクティス

| # | Tips | 重要度 |
|---|------|--------|
| B-1 | **Event Type**: Adds only（新規追加時のみ）vs Updates only（更新時のみ）vs Deletes only（削除時のみ） | 高 |
| B-2 | **Updates only使用時はConditionで無限ループ防止必須** | 高 |
| B-3 | Condition式で `[_THISROW_BEFORE]` と `[_THISROW_AFTER]` を使って変更前後を比較 | 高 |
| B-4 | データソースに直接変更を加えてもBotは発火しない（AppSheet経由のみ） | 高 |
| B-5 | スケジュール実行時 `USEREMAIL()` は NULL になる | 中 |

### 4.2 Process（Step）設定

| # | Tips | 重要度 |
|---|------|--------|
| B-6 | Send emailのBody/Subject内で `<<[列名]>>` で値を参照 | 中 |
| B-6a | **Send emailのToフィールドは `[列名]` のみ（`<<>>` 不要）** | 高 |
| B-7 | Set column valuesは1ステップで複数列を更新可能 | 中 |
| B-8 | 複数ステップを連鎖させる場合は順序を明確に設計 | 高 |
| B-9 | **タイムアウト**: Add/Update/DeleteとBot実行の合計が2分を超えるとタイムアウト | 高 |

### 4.3 Bypass Security Filters

| # | Tips | 重要度 |
|---|------|--------|
| B-10 | **スケジュールBotで全データにアクセスする場合は有効化** | 高 |
| B-11 | ユーザー操作トリガのBotでは通常は無効（セキュリティフィルタ適用） | 中 |
| B-12 | 有効化するとBotは全データを参照・更新可能になる | 高 |

### 4.4 よくある間違い・トラブルシューティング

| # | 原因 | 対処法 | 重要度 |
|---|------|--------|--------|
| B-13 | Botが発火しない: データソースに直接変更 | AppSheet経由でデータを変更 | 高 |
| B-14 | Botが発火しない: Condition式が常にFALSE | Audit Historyでログを確認、式を修正 | 高 |
| B-15 | Botが発火しない: 有料プラン未契約 | Prototypeでは一部機能が制限される | 高 |
| B-16 | 無限ループ: Updates onlyでCondition未設定 | `[_THISROW_BEFORE].[列] <> [列]` で変更時のみ発火 | 高 |
| B-17 | タイムアウト: 処理時間が2分超過 | ループ処理をGASで実装、Botをシンプルに | 中 |
| B-18 | スケジュール実行失敗: 同時刻に複数Bot | 実行時刻を数分ずらす（毎時00分を避ける） | 中 |

### 4.5 実装パターン

#### パターン1: 申請受付通知Bot（新規申請時にメール送信）

```
Bot Name: 申請受付通知
Event Type: Data change
  Table: Applications
  Data change type: Adds only
  Condition: （空白 = 常に発火）
Process: 申請受付通知プロセス
  Step 1: Send an email
    To: <<[メールアドレス]>>
    Subject: 【睦備建設】申請を受け付けました
    Body: <<[氏名]>> 様\n\nご申請（申請ID: <<[申請ID]>>）を受け付けました。
```

#### パターン2: 承認処理Bot（ステータス変更時にデータ更新）

```
Bot Name: 承認時契約レコード作成
Event Type: Data change
  Table: Applications
  Data change type: Updates only
  Condition: AND([_THISROW_BEFORE].[ステータス]<>"承認済み",[ステータス]="承認済み",[申請種別]="契約")
Process: 契約レコード作成プロセス
  Step 1: Set column values in this row
    Set these columns:
      - 承認日: TODAY()
      - 承認者: USEREMAIL()
  Step 2: Call a webhook (or execute a grouped action)
    URL: （n8n webhook URL）
    Body: {"申請ID": "<<[申請ID]>>", "区画ID": "<<[区画ID]>>"}
```

### 4.6 デバッグ方法

| # | 方法 | 手順 | 重要度 |
|---|------|------|--------|
| B-19 | Audit History確認 | Monitor → Audit History → Botを選択 → Details確認 | 高 |
| B-20 | Result: Success確認 | "Result": "Success" なら実行成功 | 高 |
| B-21 | エラーメッセージ確認 | ログにエラー理由が表示される | 高 |
| B-22 | Condition式のテスト | テストデータで意図通り発火するか確認 | 中 |

### 4.7 Bot Step: 関連テーブル更新（v1.4追加）

| # | Tips | 重要度 |
|---|------|--------|
| B-23 | **関連テーブル更新は「Run a data action」→「Run action on rows」を使用** | 高 |
| B-24 | Referenced Table: 更新対象テーブル、Referenced rows: `LIST([参照列])` | 高 |
| B-25 | Referenced Action: 事前に対象テーブルにActionを作成しておく（Position=Hide） | 高 |
| B-26 | 複数テーブル更新は複数Stepを追加（Step1: 旧区画→空き、Step2: 新区画→契約中） | 中 |

### 4.8 Google Workspace セキュリティ制限（v1.4追加）

| # | Tips | 重要度 |
|---|------|--------|
| B-27 | **AppSheet Core セキュリティ有効時、Webhookは使用不可** | 高 |
| B-28 | 回避策1: 管理コンソール → アプリ → AppSheet → セキュリティ無効化 | 中 |
| B-29 | 回避策2: メール通知で代替（社内メールはセキュリティ制限対象外） | 高 |
| B-30 | 設定場所: 管理コンソール検索バーで「AppSheet」→「AppSheet Core ライセンスのセキュリティ設定」 | 中 |

---

## 5. 一般的な注意点

### 5.1 UI/UXのベストプラクティス

| # | Tips | 重要度 |
|---|------|--------|
| U-1 | プライマリナビには5個以下のViewを配置 | 中 |
| U-2 | Column Nameは短く、Descriptionで詳細説明 | 中 |
| U-3 | モバイルファーストで設計 | 中 |

---

## 6. よく使う数式

### 6.1 フィルタ・参照

```
# 空き区画のみ（施設種別・物件番号でフィルタ）
SELECT(ParkingSpaces[区画ID],AND([契約状況]="空き",[施設種別]=[_THISROW].[施設種別],[物件番号]=[_THISROW].[物件番号]))

# 自分が契約中の区画のみ
SELECT(ParkingSpaces[区画ID],AND([契約状況]="契約中",[契約部屋番号]=[_THISROW].[部屋番号],[物件番号]=[_THISROW].[物件番号]))

# 自分の申請履歴のみ（Slice用）
[メールアドレス]=USEREMAIL()
```

### 6.2 条件分岐

```
# 申請種別による表示切替
OR([申請種別]="契約",[申請種別]="移動")
```

### 6.3 日付バリデーション

```
# 30日以降の日付のみ許可
[_THIS] >= TODAY() + 30
```

### 6.4 Bot条件

```
# ステータスが「承認済み」に変更された時（契約申請の場合）
AND([_THISROW_BEFORE].[ステータス]<>"承認済み",[ステータス]="承認済み",[申請種別]="契約")
```

---

## 8. Ref型とLOOKUP関数（v1.5追加）

### 8.1 Ref型の動作原理

| # | Tips | 重要度 |
|---|------|--------|
| R-1 | **Ref参照は参照先テーブルのKEY値と一致する必要がある** | 高 |
| R-2 | KEYでない列の値がRef列に入っていると`[列名].[参照列]`が動作しない | 高 |
| R-3 | 例: PropertiesのKEYが「物件ID」(P001)の場合、Refに「65」を入れても参照できない | 高 |

### 8.2 LOOKUP関数での代替

| # | Tips | 重要度 |
|---|------|--------|
| R-4 | **KEYでない列を参照するにはLOOKUP関数を使用** | 高 |
| R-5 | 構文: `LOOKUP([検索値], "テーブル名", "検索列", "取得列")` | 高 |
| R-6 | Bot内での使用: `<<LOOKUP([物件番号], "Properties", "物件番号", "物件名")>>` | 高 |

**実例**:
```
# Ref参照が動作しない場合
<<[物件番号].[物件名]>>  ← KEYが「物件ID」の場合動作しない

# LOOKUP関数で代替
<<LOOKUP([物件番号], "Properties", "物件番号", "物件名")>>  ← 正しく動作
```

---

## 9. Slice設計パターン（v1.5追加）

### 9.1 マイページ用 vs 管理者用 Slice

| # | Tips | 重要度 |
|---|------|--------|
| S-4 | **マイページ用SliceはUSEREMAIL()でフィルタ** | 高 |
| S-5 | **管理者用Sliceは全件表示または別条件でフィルタ** | 高 |
| S-6 | 同じテーブルに対して用途別に複数Sliceを作成 | 中 |

**Slice設計例**:

| Slice名 | 用途 | Row filter condition |
|---------|------|---------------------|
| 契約中区画_マイページ | ユーザー自身の契約 | `AND([ステータス]="契約中", [部屋番号]=LOOKUP(...))` |
| 契約中区画_管理者用 | 管理者が全件確認 | `[ステータス]="契約中"` |
| 申請履歴_本人のみ | ユーザー自身の申請 | `[メールアドレス]=USEREMAIL()` |
| 申請履歴_管理者用 | 管理者が全件確認 | `OR([ステータス]="申請中", [ステータス]="承認済み")` |

### 9.2 管理者用ダッシュボード構成

| # | Tips | 重要度 |
|---|------|--------|
| S-7 | **管理者用ホームはDashboard Viewで作成** | 高 |
| S-8 | View entriesで管理者用Sliceを参照するビューを配置 | 高 |
| S-9 | Show_Ifで管理者のみ表示: `IN(USEREMAIL(), {"admin@example.com"})` | 高 |

---

## 10. 初期値とステータス管理（v1.5追加）

### 10.1 ステータス列のInitial value

| # | Tips | 重要度 |
|---|------|--------|
| I-1 | **新規レコードのステータスが空になる問題はInitial valueで解決** | 高 |
| I-2 | 設定: Data → Tables → 列 → Initial value → `"申請中"` | 高 |
| I-3 | Initial valueを設定しないとSliceフィルターで新規レコードが表示されない | 高 |

### 10.2 日付列のInitial value

| # | Tips | 重要度 |
|---|------|--------|
| I-4 | 申請日のデフォルト: `TODAY()` | 中 |
| I-5 | 解約日のデフォルト（30日後）: `TODAY() + 30` または `[申請日] + 30` | 中 |

---

## 11. 入力バリデーション（v1.5追加）

### 11.1 数字のみ入力制限（MATCHES関数なしの場合）

| # | Tips | 重要度 |
|---|------|--------|
| V-10 | **AppSheetにMATCHES関数がない場合、SUBSTITUTE連鎖で対応** | 高 |
| V-11 | 数字を全て除去し、残りが空なら数字のみ | 高 |

**口座番号バリデーション（7-8桁の数字のみ）**:
```
AND(
  LEN([_THIS])>=7,
  LEN([_THIS])<=8,
  LEN(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(
    SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(
      [_THIS],"0",""),"1",""),"2",""),"3",""),"4",""),
      "5",""),"6",""),"7",""),"8",""),"9",""))=0
)
```

---

## 更新履歴

| 日付 | 内容 |
|------|------|
| 2026-02-16 | 初版作成（d1-poc-learnings.md Section 5 を移動・拡張） |
| 2026-02-16 | Views設定の学び V-1〜V-6 を追加 |
| 2026-02-16 | Show型Virtual Column（D-12〜D-15）を追加 |
| 2026-02-16 | Action非表示（Position=Hide）、LINKTOFORM構文、Dashboard制限事項を追加 |
| 2026-02-17 | Ref型とLOOKUP関数（R-1〜R-6）を追加 |
| 2026-02-17 | Slice設計パターン（S-4〜S-9）を追加 |
| 2026-02-17 | 初期値とステータス管理（I-1〜I-5）を追加 |
| 2026-02-17 | 入力バリデーション（V-10〜V-11）を追加 |
