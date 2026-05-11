# hotpepper-blog-draft-slnH000627543

Claude Code Routines による HotPepper Beauty ブログ下書き自動配信システム。
1日1回、ブログ下書き3本を生成し、Google Chat スペースへ Webhook 経由で送信する。

## 構成
- 実行基盤：Claude Code Routines（Cloud）
- 送信先：Google Chat スペース（Webhook 1本、TEST/PROD 共用）
- スケジュール：毎日 07:00 JST（`0 22 * * *` UTC）

## モード切替
`config/run-mode.md` の値で動作が変わる。

| 値 | 見出しプレフィックス | 履歴追記先 |
|---|---|---|
| TEST | 【TEST】 | `data/test-topics.md` |
| PROD | 【PROD】 | `data/past-topics.md` |

Webhook URL は1本のため、送信先は切り替えない。

## Webhook URL の管理
Routines 画面の「指示」プロンプト内に直接記載する。
リポジトリには絶対に保存しない。漏洩時は Chat スペース側で Webhook を再発行し、
Routine の「指示」を書き換える。

## 運用フロー
1. Routine が毎朝起動
2. リポジトリの各種設定・データを読み込み
3. テーマ3本を決定（過去30件と重複しないこと）
4. 記事3本を生成
5. 見出し1通＋記事3通の計4メッセージを Google Chat Webhook へ POST
6. 履歴ファイルに追記して終了
