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
6. `templates/email-template.md` に従い **1通のメール** にまとめる。
7. 確認したモードに応じた宛先に送付する。
8. 生成したテーマ（タイトル・要旨）を、モードに応じたファイルへ追記する。
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
  送付先は `config/recipients.md` の TEST 宛先（`tomo0014jp@gmail.com`）のみ。
- **メール件名の冒頭に `【TEST】` を付ける。**
  例: `【TEST】HotPepperブログ下書き 3本（YYYY-MM-DD）`
- **生成したテーマは `data/test-topics.md` のみに記録する。**
  `data/past-topics.md` には絶対に追記しない。
- TEST モードの生成物は本番の重複判定対象に含めない。

---

## 5. PROD モードのルール

- 送付先は `config/recipients.md` の PROD 宛先のみ。
- 生成したテーマは `data/past-topics.md` に追記する。
- 件名に `【TEST】` は付けない。

---

## 6. コンテンツ生成上のルール

- **薬機法・景品表示法に配慮**し、断定的な効果表現を避けること。
  - NG例: 「シミが消えます」「絶対に痩せます」「100%効果があります」
  - OK例: 「乾燥が気になる季節におすすめのケアです」「サラッとした仕上がりを目指せます」
  - 詳細は `config/tone.md` の「避けるべき表現」を参照。
- **画像は扱わない。** テキストのみ生成する。画像URLや画像の埋め込み指示も出さない。
- **メールは必ず1通にまとめる。** 3本を別々のメールで送らない。
- 各記事は `templates/post-template.md` の項目をすべて埋める。
- 文字数目安は本文 600〜1000文字（1記事あたり）。

---

## 7. ファイル構成

```
hotpepper-blog-draft-slnH000627543/
├── CLAUDE.md                  ← 本ファイル（運用ルール）
├── README.md                  ← リポジトリ概要
├── config/
│   ├── salon.md               ← サロン基本情報
│   ├── tone.md                ← 文体ルール／NG表現
│   ├── recipients.md          ← 送付先（TEST / PROD）
│   └── run-mode.md            ← 実行モード（TEST / PROD）
├── templates/
│   ├── post-template.md       ← ブログ1本分のテンプレ
│   └── email-template.md      ← メール本文のテンプレ
└── data/
    ├── menu.md                ← メニュー一覧
    ├── staff.md               ← スタッフ情報
    ├── campaigns.md           ← キャンペーン情報
    ├── past-topics.md         ← PROD用の重複判定履歴
    └── test-topics.md         ← TEST用の生成履歴
```

---

## 8. トラブル時の方針

- 必須ファイルが欠けている、または `run-mode.md` の値が `TEST`/`PROD` 以外の場合は、
  メール送信を中止し、エラー内容を TEST 宛先（`tomo0014jp@gmail.com`）に通知する。
- 不明点があれば送信せず、TEST 宛先に「要確認」のメモを送るに留めること。
