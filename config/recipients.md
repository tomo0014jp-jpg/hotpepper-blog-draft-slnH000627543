# recipients.md — メール送付先（Apps Script Webhook 経由）

> 本リポジトリでは、メール送信を **Google Apps Script Webhook 経由** で行います
> （Gmail コネクタは使用しません）。
> 実際にメールアドレスへ配信するのは Apps Script 側（`MailApp.sendEmail`）で、
> Claude（クライアント側）は Webhook へ `mode` と本文を POST するだけです。
>
> このファイルは「想定される送付先」を運用ドキュメントとして明示するためのもので、
> 宛先の最終制御は **Apps Script 側でも `mode` をもとに二重に行う**前提です。

---

## TEST 宛先（テスト運用中の送付先）

```
tomo0014jp@gmail.com
```

- 上記は大石氏の個人アドレスです。
- TEST モード（`config/run-mode.md` が `TEST`、Webhook ペイロードの `mode` が `"TEST"`）では、
  **この宛先のみ**にメールが届く想定です。

## PROD 宛先（本番運用の送付先）

```
（未設定 — 後でサロン担当者のメールアドレスを記入）
```

- PROD モード移行前に、上記を実際の本番送付先（サロン担当者のメールアドレス）に書き換えてください。
- 複数アドレスにする場合は、改行して列挙してください。
- **Apps Script 側の PROD 用送付先設定にも、同じアドレスを反映**してください
  （リポジトリ側だけ書き換えても Apps Script 側に反映されないと送信先が変わりません）。

---

## 宛先制御の二重化（重要）

メールを実際に送るのは Apps Script 側ですが、Claude（クライアント側）も以下を必ず守ること:

- **TEST モード**:
  - Webhook ペイロードの `mode` には必ず `"TEST"` を入れる。
  - Apps Script 側は `mode === "TEST"` のとき TEST 宛先（`tomo0014jp@gmail.com`）のみへ送信する設計。
  - Claude が `body` の中に PROD 宛先（顧客のメールアドレス）を書くこともしない。
- **PROD モード**:
  - Webhook ペイロードの `mode` には必ず `"PROD"` を入れる。
  - Apps Script 側は `mode === "PROD"` のとき PROD 宛先のみへ送信する設計。
  - TEST 宛先には送らない。
- **どちらのモードでも、Claude が宛先を独自判断で追加・変更することはしない。**
- 同じメールを TEST 宛先と PROD 宛先の両方に送ることは**しない**。

この二重制御により、「TEST モードなのに顧客アドレスへ送ってしまう」事故をクライアント側
（Claude）と Apps Script 側の両方で防ぐ設計です。

---

## メモ

- Webhook URL とペイロード仕様は `CLAUDE.md` §6 を参照。
- WEBHOOK_TOKEN は **このファイルにも絶対に書かない**。Routine 実行時に外部から注入する。
