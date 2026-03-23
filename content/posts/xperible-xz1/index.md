---
title: "Xperableを使ってXZ1をBLUする"
date: 2026-03-23T20:02:19+09:00
draft: false
featured: false
showHero: true
heroStyle: "basic"
heroImage: "hero.jpg"
thumbnail: "thumbnail.jpg"
categories: []
tags: ["xperia", "root", "BLU"]
summary: "Sony Xperia XZ1 (SO-01K) のブートローダーをXperibleを使ってアンロックする手順まとめ。"
---
どうも、じゃがびぃさんです。  
どうやら、xperibleなるツールがあるというお話があって、BLU取るのがｶﾅｰﾘ大変だったのでメモ取っておきます。

このXZ1ちゃんは、ワイが中学生の頃に自分の金で買った中古の個体でした。カメラボタンが陥没してたので4000円くらいで買えたやつ。
OS古いし、qunlocktools買ってBLUしよーとか思ってたらいつの間にか鯖サ終してて放置してました。  
いや～～～～待ってみるもんですね。

ということで方法は以下に書きます。

## 環境

- 端末：Sony Xperia XZ1 SO-01K（ファームウェア 47.1.F.1.105）
- ファームウェア：Android Oreo であること
- PC：Windows + adb/fastboot 環境
- 使用ツール：bindershell-v2、xperable-1.1、Zadig

## Step 0: Android 8.0へダウングレード

でですね、今回の端末はAndroid 9.0 Pieなんですね。まずダウングレードする必要があります。

以下のURLから SO-01K_47.1.F.1.105_1311-7618_R18B.ftf をダウンロードしました。   
https://archive.org/download/xperia-firmware/国内版Xperia/Xperia%20XZ1/

update.xmlは無かったので、以下を生成して使いました。
```
<?xml version="1.0" encoding="utf-8"?>
<UPDATE>
  <SYSTEMPARTITIONIMAGE>partition.zip</SYSTEMPARTITIONIMAGE>
  <BOOT>boot.zip</BOOT>
  <NOERASE>appslog_X-FLASH-ALL-99C3.sin</NOERASE>
  <NOERASE>diag_X-FLASH-ALL-99C3.sin</NOERASE>
  <NOERASE>ssd_X-FLASH-ALL-99C3.sin</NOERASE>
  <NOERASE>Qnovo_X-FLASH-ALL-99C3.sin</NOERASE>
  <FACTORY_ONLY>persist_X-FLASH-ALL-99C3.sin</FACTORY_ONLY>
  <NOERASE>userdata_X-FLASH-CUST-99C3.sin</NOERASE>
  <NOERASE>cust-reset.ta</NOERASE>
  <NOERASE>master-reset.ta</NOERASE>
  <RESETNONSECUREADB>reset-non-secure-adb.ta</RESETNONSECUREADB>
  <NOERASE>reset-wipe-reason.ta</NOERASE>
  <SIMLOCK>simlock.ta</SIMLOCK>
</UPDATE>
```
で、ftfファイルを7-zipで先に展開します。  
その中に、newflasher.exe一式をコピペしてnewflasher.exe を実行します。  
以下の通りの返答をすると、flashが始まります
```

Optional step! Type 'y' and press ENTER if you need GordonGate flash driver, or type 'n' to skip.
This creates GordonGate driver installer in the same dir with newflasher.exe!
n
Device path: \\?\usb#vid_0fce&pid_b00b#5&3a22045f&0&1#{a5dcbf10-6530-11d2-901f-00c04fb951ed}
Class Description: SOMC Flash Device
Device Instance Id: USB\VID_0FCE&PID_B00B\5&3A22045F&0&1


Do you want to keep userdata? Type 'y' and press ENTER to confirm, or type 'n' to erase userdata.
n

Reboot mode at the end of flashing:
  type 'a' for reboot to android, type 'f' for reboot to fastboot, type 's' for reboot to same mode, type 'p' for poweroff, and press ENTER.
a

Optional step! Type 'y' and press ENTER if you want dump trim area, or type 'n' and press ENTER to skip.
Do in mind this doesn't dump drm key since sake authentifiction is need for that! But it is recommend to have dump in case hard brick!
n
```
これで、android 8.0になりました。  
注意ですが、androidは初期化されます。また、オフラインでセットアップを完了してください。ネットに繋げると、googleアシスタントに「この端末は管理されています」って言われて進めません(1敗)。

