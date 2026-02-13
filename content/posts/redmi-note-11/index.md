---
title: "Redmi Note 11について"
date: 2026-02-13T15:50:00+09:00
draft: false
featured: false
showHero: false
categories: ["Android", "スマートフォン"]
tags: ["Redmi Note 11", "カスタムROM", "crDroid", "Bootloader Unlock", "LineageOS", "Xiaomi", "MIUI", "HyperOS"]
summary: "Redmi Note 11にカスタムROM（crDroid）を焼いた話。MIUIの問題点、Bootloader Unlockの手順、カスタムROMの選び方と注意点について解説。"
---
# Redmi Note 11について

> Redmi Note 11は、2022年3月に発売されたXiaomiの低価格（神コスパ）な4Gスマートフォン。約2.5万円（当時）という安さながら、90Hz駆動の6.43インチ有機ELディスプレイ、5000万画素の4眼カメラ、33W急速充電、大容量5000mAhバッテリーを搭載し、日常利用に十分な機能を持っています。

by Gemini

個人的にMIUI及びHyperOSが非常に嫌いなのでOSを書き換えました。
HyperOSを4年前のミドルレンジで動かすのは動作重くて無理がある流石に。OSアプデも降ってこないし。

というのは本音で建前はwifi接続問題、bluetooth接続問題が頻発していたためOSを書き換えます。
MIUI,HyperOSはご存知の通りクセ強OSなので、タスクキルするついでにwifiのDDS関連の転送処理いらんやろwって送信しなくなるっぽい。Pixelちゃんたちは何ら問題なさそうなのでAOSPなOSにします。
けどワイのeuROM入れた12Turboちゃんも通信関連問題ないんですよね。何故コイツラだけ異様に調子悪いのか分からない。やっぱxiaomi.eu romにせんといけんかなあ。。。だからグロ版嫌いなんすよ

# Bootloader Unlockについて

