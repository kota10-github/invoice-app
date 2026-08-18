# 引き継ぎメモ: invoice

> VPS上のパス: `/root/products/invoice`（元は Mac の `/Users/kota/invoice`）

## このプロジェクトは何か(1-3行)

日本語の請求書をブラウザ上で作成し、PDF出力（`window.print()` によるベクターテキスト印刷）できる静的Webアプリ「請求書作成ツール」。
「モダン」「クラシック」の2テンプレートを持ち、取引先プロファイルごとに発行元・振込先・品目など全フィールドを切り替えて保存できる。
リポジトリ: `git@github.com:kota10-github/invoice-app.git`／本番: https://kota10-github.github.io/invoice-app/

## 技術スタック

- **純粋な静的サイト**。ビルド不要（`package.json` の build は echo のみ）
- 実体は `public/index.html` 1ファイル（約2,200行）に HTML/CSS/JS すべてインライン。外部JSライブラリなし
- フォント: Google Fonts（Noto Sans JP / Noto Serif JP）を `@import` で読み込み
- データ永続化: **ブラウザの localStorage のみ**（サーバーAPIなし）
- 開発サーバー: `npx serve public -l 3000`、または `.claude/launch.json` の `python3 -m http.server 8766`（cwd: public）
- デプロイ: **GitHub Pages**。`main` への push で GitHub Actions（`.github/workflows/deploy.yml`、peaceiris/actions-gh-pages、publish_dir: `./public`）が `gh-pages` ブランチへ自動デプロイ

## 最近やっていたこと

セッション履歴（〜2026-07-11）と git log より:

- **2026年5月頃（worktree `claude/heuristic-tharp-dd18df`、mainにマージ済み）**: 取引先プロファイル機能を実装。切り替え時に取引先名・税設定・備考だけでなく発行元・振込先・品目・数量・単価まで全部切り替わるよう `switchClientProfile` / `addClientProfile` / `renameClientProfile` / `deleteClientProfile` を全面改修。既存データは自動マイグレーション対応
- **2026-06〜07-10**: ログイン機能を撤廃しアプリ直接表示に変更。その際に保存キー変更でユーザーデータが見えなくなる事故があり、旧キー（メールハッシュ→固定キー `inv_default_user`）からの自動移行・復旧コードを実装。PDF出力をベクターテキスト化
- **2026-07-10〜11（最終セッション）**: 参考請求書（実物PDF）に見た目を近づける微調整の連続。フォントサイズ・太さ・字間・余白の調整、モダンの薄いグレー文字をすべて `#333` の黒に統一（最終コミット `1ae10ed`）。各変更は視覚確認のうえデプロイ完了まで確認済み
- **2026-07-19**: 取引先プロフィール消失バグを修正（未コミット/未デプロイ）。根本原因=保存系関数が `cloudGet()` をベースにしており、本番はクラウド無効で常に null のため保存・切替・自動保存のたびに編集中1件以外を全消去していた。`getStoredProfileData()`（localStorage優先）+ `persistProfile()` に集約して修正。applyProfileFieldValues の内容ブリードも修正。上書き前の世代バックアップ `inv_bak_*`（直近5世代）を追加。実コードの関数本体をnodeで実行する回帰テストで3件保持・ブリード無しを確認。デプロイ済み（コミット `e500f45`、GitHub Pages 反映確認済み）。本番でも Claude-in-Chrome によりダミー取引先2件を注入→切替＋自動保存の経路を実行→3件全保持を確認して合格（ダミーは撤去済み・実データ無傷）。今回消えた実データは localStorage・jsonblob（404）とも生存コピー無く復旧不能をユーザー了承済み

## 未完了・進行中のタスク

- **明確な未完了タスクは見当たらない**。最終セッションは「デプロイ完了しました」で正常終了しており、作業ツリーはクリーン、`main` は `origin/main` と一致
- worktree `.claude/worktrees/heuristic-tharp-dd18df`（ブランチ `claude/heuristic-tharp-dd18df`）が残っているが、内容はすべて main にマージ済みなので削除して問題ないはず（推測）
- 今後も「参考請求書に見た目を寄せる」系の微調整依頼が続く可能性が高い（推測。直近の作業パターンより）

## 注意点(環境変数、デプロイ方法など判明しているもの)

- **環境変数・シークレットは不要**（GitHub Actions は標準の `GITHUB_TOKEN` のみ使用）
- **デプロイ方法**: `main` に push するだけ。Actions が `public/` を GitHub Pages に公開。デプロイ完了は `gh run` で監視していた
- **本番は GitHub Pages の静的ホスティング**: `vercel.json` に jsonblob.com への `/api/blob` リライト設定が残っているが、本番（GitHub Pages）にはサーバーAPIが存在せず `cloudSync`/`cloudInit` は silent fail する。**実際のデータ保存は localStorage のみ**という前提でストレージ関連の変更をすること（vercel.json は過去に Vercel を使っていた名残と思われる（推測））
- **ユーザーデータを絶対に消さない**: localStorage のキー名やデータ構造を変えるときは、旧キー・旧構造からの自動移行コードを必ず同時に実装し、旧データは移行後も削除しない（過去にデータ消失事故があり、ユーザーからルール化を指示されている）
- **視覚確認してから完了報告**: 修正はプレビューで本番相当の状態（localStorage シード等）を再現し、スクリーンショット・DOM検証で確認してからコミット・完了報告する（これもユーザー指示によるルール）
- CLAUDE.md / README は存在しない。上記ルールは `~/.claude/projects/-Users-kota-invoice/memory/` のメモリファイル由来（VPS側には引き継がれないため本メモに転記済み）
