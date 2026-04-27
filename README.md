# hotpepper-blog-draft-slnH000627543

HotPepper Beauty 店舗ブログ向けに、毎朝3本分の下書きを生成し、メールで送付する Claude Code Routines 用リポジトリです。

## 概要

- **対象媒体**: HotPepper Beauty 店舗ブログ（[クラブオーサム(Club Awesome!)](https://beauty.hotpepper.jp/kr/slnH000627543/)）
- **頻度**: 毎朝1回 / 1日3本 / メール1通
- **出力**: テキストのみ（画像なし）
- **メール送信方式**: **Google Apps Script Webhook** に HTTP POST する方式
  （Claude 側に Gmail コネクタを用意する必要はありません）

## 現在の運用状況

**🚧 現在は TEST モードで運用中です。**

- TEST モードでは送付先は `tomo0014jp@gmail.com`（大石氏個人）のみ
- 顧客（サロン）には送られません
- TEST モードで生成したテーマは本番の重複判定に含まれません

## 構成

```
hotpepper-blog-draft-slnH000627543/
├── CLAUDE.md                          Routine 運用ルール（Claude が必ず読む）
├── README.md                          本ファイル
├── config/
│   ├── salon.md                       サロン基本情報
│   ├── tone.md                        文体ルール／薬機法・景表法上の NG 表現
│   ├── recipients.md                  送付先（TEST / PROD）と Webhook 宛先制御メモ
│   └── run-mode.md                    実行モード切替（TEST / PROD）
├── templates/
│   ├── post-template.md               ブログ1本分のテンプレ
│   ├── email-template.md              メール件名・本文のテンプレ
│   └── webhook-payload-example.json   Webhook POST ペイロードのサンプル（token ダミー）
└── data/
    ├── menu.md                        メニュー一覧
    ├── staff.md                       スタッフ情報
    ├── campaigns.md                   キャンペーン情報
    ├── past-topics.md                 PROD 用の重複判定履歴
    └── test-topics.md                 TEST 用の生成履歴
```

---

## メール送信方式（Apps Script Webhook）

### Webhook URL

```
https://script.google.com/macros/s/AKfycbwQEur3HW4usdPe4v8NSJyG9v6S4uWJPmo2pVcQOBR23cpg9ehu-aUFFPznNJgcTnHL/exec
```

### POST ペイロード

```json
{
  "token": "<<WEBHOOK_TOKEN>>",
  "mode": "TEST",
  "subject": "【TEST】【ブログ下書き 2026-04-27】クラブオーサム(Club Awesome!) 3本分",
  "body": "（3本分のブログ下書き本文）"
}
```

詳細は [CLAUDE.md](./CLAUDE.md) §6 と [templates/webhook-payload-example.json](./templates/webhook-payload-example.json) を参照。

### 🔐 WEBHOOK_TOKEN の取り扱い

- **WEBHOOK_TOKEN はリポジトリに絶対にコミットしないでください。**
  - `*.md` `*.json` どこにも書かない（サンプル JSON もダミー値のみ）。
  - コミットメッセージ・PR・Issue にも書かない。
- **Routine 実行時に外部から渡してください。** 想定方法:
  - Claude Code Routines のプロンプト内に直接埋め込む（実行ごとに）、または
  - 実行環境の環境変数（例: `WEBHOOK_TOKEN`）として注入し、Routine 起動時に Claude へ参照させる。
- 万一誤ってトークンを push した場合は、Apps Script 側でトークンをローテーションし、
  リポジトリ履歴からも除去してください。

---

## モード切替

`config/run-mode.md` の値を `TEST` または `PROD` に書き換えることで切り替えます。

詳細は [CLAUDE.md](./CLAUDE.md) を参照してください。

---

## 初回テスト手順（TEST モード）

リポジトリを clone した直後、最初に動作確認するときの手順です。

1. **WEBHOOK_TOKEN を入手する。**
   Apps Script 側で発行したトークンを、安全な経路で受け取ります。
   （以後、ターミナル履歴やチャット履歴に残らないよう注意）

2. **`config/run-mode.md` が `TEST` になっていることを確認する。**
   ```bash
   cat config/run-mode.md
   ```

3. **`config/recipients.md` の TEST 宛先が `tomo0014jp@gmail.com` になっていることを確認する。**

4. **手動で Webhook 疎通テスト**（任意）。
   トークンを環境変数に入れ、curl で疎通確認します。
   ```bash
   export WEBHOOK_TOKEN="xxxxxxxxxxxxxxxx"   # ← 入力履歴に残らないよう注意
   curl -X POST \
     -H "Content-Type: application/json" \
     -d "{\"token\":\"$WEBHOOK_TOKEN\",\"mode\":\"TEST\",\"subject\":\"【TEST】疎通確認\",\"body\":\"これはテスト送信です。\"}" \
     https://script.google.com/macros/s/AKfycbwQEur3HW4usdPe4v8NSJyG9v6S4uWJPmo2pVcQOBR23cpg9ehu-aUFFPznNJgcTnHL/exec
   unset WEBHOOK_TOKEN
   ```
   `tomo0014jp@gmail.com` に件名「【TEST】疎通確認」のメールが届けば OK。

5. **Claude Code Routines を起動する。**
   Routine のプロンプトに WEBHOOK_TOKEN を渡し、CLAUDE.md の手順通りに3本生成 → Webhook POST。

6. **届いたメールを確認**:
   - 件名冒頭が `【TEST】` か
   - 末尾に「※本メールはTEST送信です。お客様には送信されていません。」が入っているか
   - 3記事が1通のメールにまとまっているか
   - `data/test-topics.md` に当日のテーマが追記されたか
   - `data/past-topics.md` には**追記されていない**こと

7. **数日 TEST モードで運用**して内容を確認したのち、本番（PROD）切替へ。

---

## 本番（PROD）切替前のチェックリスト

- [ ] `config/recipients.md` の PROD 宛先を実際の送付先に更新
- [ ] Apps Script 側でも PROD 用の宛先制御を実装・確認
- [ ] `config/salon.md` のサロン情報を埋める（TODO 残を確認）
- [ ] `data/menu.md` のメニュー情報を埋める
- [ ] `data/staff.md` のスタッフ情報を埋める
- [ ] `data/campaigns.md` のキャンペーン情報を埋める
- [ ] TEST モードで数日運用し、内容を確認
- [ ] WEBHOOK_TOKEN を Routine 実行環境に確実に渡せる状態か再確認
- [ ] `config/run-mode.md` を `PROD` に変更
