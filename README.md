# hotpepper-blog-draft-slnH000627543

HotPepper Beauty 店舗ブログ向けに、毎朝3本分の下書きを生成し、メールで送付する Claude Code Routines 用リポジトリです。

## 概要

- **対象媒体**: HotPepper Beauty 店舗ブログ（[クラブオーサム(Club Awesome!)](https://beauty.hotpepper.jp/kr/slnH000627543/)）
- **頻度**: 毎朝1回 / 1日3本 / メール1通
- **出力**: テキストのみ（画像なし）
- **メール送信方式**: **Claude Code の Gmail コネクター（MCP）の `send_email` ツールを直接呼び出す**
  方式です。以前の Apps Script Webhook 方式は廃止しました。

## 現在の運用状況

**🚧 現在は TEST モードで運用中です。**

- TEST モードでは送付先は `tomo0014jp@gmail.com`（大石氏個人）のみ
- 顧客（サロン）には送られません
- TEST モードで生成したテーマは本番の重複判定に含まれません

## 構成

```
hotpepper-blog-draft-slnH000627543/
├── CLAUDE.md                  Routine 運用ルール（Claude が必ず読む）
├── README.md                  本ファイル
├── config/
│   ├── salon.md               サロン基本情報
│   ├── tone.md                文体ルール／薬機法・景表法上の NG 表現
│   ├── recipients.md          送付先（TEST / PROD）
│   └── run-mode.md            実行モード切替（TEST / PROD）
├── templates/
│   ├── post-template.md       ブログ1本分のテンプレ
│   └── email-template.md      メール件名・本文のテンプレ
└── data/
    ├── menu.md                メニュー一覧
    ├── staff.md               スタッフ情報
    ├── campaigns.md           キャンペーン情報
    ├── past-topics.md         PROD 用の重複判定履歴
    └── test-topics.md         TEST 用の生成履歴
```

---

## メール送信方式（Gmail コネクター / MCP）

Routine 実行時、Claude は **Gmail コネクター**を通じてメールを送信します。

### 使用するツール

- **`send_email`** — メールを直接送信（推奨 / 自動運用向け）
- **`create_draft`** — Gmail 下書きとして保存（送信前確認したいとき）

### ツール呼び出しのパラメータ

| フィールド | 値 |
|---|---|
| `to` | TEST モード: `tomo0014jp@gmail.com` ／ PROD モード: `config/recipients.md` の PROD 宛先 |
| `subject` | TEST モード: `【TEST】【ブログ下書き YYYY-MM-DD】クラブオーサム(Club Awesome!) 3本分` ／ PROD モード: 冒頭の `【TEST】` を外す |
| `body` | `templates/email-template.md` に沿って組み立てた3本分の下書き本文（プレーンテキスト・絵文字なし） |

詳細は [CLAUDE.md](./CLAUDE.md) §6 と [templates/email-template.md](./templates/email-template.md) を参照。

### Gmail コネクター接続

1. claude.ai の [Connectors](https://claude.ai/customize/connectors) ページで Gmail コネクターを接続。
2. Routine 設定（[管理画面](https://claude.ai/code/routines)）で当該 Routine に Gmail コネクターを紐付け。
3. OAuth 認可は Routine 所有者（大石氏）の Gmail 権限で行われます。

---

## モード切替

`config/run-mode.md` の値を `TEST` または `PROD` に書き換えることで切り替えます。

詳細は [CLAUDE.md](./CLAUDE.md) を参照してください。

---

## 初回テスト手順（TEST モード）

1. **Gmail コネクターが Routine に接続されていることを確認**
   - 未接続の場合、`send_email` が呼べず Routine が失敗します（§10 で送信中止 → 履歴非追記）。
2. **`config/run-mode.md` が `TEST` になっていることを確認**
3. **`config/recipients.md` の TEST 宛先が `tomo0014jp@gmail.com` であることを確認**
4. **Routine を手動実行**（[管理画面](https://claude.ai/code/routines) の「Run now」 または daily スケジュール待ち）
5. **届いたメールを確認**:
   - 件名冒頭が `【TEST】` か
   - 末尾に「※本メールはTEST送信です。お客様には送信されていません。」が入っているか
   - 3記事が1通のメールにまとまっているか
   - 本文に絵文字が含まれていないか（プレーンテキスト文字化け防止）
   - `data/test-topics.md` に当日のテーマが追記されたか
   - `data/past-topics.md` には**追記されていない**こと

---

## 本番（PROD）切替前のチェックリスト

- [ ] `config/recipients.md` の PROD 宛先を実際の送付先に更新
- [ ] `config/salon.md` のサロン情報を埋める（TODO 残を確認）
- [ ] `data/menu.md` のメニュー情報を埋める
- [ ] `data/staff.md` のスタッフ情報を埋める
- [ ] `data/campaigns.md` のキャンペーン情報を埋める
- [ ] TEST モードで数日運用し、内容を確認
- [ ] Gmail コネクターが Routine に接続されていることを再確認
- [ ] `config/run-mode.md` を `PROD` に変更

⚠️ Gmail コネクター方式では、宛先の最終ガードは Claude 側のみ（CLAUDE.md と recipients.md の参照）です。
PROD 宛先は誤りなく `config/recipients.md` に記入してください。
