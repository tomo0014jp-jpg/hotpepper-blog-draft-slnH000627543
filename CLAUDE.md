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
- メール送信方式: **Gmail コネクター（MCP）の `send_email` ツールを直接呼び出す**
  （以前は Apps Script Webhook 方式 → 廃止）

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
7. **Gmail コネクターの `send_email` ツールでメールを送信する**（詳細は §6）。
   - 必要に応じて `create_draft`（下書きとして保存）を使ってもよい。送信運用に切り替える前の動作確認等に。
8. ツールの応答を確認する。
   - 成功（メッセージID 返却 / エラーなし）なら処理を続行。
   - 失敗（権限エラー、ネットワークエラー、コネクター未接続等）の場合は §9 の手順で扱う。
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
  Gmail コネクター呼び出し時の `to` フィールドは `config/recipients.md` の TEST 宛先
  （`tomo0014jp@gmail.com`）のみ。
- ⚠️ **Gmail コネクター方式では、宛先制御の最後の砦は Claude（このルール）だけです。**
  以前の Apps Script Webhook 方式では Apps Script 側でも `mode` チェックで二重ガードできたが、
  コネクター直叩きにしたことでサーバー側のガードはなくなった。Claude は CLAUDE.md と
  `config/recipients.md` を必ず確認し、TEST 宛先以外の `to` を絶対に組み立てないこと。
- **メール件名（subject）の冒頭に `【TEST】` を付ける。**
  例: `【TEST】【ブログ下書き YYYY-MM-DD】クラブオーサム(Club Awesome!) 3本分`
- **メール本文（body）の末尾に「※本メールはTEST送信です。お客様には送信されていません。」を必ず入れる。**
- **生成したテーマは `data/test-topics.md` のみに記録する。**
  `data/past-topics.md` には絶対に追記しない。
- TEST モードの生成物は本番の重複判定対象に含めない。

---

## 5. PROD モードのルール

- 送付先は `config/recipients.md` の PROD 宛先のみ。
- 件名に `【TEST】` は付けない。
- 生成したテーマは `data/past-topics.md` に追記する。
- PROD 切替前に `config/recipients.md` の PROD 宛先が正しい本番アドレスになっていることを必ず確認。

---

## 6. メール送信方式（Gmail コネクター / MCP）

### 6.1 使用するツール

Claude Code Routines に Gmail コネクター（MCP）を接続したうえで、以下のいずれかのツールを使う:

- **`send_email`** — メールを直接送信する（推奨 / 自動運用向け）
- **`create_draft`** — Gmail の下書きを作成する（送信前確認したいとき向け）

### 6.2 ツール呼び出しのパラメータ（最小セット）

```
to:      "tomo0014jp@gmail.com"   ← TEST モードの宛先（recipients.md から決定）
subject: "【TEST】【ブログ下書き YYYY-MM-DD】クラブオーサム(Club Awesome!) 3本分"
body:    （templates/email-template.md に沿って組み立てた、3本分の下書き本文）
```

- `to` の値は **モード（TEST/PROD）に応じて `config/recipients.md` から決定する**。
  ハードコードしない（コネクター呼び出し直前にファイルを参照する）。
- `cc` / `bcc` は使わない。
- 送信形式は **プレーンテキスト**（HTML メールにしない）。
- Gmail コネクターが追加のフィールド（例: `threadId` 等）を要求する場合は、それも適切に埋める。

### 6.3 認証・接続

- Gmail コネクターの OAuth 認可は、Routine の所有者アカウント（大石氏）の Gmail 権限で行われる。
- コネクター UUID・URL は claude.ai 側の設定（[Connectors](https://claude.ai/customize/connectors)）に紐づく。
- コネクターが Routine に未接続の状態で実行された場合、`send_email` ツールが利用できないため
  §9 のトラブル時手順に従って中止する。

---

## 7. （旧）Apps Script Webhook 方式について

**廃止されました。** WEBHOOK_TOKEN・Webhook URL の取り扱いに関する旧ルールは適用されません。
リポジトリ内のいずれのファイル（`*.md` `*.json` 等）にも、Webhook URL・トークンの実値を残さないこと。

---

## 8. コンテンツ生成上のルール

- **薬機法・景品表示法に配慮**し、断定的な効果表現を避けること。
  - NG例: 「酵素浴で病気が治ります」「絶対に痩せます」「自律神経が整います」
  - OK例: 「あたたかくお過ごしいただけます」「すっきりとした時間として」
  - 詳細は `config/tone.md` の「避けるべき表現」を参照。
- **メール本文には絵文字を使わないこと（プレーンテキスト送信のため文字化けする）。**
  - メール本文（`send_email` の `body`）は Gmail にプレーンテキストで送られるため、
    Unicode 絵文字が文字化けすることがある。
  - NG: ✨ 🌿 🌸 ☆ 🌟 ♪ ❤ 💕 😊 など Unicode 絵文字全般（本文・CTA文に含めない）。
  - OK: `subject` 内の `【TEST】` `【ブログ下書き YYYY-MM-DD】` などの括弧記号。
  - OK: `body` 内の `────`（罫線）・`■` `▼` `／` など日本語の標準記号。
  - OK: ハッシュタグ自体（絵文字を含まない `#xxx` の形）。
  - 季節感や親しみは語彙と文章のリズムで表現する（例: 「ぽかぽか」「ふんわり」「やわらかな」）。
- **画像は扱わない。** テキストのみ生成する。画像URLや画像の埋め込み指示も出さない。
- **メールは必ず1通にまとめる。** 3本を別々のメールで送らない（= `send_email` も 1回のみ）。
- 各記事は `templates/post-template.md` の項目をすべて埋める。
- 文字数目安は本文 600〜1000文字（1記事あたり）。

---

## 9. ファイル構成

```
hotpepper-blog-draft-slnH000627543/
├── CLAUDE.md                  ← 本ファイル（運用ルール）
├── README.md                  ← リポジトリ概要・初回テスト手順
├── config/
│   ├── salon.md               ← サロン基本情報
│   ├── tone.md                ← 文体ルール／NG表現
│   ├── recipients.md          ← 送付先（TEST / PROD）
│   └── run-mode.md            ← 実行モード（TEST / PROD）
├── templates/
│   ├── post-template.md       ← ブログ1本分のテンプレ
│   └── email-template.md      ← メール件名・本文のテンプレ（Gmail send_email の subject/body）
└── data/
    ├── menu.md                ← メニュー一覧
    ├── staff.md               ← スタッフ情報
    ├── campaigns.md           ← キャンペーン情報
    ├── past-topics.md         ← PROD用の重複判定履歴
    └── test-topics.md         ← TEST用の生成履歴
```

---

## 10. トラブル時の方針

- 必須ファイルが欠けている、または `run-mode.md` の値が `TEST`/`PROD` 以外の場合は、
  メール送信を行わず処理を中止する。
- Gmail コネクターが Routine に未接続、または `send_email` 呼び出しが権限エラー等で
  失敗した場合は、自動再試行は **1回まで**（過剰再送による多重送信を防ぐ）。
- 再試行も失敗した場合は、生成テーマを `data/test-topics.md` / `data/past-topics.md` に
  **追記しない**（送れていないものを履歴に積まない）。
- 失敗内容は Routine の実行ログに記録するに留め、リポジトリには書き込まない。
- 不明点があれば送信せず、Routine の実行ログに「要確認」のメモを残すに留めること。
