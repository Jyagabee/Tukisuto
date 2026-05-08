---
title: "GrowiのPluginを作った話"
date: 2026-05-08T11:27:03+09:00
draft: false
featured: false
showHero: true  # ヒーロー画像を表示するか
heroStyle: "basic"  # ヒーロー画像のスタイル（basic, background, thumbなど）
heroImage: "hero.jpg"  # 記事のメイン画像
thumbnail: "thumbnail.jpg"  # 一覧表示用画像
categories: []
tags: []
summary: ""
---
# GROWI page age warning plugin notes
ツレに、`growiにもqiitaみたいな最終更新日時による警告実装できないかな`って言われたんで gpt-5.5 high にシコシコ書かせました。  

とりま以下に作らせた生成物が入ってるリポジトリ貼っておきます  
https://github.com/Miyabi0619/growi-plugin-page-age-warning  

何かあったら、適当に異臭なりなんなり投げてください。  
chatgptはclaudeより使用量に余裕があっていいねと言いたかったけどclaudeの容量2倍になっててアレ

## 概要

GROWI に、作成から一定期間が経過したページへ注意喚起を表示する script plugin を追加した。
Qiita の古い記事警告のように、ページ本文の冒頭へ「作成から1年以上経過しています」などのメッセージを出す。

対象リポジトリ:

- `growi-plugin-page-age-warning`

## 目的

古いページを参照した利用者が、内容の鮮度に注意できるようにする。
特にデータ移行で作成日が古いページや、運用ルール・手順が古くなっているページを見落としにくくする。

## 現在の仕様

- 判定基準は `createdAt`
- `createdAt` から365日以上で警告表示
- `createdAt` から730日以上で、より強い文言の警告表示
- 365日未満のページでは何も表示しない
- 警告は記事本文コンテナの先頭へ挿入する
- GROWI のページタイトルや右側のタグ/見出しまとめ領域はなるべく押し下げない
- `/` と `/Sidebar` では表示しない
- `_Template` / `__Template` 系ページでは表示しない
- `/admin`, `/search`, `/tags`, `/_api` などの非ページ系ルートでは表示しない

## 表示文言

1年以上:

```text
作成から1年以上経過しています
作成日: YYYY年M月D日。内容が現在の運用と異なる可能性があります。
```

2年以上:

```text
作成から2年以上経過しています
作成日: YYYY年M月D日。内容が古い可能性が高いため、参照時は注意してください。
```

## 実装メモ

エントリポイントは `client-entry.tsx`。

GROWI の script plugin 仕様に合わせて、`window.pluginActivators['growi-plugin-page-age-warning']` に `activate` / `deactivate` を登録している。

日付取得は次の順番で行う。

1. GROWI REST API v3 `/_api/v3/page?path=...`
2. API で取得できない場合、画面 DOM 上の `作成日 YYYY/MM/DD HH:mm` を fallback として読む

DOM 挿入先は、`.markdown-body`, `.markdown-preview`, `.wiki` などの本文コンテナ候補から選ぶ。
`$lsx(./)` の展開結果のような下部の大きなブロックを誤って選ばないよう、面積や文字量ではなく、画面上で上にある本文候補を優先している。

## デバッグ方法

ブラウザのコンソールでデバッグログを有効化する。

```js
localStorage.setItem('growi-plugin-page-age-warning:debug', 'true')
location.reload()
```

プラグインが読み込まれているか確認する。

```js
window.pluginActivators?.['growi-plugin-page-age-warning']
```

期待値は `activate` と `deactivate` を持つ object。

よく見るログ:

- `insert target was not found`: 挿入先の本文コンテナが見つからない
- `could not find createdAt`: API/DOM のどちらからも作成日が取れない
- `message hidden because page age is under threshold`: 作成から365日未満なので非表示
- `date resolved`: 日付取得に成功

## ビルド

```sh
pnpm install
pnpm build
```

GROWI 7.3 系で読ませるため、Vite manifest は `dist/manifest.json` に出力している。

## 運用メモ
やー、ちょっと手こずりましたね。まあワイは何もしてないわけですが  
web関連とgrowiの知識なかったのがアカンね、デバッグしんどかった。  
やっぱwebようわからんわ。あんまり好かんです。

某人に見られたら鼻で笑われるんだろうな、と思ったり思わなかったり。  
とはいえ、こんなプラグインくらいAI使えばええやんと思ったり思わなかったり。

まあ、テスト前の時間と次の授業半分くらいでこれくらい動くものができるのはやっぱりすごいね 

終わり。