---
title: "Android Studio Meerkat | 2024.3.1 を日本語化した話"
date: 2025-03-17T10:34:08+09:00
draft: false
featured: false
showHero: false  # ヒーロー画像を表示するか
heroStyle: "basic"  # ヒーロー画像のスタイル（basic, background, thumbなど）
categories: []
tags: []
summary: ""
---
## はじめに

皆さんAndroidStudioをmeerkatへアップデートしてらっしゃるでしょうか。ワイはノリと勢いで更新してしまいました。
アプデしてから日本語化できないことに気づき、1週間くらい我慢してたけど耐えられなくなったので日本語化しました。
備忘録的なアレを残しておきます

### プラグインあるんじゃないの

ってことですよね。ﾊｲ。laybugはあるけどmeerkatは無いです。日本語化プラグインがバージョン不一致でエラー出て使えんなりました。  
[このプラグイン](https://plugins.jetbrains.com/plugin/13964-japanese-language-pack------/versions)は更新されるかどうか分からんらしいです。どうやらInteliJ IDEに日本語が内蔵されてしまった関係でメンテナンスされないらしい。~~控えめに言ってクソ~~

## 日本語化する方法
ということで解説していきます。
1. InteliJ IDE Comunity Editionをインストールする
2. pluginフォルダから日本語化プラグインを頂く
3. Android Studioのpluginフォルダへコピー
4. Appearance & Behavior>System Settings>Language and RegionのLanguageを日本語に変更

こんだけです。
以下に注意点だけまとめておきます。

### 注意点
InteliJ IDE のpluginフォルダパス:C:\Users\ameri\AppData\Local\Programs\IntelliJ IDEA Community Edition\plugins  
この中にlocalization-jaがあるのでコピー  

Android Studioのpluginフォルダパス:C:\Users\Users\AppData\Local\Programs\Android Studio\plugins
先ほどコピーしてきたディレクトリごとコピペ

linuxは知らない。いずれはやらんとあかんねえ…

## おわりに
以上です。備忘録なので信頼性等々求めてる人はよそ様の記事なりchatgptなりに聞いてください。  
日本語化する気あるのかないのか分からんの何とかしちくり～～～～

おわり。