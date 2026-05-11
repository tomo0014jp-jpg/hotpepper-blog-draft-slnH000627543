# 送信先設定

## 方式
Google Chat スペースの Incoming Webhook（1本のみ）

## URL の保管場所
Claude Code Routines の「指示」プロンプト内に直接記載。
**リポジトリには Webhook URL を絶対に保存しないこと。**

## TEST / PROD の区別
Webhook URL は共用。`config/run-mode.md` の値に従い、
メッセージ先頭の見出しタグで TEST / PROD を識別する。

| run-mode | 見出し先頭タグ |
|---|---|
| TEST | 【TEST】 |
| PROD | 【PROD】 |

## 失効・再発行
Webhook URL が漏洩・失効した場合は、
1. Chat スペース → アプリと統合 → Webhook を削除
2. 同一名で再発行し、新しい URL を取得
3. Routine の「指示」プロンプト内の URL を新URLで上書き
