# AppSheet Views 設定ガイド

> 作成日: 2026-02-16

## このファイルの役割

**目的**: AppSheetのViews設定に特化した詳細ガイド

**対象読者**: Views設計・実装を行うエージェント・開発者

**このファイルを参照するタイミング**:
- View Typeの選択に迷う時
- Dashboard/Form/Deck等の設定手順を確認する時
- ベストプラクティスを確認する時

**関連ファイル**:
- 実装Tips（Position/Slice詳細）: `/learnings/appsheet-tips.md`
- 問題解決時: `/learnings/appsheet-troubleshooting.md`

---

## 1. View Types 一覧

AppSheet には複数の View タイプがあり、データの表示方法とユーザーインタラクションを制御できる。

### 1.1 主要 View タイプ一覧

| View タイプ | 用途 | 特徴 |
|------------|------|------|
| **Table** | データ一覧（コンパクト表示） | 大量データの高速スクロール、検索向き |
| **Deck** | カード形式の一覧表示 | 画像 + テキスト2列、視覚的識別に最適 |
| **Detail** | 個別レコードの詳細表示 | 1件のデータを詳細に表示 |
| **Form** | データ入力・編集 | 自動生成、カスタマイズ可能 |
| **Dashboard** | 複数 View の統合表示 | Calendar, Map, Chart, Deck, Table 等を配置可能 |
| **Gallery** | 画像中心の表示 | 写真・メディア管理向き |
| **Chart** | グラフ・チャート | 数値データの可視化（棒、折れ線、ヒストグラム） |
| **Map** | 地図表示 | 住所データを自動認識して表示 |
| **Calendar** | カレンダー表示 | 日付データの時系列表示 |

### 1.2 View タイプ選択の判断基準

```
データ一覧を表示したい
├─ 画像で視覚的に識別したい → Deck View
├─ 大量データを素早く検索したい → Table View
├─ 写真がメインコンテンツ → Gallery View
└─ 地図上に表示したい → Map View

データを入力・編集したい → Form View

個別データの詳細を見たい → Detail View

複数の情報を一画面で見たい → Dashboard View

数値データを分析したい → Chart View

日程・スケジュール管理したい → Calendar View
```

---

## 2. 主要 View 設定手順

### 2.1 Dashboard View

**概要**
複数の View を1つの画面に統合表示する。ホーム画面やサマリー画面に最適。

**設定手順**

1. **UX > Views** で新規 View 作成
2. **View type** で `Dashboard` を選択
3. **View Entries** で表示する View を追加
   - Calendar, Map, Chart, Gallery, Deck, Table, Detail が追加可能
4. **Interactive Mode** を有効化（推奨）
   - 1つの View で選択した内容が他の View に反映される
5. Grid レイアウトを調整

**設定項目**

| 項目 | 説明 | 推奨設定 |
|------|------|----------|
| View entries | 含める View のリスト | 3-5個程度が視認性良好 |
| Interactive mode | 連動表示機能 | ON（関連データがある場合） |
| Starting view | 初期表示時のフォーカス | メインコンテンツの View |

**コード例（UX > Options で設定）**
```
Dashboard に含める View:
- 売上サマリー（Chart）
- 最近の取引（Deck）
- 今月のカレンダー（Calendar）
```

### 2.2 Form View

**概要**
データ入力・編集用の View。テーブルに Add/Update 権限があると自動生成される。

**設定手順**

1. **Data > Columns** で各列の設定を確認
2. **Description** に入力ガイドを記載（フォームの質問文として表示）
3. **Show?** で表示/非表示を制御
4. **UX > Views > Form View** で詳細設定

**Show Type（表示形式）の活用**

| Show Type | 用途 |
|-----------|------|
| `Show` | 通常表示（デフォルト） |
| `Show_if` | 条件付き表示 |
| `Page_Header` | ページヘッダーとして表示 |
| `Section_Header` | セクションの見出し |

**ベストプラクティス**

```
Column Name: CustomerID（短い名前）
Description: お客様の ID を入力してください（長い説明文）
```