以下のサイトを参考に行った
[グローバル・日本版HyperOSのBootloaderアンロックのやり方。実名認証、レベル上げ不要！！](https://mitanyan98.hatenablog.com/entry/2024/02/01/011358)

[PCを使ってMi communityのUnlockBootloader権限の申請をする方法](https://mitanyan98.hatenablog.com/entry/2026/02/01/015610)
↑これのおかげでBLU取れました。ぽみゃーら感謝せえよ

[Xiaomi系スマートフォンのブートローダーアンロック方法【Xiaomi/Redmi/POCO】](https://mitanyan98.hatenablog.com/entry/2020/06/25/190552#%E5%88%A5%E3%83%84%E3%83%BC%E3%83%AB%E3%81%AE%E6%96%B9%E6%B3%95%E5%AE%9F%E3%81%AF%E3%81%93%E3%81%A3%E3%81%A1%E3%81%AE%E3%81%BB%E3%81%86%E3%81%8C%E3%81%8A%E3%81%99%E3%81%99%E3%82%81)

これ見て理解できたらある程度xiaomi系統のスマホの扱い方が分かると思います。これ見ても分からないんだったら、ビビり散らかしながら実機をBLUしてください。
手ｪ動かさんと知識付きませんわ

## 追記: うんちカスMIUI,HyperOSのBLU制限について

MIUIとHyperOSはうんちカスなので、以下の制限を設けています。以下の制限は本国内のサーバーで管理されており、上記のBLU申請が取れていても、制限に引っかかっていたら容赦なくブロックされます。

- MIUIの頃の端末なら1ヶ月空けば最大年4台まで可
- HyperOSプリインストールの端末は年1台まで
- MIUIの頃の端末でもHyperOS 2まで上げるとMIUIに偽装する方法が使えなくなるので、年1台までの法則が適用される

今回、RedmiNote11は奇跡的にプリインストールOSがMIUIですが、HyperOS2まで上げてしまったので1年空くまでBLUできません。ほんまカス屋根。だからヤなんだよこのOS。Pixel見習え
HyperOS2.0まで上げてしまっても、BLU申請する垢変えたらもう一台BLUできます。よかったね。
マジでさあ、わけわからん独自ルール作るのやめてくれ誰が嬉しいんこんなの。

# 入れたOS

個人的にAOSPなUIが好みのため、LineageフォークのCrDroid v12(A16)を焼き焼きしました。

こいつはほぼAOSPな挙動するのに動作が軽く独自機能が付いてて良き。
頻発していたwifi接続問題、bluetooth接続問題はほぼこいつで解消されるでしょう知らないけど。

![Screenshot_20260212-095714_設定.png](/attachment/698d25a54f3876aa5e978399)
う～んシンプルで良い。

![Screenshot_20260212-100012_設定.png](/attachment/698d26414f3876aa5e978916)
↑これが前言ってた独自設定ね。いや～～～～～痒い所に手が届きすぎて感動ですわ。なんもせんとintegrityをstrongまで通してくれるんですか?!って感じ。ガチありがたい。ホンマぽみゃーらメンテナーの方々に感謝せえよ。crDroidしゅきしゅきだいしゅき～～～～

ワイがPixel5に入れてたPixelBuild(LineageフォークA14)の調子はすこぶる良いので、多分動くと思っております。動かんかったら許してちょ

あ、今回はcrDroidの公式recoveryを焼いております。TWRPが好きな人は申し訳ないけど、まあ文字読めば使い方分かると思います。crDroidの公式recoveryって言っても、lineageOS19以降のリカバリーなんで、見たことある人は多いと思います。メンテアプデ諸々するときは頑張って

あと、カスロムの特徴として基本バージョンまたいだOSアプデはできません。要するに、今回入れたcrDroidのv12は基本そのバージョンで死ぬまで使うことになります。メンテナーおらんなったらセキュリティ下がるってのはそういう意味ね。できなくもないらしいけど、多分バージョンまたいでも良いようにlineageもcrDroidも作ってないと思うので、大人しくアプデしたかったら初期化してください。swiftbackupとか使えばそのままOSの環境引っ越せるんで、それで対処してください。

# 入れたGapps

> LineageOS、crDroid、BlissRoms などのカスタムAndroid ROMを使用している場合、Playストア、Gmail、マップなどの重要なGoogleサービスが欠けていることに気付くかもしれません。
>
> MindTheGappsは、カスタムROMユーザー向けに最小限かつ信頼性の高いGoogle Apps(GApps)パッケージを提供しており、不要な負荷をかけずに必要なGoogleサービスをシームレスに統合できます。

by MindGapps

まあ要するに、GooglePlayとか開発者サービスとかGmailとかgoogleとかyoutubeとかが一つになったやつです。lineageやらcrdroidやらは、そういったGoogle関連のアプリが入ってないので、後からアプリセットを焼く必要があります。

ワイは毎回インスコするOSが推奨しているやつでええかの民なので、今回はMindGappsを焼き焼きしました。これねー、OSのバージョンと合ったやつ焼かないと初期設定が不安定になったり、最初の起動なのにplayストアが逝かれることがあるので注意(2敗)
まあ[ここらへん](https://note.com/reindex/n/ncc1e3f465f81)読んどけば多少知識付くと思います。個人的にNikGappsのcoreかomuniが好きです。~google関連のアプリは入れすぎたら重いだけ~

なんでGoogleのアプリ入れて個人情報を差し出さないとアカンねん！！！って人向け(GrapheneOS)のGappsとかあるので、見たかったらググったらよいと思われます(データを紐づけずに云云かんぬんする)。めんどいので解説しません

# 注意点

- BLUすると[ゴミカスアホボケ詩ねクソPlay Integrity API](https://developer.android.com/google/play/integrity/overview?hl=ja)に通らない
- Integrity APIに通らないので銀行系、マイナンバー系のアプリが動かない
- BLUするとセキュリティが下がってるので乗っ取られたら終わる
- OSメンテナーが消えたらOSアプデが来ない(ぽみゃーらがやるんだよ独自OSつくれ)
- 電波法云々にタッチしちゃうかもしれない ~一般人が見て分からないのでほぼ無視できるリスク~
- `<span style="color:#F00;">`絶対OEMをロックするな！入れたOSによっては文鎮化する
- アプデがめんどくさい(設定からアプデしても失敗することがある)
- adbの使い方を覚える必要がある(`adb devices`, `adb reboot bootloader`, `adb sideload crdroid~xx.zip`あたり覚えときゃええと思いますよ)
- OS新規インストール時にgappsの焼くタイミングミスるとダルい(ミスって覚えろ)

こんなもんでしょうか。まあめんどくさかったらeu.rom焼いてもいいですよ。重いと思うけど。今見たらeu.romのHyperOS2.0があるんやね。3.0行かないやろなあ…

追記: もしOSを上書きしたい場合、[XDA Forum: Redmi Note 11 (spes/spesn)](https://xdaforums.com/f/redmi-note-11-spes-spesn.12617/?prefix_id=33)か[eu.rom](https://sourceforge.net/projects/xiaomi-eu-multilang-miui-roms/files/xiaomi.eu/HyperOS-STABLE-RELEASES/)あたりを見て、良さそうな奴をDLして焼き焼きしてください。この機種は何故かTelegramのチャンネル見つからんかったので...。Telegramのチャンネル見つけたら、そっちからインスコする方が良いと思います。最悪謎の外国人兄貴たちに対処法聞けるし、どこの馬の骨が作ったか分からない超絶謎OSが焼けます。~そっちの方がおもろい~
![スクリーンショット 2026-02-12 095148.png](/attachment/698d24314f3876aa5e9781c6)
↑コレ全部(?)謎OS。大体Lineageかeuのフォークで、変な機能(なんちゃって顔認証とか)が謎技術で移植されてる

Pixelちゃんよかインストール時に気にかけないことだったり、入れた後のメンテナンスめんどくさいけど、BLUしたからまあなんとかなるでしょ。最悪初期化したらなんとかなります。ヤバかったらワイに連絡くれれば対応します。edl関連と文鎮化は難しいかも

crDroidの素晴らしさをみんなで感じましょうや...
