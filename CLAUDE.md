# CLAUDE.md — HotPepper Beautyブログ下書き自動メール送付システム

このファイルは Claude Code Routines が本リポジトリで動作するときの**運用ルール**をまとめたものです。
Routine 実行時、Claude はまずこの CLAUDE.md を読み、ここに書かれているルールを必ず守って動作します。

---

## 1. このリポジトリの目的

HotPepper Beauty 店舗ブログ向けに、**毎朝1回**、ブログ下書きを **3本** 生成し、
**メール1通にまとめて送付**するための Claude Code Routines 用リポジトリです。

- 対象媒体: HotPepper Beauty 店舗ブログ
- 出力単位: 1日3本 / メール1通
- 出力物: テキストのみ（画像は扱わない）
- 想定読者: サロンに通うお客様（既存・見込み）
- メール送信方式: **Google Apps Script Webhook への HTTP POST**（Gmailコネクタは使用しない）

---

## 2. Routine 実行時の手順

毎朝の Routine 起動時、Claude は以下の順で動作します。

1. `config/run-mode.md` を読み、現在のモード（**TEST** / **PROD**）を確認する。
2. `config/recipients.md` を読み、送付先メールアドレスを確認する。
   - **TEST モード** → TEST 宛先のみに送付
   - **PROD モード** → PROD 宛先にのみ送付
3. `config/salon.md` `config/tone.md` `data/menu.md` `data/staff.md` `data/campaigns.md` を読み、
   サロンの基本情報・文体ルール・素材を把握する。
4. 重複判定用ファイルを読む。
   - **TEST モード** → `data/test-topics.md`
   - **PROD モード** → `data/past-topics.md`
5. 上記と重複しないテーマで **3本分の下書き** を `templates/post-template.md` に沿って生成する。
6. `templates/email-template.md` に従い **1通のメール本文（テキスト）** にまとめる。
7. **Apps Script Webhook URL に HTTP POST する**（詳細は §6）。
   - メール送信は Webhook 側（Apps Script）が `MailApp.sendEmail` で行う。
   - Claude 自身はメールを直接送らない。
8. Webhook のレスポンスを確認する。
   - 200 系 + `{"ok": true}` 等の成功応答なら処理を続行。
   - 失敗（4xx/5xx、`{"ok": false}`、タイムアウト等）の場合は §10 の手順で扱う。
9. 生成したテーマ（タイトル・要旨）を、モードに応じたファイルへ追記する。
   - **TEST モード** → `data/test-topics.md` に追記
   - **PROD モード** → `data/past-topics.md` に追記
   - ※TEST モードで生成したテーマは **絶対に** `data/past-topics.md` に書かない

---

## 3. TEST / PROD の切り替えルール

- 切り替えは `config/run-mode.md` の値を `TEST` または `PROD` に書き換えることで行う。
- **初期値は必ず `TEST`**。
- 本番運用への切り替えはユーザー（大石氏）の明示的な指示があってから行うこと。
- Claude が自分の判断で `TEST` → `PROD` に切り替えてはいけない。

---

## 4. TEST モードの厳守事項

TEST モードで動作する場合、Claude は以下を**必ず**守ること。

- **顧客（サロン側）宛先には絶対に送らない。**
  Webhook へ POST する `body` の宛先は `config/recipients.md` の TEST 宛先（`tomo0014jp@gmail.com`）のみ。
  ※宛先の最終制御は Apps Script 側でも実施されるが、Claude も必ずクライアント側で守ること。
- **Webhook ペイロードの `mode` フィールドは `"TEST"` を必ず指定する。**
  Apps Script 側で `mode === "TEST"` の場合のみ TEST 宛先に絞る制御が入る前提。
- **メール件名（`subject`）の冒頭に `【TEST】` を付ける。**
  例: `【TEST】【ブログ下書き YYYY-MM-DD】クラブオーサム(Club Awesome!) 3本分`
- **メール本文（`body`）の末尾に「※本メールはTEST送信です。お客様には送信されていません。」を必ず入れる。**
- **生成したテーマは `data/test-topics.md` のみに記録する。**
  `data/past-topics.md` には絶対に追記しない。
- TEST モードの生成物は本番の重複判定対象に含めない。

---

## 5. PROD モードのルール

- 送付先は `config/recipients.md` の PROD 宛先のみ。
- Webhook ペイロードの `mode` は `"PROD"`。
- 件名に `【TEST】` は付けない。
- 生成したテーマは `data/past-topics.md` に追記する。

---

## 6. メール送信方式（Apps Script Webhook）

### 6.1 送信先 Webhook URL

```
https://script.google.com/macros/s/AKfycbwQEur3HW4usdPe4v8NSJyG9v6S4uWJPmo2pVcQOBR23cpg9ehu-aUFFPznNJgcTnHL/exec
```

- このURL自体はリポジトリに記載してOK（公開しても、token がなければメールは飛ばない設計）。

### 6.2 リクエスト

- メソッド: **POST**
- Content-Type: **application/json**
- Body: 以下の JSON 形式

```json
{
  "token": "<<WEBHOOK_TOKEN>>",
  "mode": "TEST",
  "subject": "【TEST】【ブログ下書き YYYY-MM-DD】クラブオーサム(Club Awesome!) 3本分",
  "body": "（メール本文。templates/email-template.md に沿って組み立てた3本分の下書きをまとめたテキスト）"
}
```