- Column Name は短く、Description で詳細説明
- 入力タイプを適切に設定（Dropdown, Date, Number など）
- 必須項目には `Required?` を設定
- `Valid_If` で入力値検証

### 2.3 Deck View

**概要**
カード形式でデータを表示。画像と2つのテキスト列を表示可能。

**設定手順**

1. **UX > Views** で新規 View 作成または既存を編集
2. **View type** で `Deck` を選択
3. **View Options** で以下を設定:
   - Primary header: メインタイトル列
   - Secondary header: サブタイトル列
   - Image: 表示する画像列
   - Action buttons: 電話、メール等のアクション

**Deck View の追加機能**

- **Nested View**: 関連データをレイヤーとして表示可能
- **Grouping**: データをグループ化して表示

**使用シーン**
- 商品カタログ
- 顧客一覧
- 物件情報
- 書籍・メディア管理

### 2.4 Table View

**概要**
スプレッドシートライクなコンパクト表示。大量データの高速閲覧に最適。

**設定手順**

1. **UX > Views** で新規 View 作成または既存を編集
2. **View type** で `Table` を選択
3. **Column order** で表示列を設定
4. **Sort by** でソート順を指定

**設定項目**

| 項目 | 説明 |
|------|------|
| Column order | 表示する列とその順序 |
| Quick edit columns | インライン編集可能な列 |
| Row height | 行の高さ |
| Group by | グループ化する列 |

**Table vs Deck 使い分け**

| 観点 | Table | Deck |
|------|-------|------|
| データ量 | 大量データ向き | 少〜中量データ向き |
| 視認性 | 情報量重視 | 視覚的インパクト |
| 画像 | 不要な場合 | 画像で識別したい場合 |
| 操作 | 素早いスクロール | じっくり閲覧 |

### 2.5 Detail View

**概要**
1件のレコードの詳細情報を表示。

**設定手順**

1. **UX > Views** で Detail View を選択
2. **Column order** で表示順を設定
3. **Image/Row Actions** を設定
4. **Related tables** で関連データの表示設定

**Show Type の活用**

Detail View と Form View では Show Type を使って表示を改善できる:

- `Page_Header`: ページ上部に大きく表示
- `Section_Header`: セクション区切りとして表示
- `Url`: リンクとして表示
- `Text`: 長文テキストとして表示

---

## 3. Position 設定

> Position設定の詳細は `/learnings/appsheet-tips.md` Section 2.2 を参照してください。

### 3.1 Position 一覧（クイックリファレンス）

| Position | 表示場所 | 用途 |
|----------|----------|------|
| `first` | プライマリナビゲーションの最初 | アプリ起動時の初期画面 |
| `menu` | メニュー（ハンバーガーメニュー内） | 補助的な View |
| `ref` | 参照時のみ表示 | ドリルダウン、編集画面 |

> **重要**: System Generated ViewsにはPosition設定がありません。新規View作成が必要です。

---

## 4. フィルタ・ソート設定

### 4.1 フィルタ設定（Slice経由）

> Row filterはSlice経由で設定します。詳細は `/learnings/appsheet-tips.md` Section 2.3 を参照してください。

**クイックリファレンス**:
1. `Data` → `Slices` → `+ New Slice`
2. Row filter condition を設定
3. `UX` → `Views` → 対象View → `For this data` で Slice を選択

**動的フィルタ（LINKTOFILTEREDVIEW）**:
```
LINKTOFILTEREDVIEW("Orders", [CustomerID] = [_THISROW].[CustomerID])
```

### 4.2 ソート設定

**基本ソート**

UX > Views > [View 名] > Behavior > Sort by:

1. ソート列を追加
2. Ascending（昇順）/ Descending（降順）を選択
3. 複数列でのソートも可能

**式を使用したソート（ORDERBY）**

```
ORDERBY(
  Table,
  [Priority], FALSE,
  [DueDate], TRUE
)
```
- FALSE = 降順
- TRUE = 昇順

### 4.3 グループ化

View 内でデータをグループ化:

