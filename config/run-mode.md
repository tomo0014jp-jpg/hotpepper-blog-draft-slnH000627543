# run-mode.md — 実行モード設定

## 現在のモード

```
TEST
```

> ⚠️ 上記の1行（`TEST` または `PROD`）が、Routine 実行時に Claude が参照する**唯一の値**です。
> コードブロック内の文字列を書き換えてください。
> **初期値は必ず `TEST` です。**

---

## モード一覧

### TEST（テスト運用 / 初期値）

- 送付先: `config/recipients.md` の **TEST 宛先のみ**（`tomo0014jp@gmail.com`）
- 件名冒頭: **`【TEST】`** を必ず付ける
- 生成テーマの記録先: `data/test-topics.md`
- 重複判定: `data/test-topics.md` を参照
- **顧客（サロン）には絶対に送らない**

### PROD（本番運用）

- 送付先: `config/recipients.md` の **PROD 宛先のみ**
- 件名冒頭: `【TEST】` は付けない
- 生成テーマの記録先: `data/past-topics.md`
- 重複判定: `data/past-topics.md` を参照

---

## 切り替え方法

1. このファイル冒頭のコードブロック内の値を `TEST` または `PROD` に書き換える。
2. 変更を保存してコミットする。
3. 次回 Routine 実行時から新しいモードで動作する。

### TEST → PROD への切り替え時の注意

- `config/recipients.md` の PROD 宛先が**正しい本番の送付先**になっていることを必ず確認すること。
- `config/salon.md` `data/menu.md` `data/staff.md` `data/campaigns.md` の TODO がすべて埋まっていることを確認すること。
- 切り替えはユーザー（大石氏）の明示的な指示があってから行うこと。Claude が独断で切り替えてはいけない。

### PROD → TEST への戻し方

- 同じくこのファイルの値を `TEST` に書き換えるだけ。
- TEST モードに戻したあとに生成したテーマは `data/test-topics.md` にのみ記録されます。