## Step 1: 一時root取得（bindershell）

bindershell-v2 をダウンロードして PC に展開したら、バイナリをデバイスに転送します。  
bindershell-v2は、普通にxda forumに落ちてる奴でOKです。

```powershell
adb push bindershell /data/local/tmp
adb shell
```

シェルに入ったら：

```bash
cd /data/local/tmp
chmod +x bindershell
./bindershell
# プロンプトが返ってきたら一時root取得成功
```

## Step 2: xfl-o77.mbn をフラッシュ

xperable の `boot/xfl-o77.mbn` を別ターミナルからデバイスに転送します。 

```powershell
adb push xfl-o77.mbn /data/local/tmp/xfl-o77.mbn
```
当たり前ですが、xfl-o77.mbnはちゃんと実行してるターミナルから見て渡してくださいねー  
上の実行例は、ターミナルが開いてるとこにxfl-o77がある前提です  
root shell 側でフラッシュ：

```bash
dd if=/data/local/tmp/xfl-o77.mbn of=/dev/block/bootdevice/by-name/xfl
sync
```
これで良いらしい？なんかあっけなく終わりますが大丈夫です  
再起動したり、bootloaderに入っても適用され続けるらしい。知らんけど  
ワイは問題なかった

## Step 3: Fastboot モードへ移行

```powershell
adb reboot bootloader
```

または電源オフ状態から音量上ボタンを押しながら USB 接続。青ランプが点いたら OK。  
なんか、緑ランプの変なモードに入って解除するんと思ってたのにワイがやった時はbootloaderに入って実行する感じだった。謎  
ワイのやり方違うかもしれない

## Step 4: Zadig でドライバを割り当て

Fastboot モード（青ランプ）で PC に繋ぐと `PID_0DDE` のデバイスが現れる。

1. Zadig を起動
2. Options → List All Devices にチェック
3. ドロップダウンで `Android (0FCE 0DDE)` を選択
4. ドライバを `libusbK` に設定して Install Driver  
これマジ謎。ちゃっぴー(claude 4.6)くんにやれって言われたけどやらなくていいかも？謎  
とりまxperiableで認識できればなんでもよいので、これ飛ばして無理だったらインスコしたらよいかも。

## Step 5: xperable でブートローダーアンロック

```powershell
.\xperable -B -U -s 0xf48080 -4 -c "oem unlock Y" -1 -c reboot -1
```

成功するとこんな感じのログが出る：

```
[+] Got LinuxLoader base addr 0x98dbd000 (0xa9ece111)
[+] LinuxLoader @ 0x98dbd000 patched successfully
```

あとはデバイスが自動リブートしてアンロック完了。  
けど、これがまた長かったんですよ  
なんか、落ちるメモリアドレス、落ちないメモリアドレスを特定していって落ちないギリギリのアドレスを実行したら解除できました(?)  
これやったけど、必要あるかどうかわかんないよー  

まーちゃっぴーに聞いたらなんとかなります。たぶん。

## 補足

- `0xf48080` は SO-01K 固有のサイズ。グローバル版（G8341 等）は `0xf49880` がREADMEの例として載ってる
- xfl の書き込みは永続的だが、xperable のパッチは一時的（Fastboot 起動ごとに再実行が必要）
- アンロック自体でユーザーデータは消えないが、DRM キー（TA 領域）はアンロック前にバックアップしておくこと

くらいですかね。ちゃっぴーいなかったらできなかった。クソ辛かった。  
海外ニキに聞いたり、ちゃっぴーに聞いたりしてようやっと出た。  
なんでみんなxperableの日本語解説されてるサイト見ずに使い方分かるの。怖いよ


ワイみたいな文字読めない人向けに残しておきます。ぐっどらっく。  

終わり。