1. UX > Views > [View 名] > View Options
2. Group by で列を選択
3. Group aggregate で集計方法を設定（Count, Sum 等）

---

## 5. ベストプラクティス

### 5.1 Do's（推奨事項）

**設計**
- 最重要な View を `first` position に設定
- ユーザーの操作フローに沿った View 配置
- 関連データは Dashboard で統合表示
- モバイルファーストで設計

**フォーム**
- Column Name は短く、Description で詳細説明
- 入力タイプ（Dropdown, Date picker 等）を適切に設定
- 必須項目を明確に
- Show_if で不要な項目を非表示

**パフォーマンス**
- Dashboard の View 数は3-5個に抑える
- 大量データは Table View を使用
- 必要な列のみ表示（Column order で制限）

**一貫性**
- 統一されたカラースキーム
- 同じデータには同じアイコンを使用
- アクションボタンの配置を統一

### 5.2 Don'ts（避けるべきこと）

**設計**
- プライマリナビに View を詰め込みすぎない（5個以下推奨）
- 1つの View に情報を詰め込みすぎない
- ユーザーに不要な Detail View を見せない

**フォーム**
- 長すぎるフォーム（セクション分けを活用）
- 不明確な入力ガイダンス
- 全項目必須にしない

**パフォーマンス**
- Dashboard 内に Dashboard を配置しない
- 大量の画像を Gallery で一度に表示しない
- 複雑な条件式を View filter に多用しない

### 5.3 UX 設計のポイント

1. **ミニマリズム**: 必要な機能のみ表示
2. **一貫性**: 色・アイコン・レイアウトの統一
3. **フィードバック**: ユーザーテストで継続的改善
4. **アクセシビリティ**: 大きなタップターゲット、明確なラベル

---

## 6. セキュリティ設定

AppSheetではテーブル・ビューレベルでセキュリティ設定を行い、データの閲覧・編集権限を制御できます。

### 6.1 Security Filter（テーブルレベル）

**概要**
テーブル全体にフィルタをかけ、ユーザーが閲覧できる行を制限する。

**設定手順**

1. **Data > Tables** で対象テーブルを選択
2. **Security > Security filter** に条件式を入力
3. 保存して動作確認

**条件式例**

| 用途 | 条件式 |
|------|--------|
| 自分のデータのみ表示 | `[メールアドレス]=USEREMAIL()` |
| 管理者はすべて表示、一般は自分のみ | `OR([メールアドレス]=USEREMAIL(), IN(USEREMAIL(), {"admin1@example.com", "admin2@example.com"}))` |
| 特定ステータスのみ表示 | `[ステータス]="公開"` |
| 日付範囲でフィルタ | `AND([申請日]>=TODAY()-30, [申請日]<=TODAY())` |

**PRD 3.5, 11.3 の要件対応例**

```
// Applicationsテーブルの例（居住者は自分の申請のみ、管理者は全件）
OR(
  [メールアドレス]=USEREMAIL(),
  IN(USEREMAIL(), {"admin1@molvia.co.jp", "admin2@molvia.co.jp"})
)
```

### 6.2 Row-level Security（行レベルセキュリティ）

**概要**
行単位でアクセス制御を行う。Security Filterの特殊ケース。

**基本パターン**

```
// 自分のレコードのみ閲覧可能
[メールアドレス]=USEREMAIL()

// 自分の部署のレコードのみ閲覧可能
[部署]=[_THISROW].[ログインユーザーの部署]
```

**マルチテナント対応**

```
// 物件番号で分離（マンション管理の場合）
OR(
  [物件番号]=[ユーザーの物件番号],
  IN(USEREMAIL(), {"admin@..."}))  // 管理者は全物件
)
```

### 6.3 管理者権限の設定

**定義**

管理者のメールアドレスリストを使用して、特定ユーザーに全データへのアクセスや編集権限を付与する。

**管理者リストの設定例**

```
// 管理者判定式（複数箇所で再利用可能）
IN(USEREMAIL(), {"admin1@molvia.co.jp", "admin2@molvia.co.jp"})
```

