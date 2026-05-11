# HotPepper Beauty ブログ下書き生成ジョブ

## ゴール
HotPepper Beauty 店舗ブログ向けに、1日3投稿分の下書きを生成し、
指定の Google Chat スペースへ Webhook 経由で送付する。

## 送信方式
- Google Chat Webhook（HTTP POST）
- Webhook URL は Claude Code Routines の「指示」プロンプト内に直接記載される。
  リポジトリには Webhook URL を絶対に保存しない。
- Webhook は TEST/PROD 共用。送信先の切替は行わず、`config/run-mode.md` の値で
  メッセージ見出しのプレフィックス（【TEST】/【PROD】）と履歴ファイルの書き分けのみを制御する。

## 入力データ
- 店舗情報：`config/salon.md`
- 実行モード：`config/run-mode.md`
- 送信先メモ：`config/chat-target.md`
- 文体・NG：`config/tone.md`
- メニュー：`data/menu.md`
- スタッフ：`data/staff.md`
- キャンペーン：`data/campaigns.md`
- 過去ネタ（直近30件、重複NG）：`data/past-topics.md`
- テスト生成履歴：`data/test-topics.md`

## 出力要件
- 記事3本。各記事は `templates/post-template.md` のフォーマットに従う。
- 3本のテーマは被らないこと。過去30日のテーマとも被らないこと。
- 薬機法・景品表示法に抵触する表現を含めないこと。
- 絵文字は使用しない。記号（*太字*、_斜体_、`code`）は Google Chat の簡易マークダウンに従う。
- 1本あたり 600〜1000 文字。

## 実行手順
1. 起動直後に `config/run-mode.md` を読み上げ、現在のモードを明示する（TEST/PRODのどちらか）。
2. 上記入力データを全て読み込む。
3. 今日の3テーマ案を出す（テーマ、ターゲット、訴求軸）。
   - `data/past-topics.md` の直近30件と被らない。
   - TESTモードであっても本番履歴との重複は避ける。
4. 各テーマで記事下書きを生成する（`templates/post-template.md` に従う）。
5. `templates/chat-message-template.md` に沿って、見出し1通＋記事3通の合計4メッセージを組み立てる。
   - 見出しの先頭タグは run-mode.md の値で【TEST】または【PROD】を必ず付与。
6. Routine プロンプトに記載された Google Chat Webhook URL へ以下の順で4回 POST する。
   - ① 見出しメッセージ
   - ② 記事1
   - ③ 記事2
   - ④ 記事3
   - メソッド：POST
   - ヘッダー：Content-Type: application/json; charset=UTF-8
   - ボディ：{ "text": "<該当メッセージ本文>" }
   - いずれかが 200 以外を返した場合は失敗とし、以降の履歴追記を行わずに終了する。
7. 全POSTが 200 で完了した場合のみ、履歴を反映する。
   - TESTモード：今日の3テーマを `data/test-topics.md` に追記する。
     `data/past-topics.md` は変更しない。
     Routineプロンプトに記載された GITHUB_PAT を使い、
     以下のコマンドで main に直接 commit & push する。
       git add data/test-topics.md
       git commit -m "chore: append test topics YYYY-MM-DD"
       git push https://x-access-token:${GITHUB_PAT}@github.com/tomo0014jp-jpg/hotpepper-blog-draft-slnH000627543.git HEAD:main
   - PRODモード：今日の3テーマを `data/past-topics.md` に追記する。
     上記と同じ方式で main に直接 commit & push する（メッセージ：`chore: append topics YYYY-MM-DD`）。
   - push に失敗した場合：履歴反映は諦め、見出しChatメッセージの末尾に
     「※ 履歴の push に失敗しました」と明記する。ただし送信完了済みのChatメッセージは取り消さない。
   - GITHUB_PAT 値は応答・ログ・コミットメッセージ・Chatメッセージ等に絶対に出力しない。

## 制約
- 画像は扱わない（テキストのみ）
- 4メッセージは必ず同一順序で送信する（見出し → 記事1 → 記事2 → 記事3）
- 1メッセージは 4,000 文字を超えないこと（Google Chat の上限は 4,096 字。余裕を持って 4,000 を上限とする）
- 失敗時は履歴を更新せず、原因を最後のログに明記して終了する
- TESTモードで生成したテーマは本番用の重複判定（past-topics.md）に含めない
- Webhook URL はログ・コミットメッセージ・出力など外部に絶対に出力しない
