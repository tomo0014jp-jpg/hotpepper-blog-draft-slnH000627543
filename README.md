# hotpepper-blog-draft-slnH000627543

HotPepper Beauty 店舗ブログ向けに、毎朝3本分の下書きを生成し、メールで送付する Claude Code Routines 用リポジトリです。

## 概要

- **対象媒体**: HotPepper Beauty 店舗ブログ
- **頻度**: 毎朝1回 / 1日3本 / メール1通
- **出力**: テキストのみ（画像なし）

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
│   └── email-template.md      メール本文のテンプレ
└── data/
    ├── menu.md                メニュー一覧（TODO）
    ├── staff.md               スタッフ情報（TODO）
    ├── campaigns.md           キャンペーン情報（TODO）
    ├── past-topics.md         PROD 用の重複判定履歴
    └── test-topics.md         TEST 用の生成履歴
```

## モード切替

`config/run-mode.md` の値を `TEST` または `PROD` に書き換えることで切り替えます。

詳細は [CLAUDE.md](./CLAUDE.md) を参照してください。

## 本番（PROD）切替前のチェックリスト

- [ ] `config/recipients.md` の PROD 宛先を実際の送付先に更新
- [ ] `config/salon.md` のサロン情報を埋める
- [ ] `data/menu.md` のメニュー情報を埋める
- [ ] `data/staff.md` のスタッフ情報を埋める
- [ ] `data/campaigns.md` のキャンペーン情報を埋める
- [ ] TEST モードで数日運用し、内容を確認
- [ ] `config/run-mode.md` を `PROD` に変更