**適用箇所**

| 設定箇所 | 用途 |
|---------|------|
| Security Filter | 管理者は全データ閲覧可 |
| Editable_If | 管理者のみ編集可能な列 |
| Show_If | 管理者のみ表示される列・View |
| View Show_If | 管理者向けView |

**PRD 11.3 対応例（管理者ダッシュボード）**

```
// UX > Views > 管理者ダッシュボード > View Options > Show_If
IN(USEREMAIL(), {"admin1@molvia.co.jp", "admin2@molvia.co.jp"})
```

### 6.4 口座情報の管理者限定表示（PRD 3.5）

**概要**
個人情報保護の観点から、口座情報などの機密性の高い項目は管理者のみ閲覧可能にする。

**設定手順**

1. **Data > Columns** で口座情報列を選択
2. **Show?** を `Show_If` に変更
3. **Show_If 条件式**:

```
// 管理者または申請者本人のみ表示
OR(
  [メールアドレス]=USEREMAIL(),
  IN(USEREMAIL(), {"admin1@molvia.co.jp", "admin2@molvia.co.jp"})
)
```

**対象列の例**

- 口座_金融機関名
- 口座_支店名
- 口座_番号
- 口座_名義人
- 口座_名義人フリガナ

**注意事項**

| 項目 | 内容 |
|------|------|
| Editable_If | 管理者のみ編集可能に設定（一般ユーザーは閲覧のみ） |
| 利用目的明示 | フォームに「本口座情報は保証金返金処理のみに使用します」を記載 |
| データ保持期間 | 返金完了後30日以内に削除（運用ルール） |

### 6.5 Editable_If（編集権限制御）

**概要**
列単位で編集可能条件を設定する。

**設定手順**

1. **Data > Columns** で対象列を選択
2. **Editable?** を `Editable_If` に変更
3. 条件式を入力

**PRD対応例（リモコン・鍵の日付は管理者のみ入力可）**

```
// Data > Columns > リモコン鍵受渡日 > Editable_If
IN(USEREMAIL(), {"admin1@molvia.co.jp", "admin2@molvia.co.jp"})

// Data > Columns > ステータス > Editable_If
IN(USEREMAIL(), {"admin1@molvia.co.jp", "admin2@molvia.co.jp"})
```

### 6.6 セキュリティ設定のベストプラクティス

**Do's（推奨事項）**

- デフォルトは閲覧制限あり、必要に応じて緩和
- 管理者リストは定数化（複数箇所で再利用）
- 個人情報・機密情報は必ずShow_IfまたはSecurity Filterで保護
- 権限設定後は必ず複数ロールでテスト

**Don'ts（避けるべきこと）**

- すべてのユーザーに全データを公開
- ハードコーディングされた条件式（保守性低下）
- Security FilterとSlice Filterの混同
- 編集権限の過度な制限（運用負荷増加）

**テスト手順**

1. 一般ユーザーでログイン → 自分のデータのみ表示されるか確認
2. 管理者でログイン → 全データが表示されるか確認
3. 他ユーザーのデータが閲覧不可か確認
4. 編集権限が正しく制御されているか確認

---

## 7. 仮想列（Virtual Column）の活用

AppSheetではテーブルに「計算列」や「参照列」を仮想的に追加できます。Google Sheetsにデータを保存せず、アプリ内でのみ表示されます。

### 7.1 Virtual Columnとは

**概要**
データテーブルには存在しないが、AppSheet上で計算・参照により動的に生成される列。

**用途**

| 用途 | 説明 | 例 |
|------|------|-----|
| 参照表示 | 他テーブルのデータを表示 | マンション名、部屋番号 |
| 計算 | 数式による計算結果 | 合計、差分、割合 |
| 連結 | 複数列の結合 | 姓+名、住所1+住所2 |
| 条件分岐 | IF文による動的表示 | ステータスに応じたラベル |

**メリット**

- Google Sheetsのデータ量削減
- 動的な計算・参照が可能
- メンテナンス性向上（マスタ更新時に自動反映）