- フィールド:
  - `token` — 認証トークン。**リポジトリには絶対に書かない**（§7 参照）
  - `mode` — `"TEST"` または `"PROD"` のいずれか（`config/run-mode.md` の値と一致させる）
  - `subject` — メール件名。TEST モードでは冒頭に `【TEST】` を必ず付ける
  - `body` — メール本文（プレーンテキスト）。3本分のブログ下書きを1通にまとめたもの

参考: `templates/webhook-payload-example.json` にダミー値入りのサンプルあり。

### 6.3 宛先について

- 宛先（To）は **Apps Script 側で `mode` をもとに決定**する設計。
- Claude（クライアント）は宛先を JSON に乗せない（or 乗せても Apps Script 側が無視・上書きする）。
- これにより「TEST モードなのに顧客アドレスへ送ってしまう」事故を二重で防ぐ。
- 詳細は `config/recipients.md` を参照。

---

## 7. WEBHOOK_TOKEN の取り扱い（機密情報）

- **WEBHOOK_TOKEN はリポジトリに絶対にコミットしない。**
  以下のいずれにも書かないこと:
  - `*.md`（CLAUDE.md / README.md / config/*.md / data/*.md / templates/*.md）
  - `*.json`（templates/webhook-payload-example.json も含め、ダミー値のみ）
  - コミットメッセージ・PR本文・Issue本文
- **トークンは Routine 実行時に外部から渡す**前提:
  - Claude Code Routines のプロンプト内に環境変数相当の入力として埋め込むか、
  - 実行環境の環境変数（例: `WEBHOOK_TOKEN`）として注入し、Routine 実行時に Claude が参照する。
- Claude は POST 直前に `<<WEBHOOK_TOKEN>>` プレースホルダをトークンの実値で置換し、
  **置換後の値はログ・ファイル・後続のメッセージに残さない**こと。
- もしトークンが GitHub 等に誤って push された場合は、即座に Apps Script 側でトークンをローテーションし、
  履歴からの除去を行う。

---

## 8. コンテンツ生成上のルール

- **薬機法・景品表示法に配慮**し、断定的な効果表現を避けること。
  - NG例: 「酵素浴で病気が治ります」「絶対に痩せます」「自律神経が整います」
  - OK例: 「あたたかくお過ごしいただけます」「すっきりとした時間として」
  - 詳細は `config/tone.md` の「避けるべき表現」を参照。
- **メール本文には絵文字を使わないこと（プレーンテキスト送信のため文字化けする）。**
  - メール本文（Webhook ペイロードの `body`）は Apps Script の `MailApp.sendEmail` に
    プレーンテキストで渡されるため、Gmail 等の受信側で絵文字が文字化けすることがある。
  - NG: ✨ 🌿 🌸 ☆ 🌟 ♪ ❤ 💕 😊 など Unicode 絵文字全般（本文・CTA文に含めない）。
  - OK: `subject` 内の `【TEST】` `【ブログ下書き YYYY-MM-DD】` などの括弧記号。
  - OK: `body` 内の `────`（罫線）・`■` `▼` `／` など日本語の標準記号。
  - OK: ハッシュタグ自体（絵文字を含まない `#xxx` の形）。
  - 季節感や親しみは語彙と文章のリズムで表現する（例: 「ぽかぽか」「ふんわり」「やわらかな」）。
- **画像は扱わない。** テキストのみ生成する。画像URLや画像の埋め込み指示も出さない。
- **メールは必ず1通にまとめる。** 3本を別々のメールで送らない（= Webhook も 1リクエストのみ）。
- 各記事は `templates/post-template.md` の項目をすべて埋める。
- 文字数目安は本文 600〜1000文字（1記事あたり）。

---

## 9. ファイル構成

```
hotpepper-blog-draft-slnH000627543/
├── CLAUDE.md                          ← 本ファイル（運用ルール）
├── README.md                          ← リポジトリ概要・初回テスト手順
├── config/
│   ├── salon.md                       ← サロン基本情報
│   ├── tone.md                        ← 文体ルール／NG表現
│   ├── recipients.md                  ← 送付先（TEST / PROD）と Webhook 宛先制御メモ
│   └── run-mode.md                    ← 実行モード（TEST / PROD）
├── templates/
│   ├── post-template.md               ← ブログ1本分のテンプレ
│   ├── email-template.md              ← メール件名・本文のテンプレ（Webhook の subject/body）
│   └── webhook-payload-example.json   ← Webhook POST ペイロードのサンプル（token はダミー）
└── data/
    ├── menu.md                        ← メニュー一覧
    ├── staff.md                       ← スタッフ情報
    ├── campaigns.md                   ← キャンペーン情報
    ├── past-topics.md                 ← PROD用の重複判定履歴
    └── test-topics.md                 ← TEST用の生成履歴
```

---

## 10. トラブル時の方針

- 必須ファイルが欠けている、または `run-mode.md` の値が `TEST`/`PROD` 以外の場合は、
  Webhook へ POST せず処理を中止する。
- WEBHOOK_TOKEN が未設定（プロンプト・環境変数のいずれにもない）場合も、POST せず処理を中止。
- Webhook が失敗（4xx/5xx, タイムアウト, `{"ok": false}` 等）した場合:
  - 自動再試行は **1回まで**（過剰再送による多重送信を防ぐ）。
  - 再試行も失敗した場合は、生成テーマを `data/test-topics.md` / `data/past-topics.md` に
    **追記しない**（送れていないものを履歴に積まない）。
  - 失敗内容は Routine の実行ログに記録するに留め、リポジトリには書き込まない。
- 不明点があれば送信せず、Routine の実行ログに「要確認」のメモを残すに留めること。
