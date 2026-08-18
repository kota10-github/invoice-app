
# 引き継ぎ
最初に HANDOFF.md を読むこと(Macからの移行プロジェクト。経緯と注意点が書いてある)。

## 運用ルール(自動記録)
- 仕様・設計・運用に関わる決定をしたら、ユーザーに言われなくてもCLAUDE.md末尾の決定ログに日付付きで1-2行追記すること
- 大きな作業の区切りでは現状を HANDOFF.md に要約すること(会話圧縮対策)

## 決定ログ
- 2026-08-05: 取引先セクションのUI再構成。「＋新規追加」を見出し右上へ隔離。「名称変更」ボタン廃止→プロフィール名を編集欄(`#clientProfileName`)化し、タップ→blurで改名(`commitProfileRename`、`persistProfile`経由で自動保存/DLと同じ永続化)。切替はネイティブselectだと名前が二重表示・見切れで不評だったため、自作ドロップダウン(「切替▾」ボタン`toggleProfileMenu`→`#profileMenu`一覧→`switchToProfile(id)`)に変更。削除はゴミ箱アイコン。デフォルトは改名不可(名前欄readOnly)。旧`renameClientProfile`/`switchClientProfile`/`refreshClientProfileSelect`とネイティブselectは削除。
- 2026-08-05: モダンの「プレビューの数字がPDF出力より太い」問題に対応。**当初 font-family から `-apple-system` を外し `'Helvetica Neue', Arial` に統一したが逆効果(プレビューがさらに太化)→ revert(7f9a403)**。真因は font-family ではなく**フォントスムージングの差**(macOS画面=サブピクセルで太い / PDF=グレースケールで細い)。対処として `.invoice-page` に `-webkit-font-smoothing: antialiased; -moz-osx-font-smoothing: grayscale;` を追加し、**画面側だけを細い描画にしてPDF(良い方)に一致**させる(PDF出力=window.print には影響しない)。フォント系はユーザーが特に細かいので実機目視で確認すること。
- 2026-08-05: 口座番号(bankAccount)に形式バリデーション追加。有効=数字ちょうど7桁(`/^\d{7}$/`、trim)。**必須項目(※)のため空欄は不可**(登録番号は任意で空欄OKだったのと対照的)。不正時は明示保存とDLをブロック、自動保存は継続。登録番号・口座番号のガードを `ensureValidOrBlock()` に統合し、複数不正は1回のアラートでまとめて指摘。入力欄下に赤エラー+赤枠(初期ロード時は非表示、操作/保存時に表示)。
- 2026-08-05: 登録番号(issuerRegNo=インボイス番号/適格請求書発行事業者登録番号)に形式バリデーション追加。有効=空欄 または「T+13桁の数字」(`/^T\d{13}$/`、trim+大文字化で正規化)。不正時は明示保存(saveProfileの非silent)とDL(exportPDF)をブロック、自動保存(silent)は継続(データ損失回避のためユーザー決定)。空欄は「任意」項目として許可。入力欄下に赤エラー文言+赤枠表示。共通ガードは `ensureRegNoOrBlock()`。
- 2026-07-19: 取引先プロフィール消失バグを修正。根本原因は保存系関数(saveProfile/switchClientProfile/add/rename/delete)が保存ベースを `cloudGet()` から取っていたこと。本番(GitHub Pages)はクラウド無効で `cloudGet` が常に null のため、保存・切替・自動保存のたびに `existing={clients:{}}` から作り直され、編集中の1件以外を全消去していた。→ `getStoredProfileData()`(localStorage優先)をベースにし、全書き込みを `persistProfile()` に集約。併せて applyProfileFieldValues の内容ブリード修正、上書き前の世代バックアップ(`inv_bak_*` 直近5世代)を追加。今回消えたデータは localStorage・jsonblob(404)とも生存コピー無く復旧不能をユーザー了承済み。