**注意点**

- 仮想列はソート・フィルタに使用可能
- 保存はされない（アプリ起動時に再計算）
- 複雑な計算は動作速度に影響

### 7.2 作成手順

**基本手順**

1. **Data > Columns** で対象テーブルを選択
2. **+ Add Virtual Column** をクリック
3. **Column name** を入力
4. **App formula** に計算式を入力
5. **Type** を設定（Text, Number, Ref等）
6. 保存

**式の入力**

```
// App formula欄に入力
LOOKUP([区画ID], "ParkingSpaces", "区画ID", "区画番号")
```

### 7.3 LOOKUP()を使ったマンション名表示（PRD 12.2）

**シナリオ**
Applicationsテーブルに物件番号しかないが、フォームや一覧にはマンション名を表示したい。

**設定例**

```
// Applicationsテーブルに仮想列「マンション名」を追加
Column name: マンション名
Type: Text
App formula: LOOKUP([物件番号], "Properties", "物件番号", "物件名")
```

**LOOKUP()構文**

```
LOOKUP(
  [検索値列],            // 例: [物件番号]
  "参照テーブル名",       // 例: "Properties"
  "参照テーブルのキー列", // 例: "物件番号"
  "取得したい列"          // 例: "物件名"
)
```

**複数項目の参照**

```
// 理事長名も取得
Column name: 理事長名
App formula: LOOKUP([物件番号], "Properties", "物件番号", "理事長名")
```

### 7.4 CONCATENATE()を使った宛先表示（PRD 12.2）

**シナリオ**
申請フォームに「〇〇マンション 管理組合 △△理事長 殿」のように動的な宛先を表示したい。

**設定例**

```
// Applicationsテーブルに仮想列「宛先」を追加
Column name: 宛先
Type: LongText
Show type: Page_Header  // フォーム上部に大きく表示
App formula:
CONCATENATE(
  LOOKUP([物件番号], "Properties", "物件番号", "物件名"),
  " 管理組合",
  CHAR(10),  // 改行
  LOOKUP([物件番号], "Properties", "物件番号", "理事長名"),
  " 理事長 殿"
)
```

**CONCATENATE()構文**

```
CONCATENATE(
  "文字列1",
  [列名],
  "文字列2",
  CHAR(10),  // 改行コード
  [別の列]
)
```

**特殊文字**

| 関数 | 説明 |
|------|------|
| `CHAR(10)` | 改行（LF） |
| `CHAR(13)` | 復帰（CR） |
| `CHAR(9)` | タブ |

### 7.5 Formula Column（計算列）の設定

**概要**
Virtual Columnの中でも計算に特化した列。

**例: 合計金額の計算**

```
// Contractsテーブルに「契約期間（月）」を追加
Column name: 契約期間（月）
Type: Number
App formula: (DAYS([契約終了日], [契約開始日]) / 30)
```

**例: ステータスラベルの動的表示**

```
// Applicationsテーブルに「ステータス表示」を追加
Column name: ステータス表示
Type: Text
App formula:
IIF(
  [ステータス]="申請中", "🟡 承認待ち",
  IIF([ステータス]="承認済み", "🟢 承認済み",
  IIF([ステータス]="却下", "🔴 却下",
  "⚪ 完了"))
)
```

**例: 姓名の連結**

```
// Applicationsテーブルに「氏名」を追加
Column name: 氏名
Type: Text
App formula: CONCATENATE([姓], " ", [名])
```

### 7.6 仮想列の表示制御

**Show type活用**

| Show type | 説明 | 適用シーン |
|-----------|------|-----------|
| `Page_Header` | ページ上部に大きく表示 | 宛先、マンション名 |
| `Section_Header` | セクション区切り | カテゴリ名 |
| `Show` | 通常表示 | 一覧の列 |
| `Show_If` | 条件付き表示 | 管理者のみ表示 |

**例: 宛先をPage_Headerとして表示**

```
// Formビューで上部に大きく表示される
Column: 宛先
Show type: Page_Header
```

