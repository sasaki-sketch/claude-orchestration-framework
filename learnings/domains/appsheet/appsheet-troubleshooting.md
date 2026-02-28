# AppSheet トラブルシューティング

> 作成日: 2026-02-16

## このファイルの役割

**目的**: AppSheet開発で遭遇する問題と解決策をQ&A形式で集約

**対象読者**: AppSheet実装で問題に遭遇したエージェント・開発者

**このファイルを参照するタイミング**:
- エラーが発生した時
- 設定項目が見つからない時
- 期待通りに動作しない時

**関連ファイル**:
- 実装Tips: `/learnings/appsheet-tips.md`
- Views詳細: `/learnings/appsheet-views-guide.md`

---

## 1. 「〇〇が見つからない」系

### Q: Position設定が見つからない

**症状**: Viewを編集しても Position の設定項目が表示されない

**原因**: System Generated Views には Position 設定がない

**解決策**: 新規Viewを作成する（`Create a new view`）

> 詳細は `/learnings/appsheet-tips.md` Section 2.1, 2.2 を参照

---

### Q: Row filter conditionが見つからない

**症状**: View の設定画面で Row filter の入力欄が見つからない

**原因**: Row filter は Slice で設定する

**解決策**: Sliceを作成し、Viewの「For this data」でSliceを選択

> 詳細は `/learnings/appsheet-tips.md` Section 2.3 を参照

---

### Q: 列がDropdownに表示されない

**症状**: View Options で Primary header などを設定しようとしても、Dropdown に列が表示されない

**原因**: その列の `SHOW?` が `No` になっている

**解決策**:
1. `Data` → 対象テーブル → 対象列を選択
2. `SHOW?` を確認
3. `No` になっている場合、`Yes` に変更
4. または、その列は非表示のまま別の列を使用することを検討

**注意**: SHOW?=No の列は View Options の Dropdown に表示されないが、数式内では参照可能

---

### Q: Move to メニューがない

**症状**: View の並び順を変更しようとして ︙ メニューを開いても Move to がない

**原因**: System Generated Views には Move to オプションがない

**解決策**:
1. 新規Viewを作成（Create a new view）
2. 新規Viewには Move to オプションが表示される
3. または、各Viewの Position 設定で表示順を制御

---

### Q: Sliceが選択できない

**症状**: View の「For this data」で Slice が選択肢に表示されない

**原因**:
- Slice がまだ作成されていない
- Slice の Source Table が View の対象テーブルと異なる

**解決策**:
1. `Data` → `Slices` で Slice が存在するか確認
2. Slice の Source Table が View の対象テーブルと一致しているか確認
3. 一致していない場合、正しい Source Table で Slice を再作成

---

## 2. 設定が反映されない系

### Q: Valid If の式が動作しない

**症状**: Valid If に式を設定したが、フォームで無効な値も入力できてしまう

**原因**:
- 式にエラーがある
- 列名が間違っている
- テーブル名の参照が必要なケースで省略している

**解決策**:
1. Expression Assistant（式の横の？アイコン）でエラーをチェック
2. 列名は正確にコピー（日本語列名は特に注意）
3. `[_THISROW].[列名]` の形式で現在行の値を参照
4. 他テーブルを参照する場合は `テーブル名[列名]` の形式

**デバッグ方法**:
```
# シンプルな式から始めて徐々に複雑にする
Step 1: [契約状況]="空き"
Step 2: AND([契約状況]="空き",[施設種別]=[_THISROW].[施設種別])
Step 3: SELECT(ParkingSpaces[区画ID],AND(...))
```

---

### Q: Show_If が効かない

**症状**: Show_If に条件を設定したが、常に表示される / 常に非表示

**原因**:
- 式がエラーになっている
- 参照している列の値が期待と異なる
- 式が常に TRUE / FALSE を返している

**解決策**:
1. Expression Assistant でエラーチェック
2. 式を別の場所（例: Initial value）で試して値を確認
3. 参照している列の実際の値を確認（大文字小文字、空白に注意）

---

### Q: Bot が発火しない

**症状**: 条件を設定した Bot が実行されない

**原因**:
- Event type が意図と異なる（Adds only vs Updates only）
- Condition 式がエラー
- _THISROW_BEFORE が存在しないタイミング（新規追加時）

**解決策**:
1. Event type を確認（新規: Adds only、更新: Updates only、両方: Adds and Updates）
2. Condition 式を Expression Assistant でチェック
3. 新規追加時は `_THISROW_BEFORE` が空なので注意
4. Bot の Activity Log を確認（Automation → Monitor → Bot Activity）

---

## 3. パフォーマンス・表示系

### Q: アプリが遅い

**症状**: アプリの読み込みや操作が遅い

**原因**:
- データ量が多い
- 複雑な式が多い
- Dashboard に View を詰め込みすぎ

**解決策**:
1. Slice でデータをフィルタして表示量を減らす
2. 複雑な式を Virtual Column から Regular Column（同期時計算）に変更
3. Dashboard の View 数を3-5個に抑える
4. Security Filter でユーザーごとにデータを制限

---

### Q: 画像が表示されない

**症状**: Image 型の列に保存した画像が表示されない

**原因**:
- ファイルパスが間違っている
- Google Drive の共有設定が不適切
- Image folder path の設定問題

**解決策**:
1. Image folder path は空欄でOK（AppSheet が自動作成）
2. Google Drive の共有設定を確認（AppSheet がアクセス可能か）
3. ファイル形式を確認（JPEG, PNG などサポート形式）

---

## 4. 権限・アクセス系

### Q: 「The file was not found」エラー

**症状**: スプレッドシートにアクセスできないエラー

**原因**:
- スプレッドシートが削除された
- 共有設定が変更された
- アカウントの権限がない

**解決策**:
1. Google Drive でスプレッドシートの存在を確認
2. 共有設定で AppSheet アカウントにアクセス権があるか確認
3. `Data` → `Tables` で接続を再設定

---

### Q: USEREMAIL() が空を返す

**症状**: USEREMAIL() で取得したメールアドレスが空

**原因**:
- ユーザーがサインインしていない
- プレビューモードで実行している

**解決策**:
1. `Settings` → `Security` → `Require user signin` を有効化
2. プレビューではなく実際のアプリURLでテスト
3. デバッグ時は `USEREMAIL()` の代わりに固定値でテスト

---

## 5. デプロイ・公開系

### Q: アプリが Deployed にならない

**症状**: Deploy ボタンを押しても Deployment Check でエラー

**原因**:
- 式にエラーがある
- 必須設定が不足している
- プランの制限

**解決策**:
1. `Manage` → `Monitor` → `Audit History` でエラー詳細を確認
2. 各テーブル・列の式をチェック
3. `Settings` の必須項目を確認

---

## 更新履歴

| 日付 | 内容 |
|------|------|
| 2026-02-16 | 初版作成 |