### 7.7 ベストプラクティス

**Do's（推奨事項）**

- マスタデータはLOOKUP()で参照する
- 頻繁に使う計算式は仮想列化
- 複数列の連結はCONCATENATE()を活用
- 仮想列の命名はわかりやすく（例: 「マンション名」「氏名」）
- Show typeで表示方法を最適化

**Don'ts（避けるべきこと）**

- 複雑すぎる計算式（パフォーマンス低下）
- 仮想列の多用（必要最小限に）
- ネストの深すぎるLOOKUP()
- 同じ計算を複数箇所で実装（仮想列にまとめる）

**パフォーマンス最適化**

- 仮想列は起動時に全行計算される
- 大量データの場合、複雑な計算は避ける
- 必要な画面でのみ表示（Show_Ifで制御）

### 7.8 トラブルシューティング

| 問題 | 原因 | 対処法 |
|------|------|--------|
| 仮想列が表示されない | Type設定ミス | Typeを確認（Text, Number等） |
| LOOKUP()が空白 | キー列の不一致 | 参照元と参照先のキー値を確認 |
| 改行が効かない | CHAR(10)未使用 | CONCATENATE()内にCHAR(10)を挿入 |
| 計算が遅い | 複雑な式 | 式を簡略化、または事前計算 |

---

## 8. トラブルシューティング

Views関連の問題は `/learnings/appsheet-troubleshooting.md` を参照してください。

---

## 9. mutsubi-dx固有の設計パターン

このセクションでは、睦備建設のマンション管理業務に特化したAppSheet Views設計パターンを解説する。

### 8.1 高齢管理人向けUI設計

#### 8.1.1 背景

睦備建設の管理人は61歳以上が8割以上を占めており、ITツール習得に時間を要する。AppSheetアプリのUI設計では、以下のアクセシビリティ要件を遵守する必要がある。

**設計要件**:
- タップターゲット最小サイズ: **44x44px**
- フォントサイズ: **最小18px**
- 色のみに依存しない設計（色覚多様性対応）
- シンプルな操作フロー

#### 8.1.2 AppSheetでの具体的設定方法

**1. タップターゲットサイズの確保**

AppSheetでは標準ボタンサイズが44px以上だが、Custom Actionを作成する場合は以下を確認:

```
UX > Views > [Form/Detail View] > Actions
- アクションボタンの高さを確保（デフォルトで適切）
- 複数アクションを縦積みにしない（横に配置すると誤タップのリスク）
```

**2. フォントサイズの設定**

AppSheetでは`Data > Columns`の`Description`に記載した文字がフォームの質問文として表示される。フォントサイズは変更できないが、以下の工夫で視認性を向上できる:

```
Column Name: 部屋番号（短く）
Description: お住まいの部屋番号を選択してください（明確な指示）
```

**注意**: Column Nameを長くすると表示が崩れるため、Descriptionで詳細を記載する。

**3. 色のみに依存しない設計**

ステータス表示では色だけでなくテキストも併記:

```
列名: 申請ステータス
Format: Text
Initial value: "申請中"

UX > Format Rules で色を設定:
- 申請中 → 灰色 + "申請中"テキスト
- 承認済み → 緑色 + "承認済み"テキスト
- 却下 → 赤色 + "却下"テキスト
```

**4. シンプルな操作フロー**

プライマリナビゲーションは5個以下に抑える:

```
UX > Views で Position を設定:
- ホーム（Dashboard） → Position: first
- 新規申請（Form） → Position: first
- 申請履歴（Deck） → Position: menu
- お問い合わせ（Detail） → Position: menu
```

管理人が日常的に使う機能のみを`first`に配置し、他は`menu`に格納する。

### 8.2 Form固定文言の実装

#### 8.2.1 背景

PRD 2.2では、申請フォームに以下の固定文言を表示する必要がある:

```
管理組合理事長 殿

※ 本申請は区分所有者ご本人のみ申請可能です。
```

AppSheetのFormには標準で固定文言表示機能がないため、`Page_Header`と仮想列を活用する。

#### 8.2.2 Page_Headerを使った実装手順

**Step 1: 仮想列の作成**

`Data > Columns` で新規列を追加:

| 設定項目 | 値 |
|---------|-----|
| Column name | 宛先_固定文言 |
| Type | Text |
| Formula | `"管理組合理事長 殿\n\n※ 本申請は区分所有者ご本人のみ申請可能です。"` |
| Show? | Yes |
| **Show type** | **Page_Header** |

**Step 2: Form Viewでの表示順序設定**

`UX > Views > [Form View] > Column order` で`宛先_固定文言`を最上部に配置:

```
1. 宛先_固定文言（Page_Header）
2. 申請種別
3. 部屋番号
...
```

**結果**: フォーム最上部に固定文言が大きく表示される。

#### 8.2.3 仮想列とLOOKUP()の組み合わせ例

マンション名を動的に表示する場合（PRD 12.2参照）:

```
Column name: 宛先_動的
Type: Text
Formula:
CONCATENATE(
  LOOKUP([物件番号], "Properties", "物件番号", "物件名"),
  " 管理組合",
  CHAR(10),
  LOOKUP([物件番号], "Properties", "物件番号", "理事長名"),
  " 理事長 殿"
)
Show type: Page_Header
```

この設定により、物件ごとに理事長名が自動的に表示される。

### 8.3 管理者限定項目の実装

#### 8.3.1 背景

PRD 2.2.2, 2.2.3では以下の項目を管理者のみが入力可能にする必要がある:

- リモコン・鍵受渡日（契約申請時）
- リモコン・鍵返却日（解約申請時）
- 申請ステータス

居住者が申請する際はこれらの項目は非表示または読み取り専用にし、管理者が後日入力できるようにする。

#### 8.3.2 Editable条件とShow_If条件の組み合わせ

**方法1: Editable条件で制御（推奨）**

`Data > Columns > [リモコン鍵受渡日]`:

| 設定項目 | 値 |
|---------|-----|
| Type | Date |
| Editable? | If → 式を入力 |
| Editable_If | `IN(USEREMAIL(), {"admin1@molvia.co.jp", "admin2@molvia.co.jp"})` |

**結果**:
- 居住者: フィールドが読み取り専用（入力不可）
- 管理者: フィールドが編集可能

**方法2: Show_If条件で非表示にする**

`Data > Columns > [リモコン鍵受渡日]`:

| 設定項目 | 値 |
|---------|-----|
| Type | Date |
| Show? | If → 式を入力 |
| Show_If | `IN(USEREMAIL(), {"admin1@molvia.co.jp", "admin2@molvia.co.jp"})` |

**結果**:
- 居住者: フィールド自体が非表示
- 管理者: フィールドが表示され、入力可能

#### 8.3.3 リモコン鍵受渡日の管理者限定編集例（実装ガイド対応）

**シナリオ**: 契約申請時、居住者はリモコン・鍵受渡日を入力できない。管理者が後日、実際の受渡日を入力する。

**設定**:

1. **Data > Columns > リモコン鍵受渡日**:
   - Type: Date
   - Required?: No（後日入力可能）
   - Editable_If: `IN(USEREMAIL(), {"admin1@molvia.co.jp", "admin2@molvia.co.jp"})`
   - Show_If: `[申請種別]="契約"`（契約申請時のみ表示）
   - Description: "管理者が実際の受渡日を入力します（居住者は入力不要）"

2. **UX > Views > Form View**:
   - Column orderで`リモコン鍵受渡日`を契約申請セクションに配置
   - 居住者が申請時、このフィールドはグレーアウト表示（Editable_Ifにより編集不可）

3. **管理者用View（ステータス更新）**:
   - UX > Views > 新規Form View作成（名前: "ステータス更新_管理者"）
   - For this data: Applications（Slice不要）
   - Column order: ステータス、リモコン鍵受渡日、リモコン鍵返却日のみ
   - Position: menu
   - Show_If: `IN(USEREMAIL(), {"admin1@molvia.co.jp", "admin2@molvia.co.jp"})`

**運用フロー**:
1. 居住者が契約申請（リモコン鍵受渡日は空欄）
2. 管理者が承認
3. 管理人がリモコン・鍵を引渡
4. 管理者が「ステータス更新_管理者」Viewで受渡日を入力
5. ステータスを"完了"に更新

#### 8.3.4 複数管理者の管理（拡張）

管理者リストをハードコーディングではなく、テーブルで管理する方法:

**Step 1: Adminsテーブル作成**

Google Sheetsに新規シート`Admins`を作成:

| メールアドレス |
|--------------|
| admin1@molvia.co.jp |
| admin2@molvia.co.jp |

**Step 2: Editable_If式を変更**

```
IN(USEREMAIL(), Admins[メールアドレス])
```

**メリット**: 管理者の追加・削除をシート編集のみで対応可能（コードやアプリ設定変更不要）

---

## 10. 参照リンク

### 公式ドキュメント

- [Views: The Essentials](https://support.google.com/appsheet/answer/10106688?hl=en) - View の基本概念
- [View types](https://support.google.com/appsheet/answer/10106772?hl=en) - View タイプ一覧
- [Dashboard view type](https://support.google.com/appsheet/answer/10106384?hl=en) - Dashboard 設定
- [Deck and table view types](https://support.google.com/appsheet/answer/10106514?hl=en) - Deck/Table 比較
- [Detail view type](https://support.google.com/appsheet/answer/10106385?hl=en) - Detail View
- [Reference views](https://support.google.com/appsheet/answer/10106516?hl=en) - Ref View
- [View settings](https://support.google.com/appsheet/answer/10107387?hl=en) - View 設定項目
- [Configure and save view configuration](https://support.google.com/appsheet/answer/12726685?hl=en) - View 設定の保存
- [Control row sort order](https://support.google.com/appsheet/answer/10106522?hl=en) - ソート設定
- [Customize input forms](https://support.google.com/appsheet/answer/10106787?hl=en) - Form カスタマイズ
- [Show types](https://support.google.com/appsheet/answer/12463320?hl=en) - Show Type 設定

### デザイン・UX

- [App design: The Essentials](https://support.google.com/appsheet/answer/10107307?hl=en)
- [App design 101](https://support.google.com/appsheet/answer/10099795?hl=en)
- [Desktop design](https://support.google.com/appsheet/answer/12407883?hl=en)
- [AppSheet Training: UX Design 101](https://appsheettraining.com/article/appsheet-training-guide-ux-design-101)

### サンプルアプリ

- [Interactive Dashboard Template](https://www.appsheet.com/templates/How-to-make-an-interactive-dashboard?appGuidString=1d06a130-fc0b-4733-837d-80bdf5ba9f2e)
- [Hierarchical Menu Template](https://www.appsheet.com/templates/How-to-organize-your-views-into-a-hierarchy?appGuidString=0ff478b5-e515-4e69-b00c-7eb1679c9ee7)
- [Filter View Template](https://www.appsheet.com/templates/Allow-the-user-to-filter-a-view-based-on-a-form?appGuidString=91266cff-e843-46b8-a42b-3d85afedb37f)

### 関数リファレンス

- [FILTER()](https://support.google.com/appsheet/answer/10108196?hl=en)
- [ORDERBY()](https://support.google.com/appsheet/answer/10107362?hl=en)
- [LINKTOFILTEREDVIEW()](https://support.google.com/appsheet/answer/10107338?hl=en)

---

## 更新履歴

| 日付 | 内容 |
|------|------|
| 2026-02-16 | 初版作成 |
| 2026-02-16 | System Generated Views の制限、Slice 経由の Row filter 設定を追記 |
| 2026-02-16 | Section 8 追加: mutsubi-dx固有の設計パターン（高齢管理人向けUI、Form固定文言、管理者限定項目） |
| 2026-02-16 | Section 6, 7 追加: セキュリティ設定、仮想列の活用（PRD 3.5, 11.3, 12.2対応） |
