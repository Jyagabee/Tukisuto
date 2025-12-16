---
title: "Macbook_to_ros2"
date: 2025-12-16T22:17:55+09:00
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

# はじめに

そんなもん入れるな！！！！！macなんて使うな！！！！！！！！！！

## 前提知識

ros2は、サポートされるTierがある。以下にtierを記述する。

### 公式サポートTier（REP 2000による定義）

| リリース                                                                                                                                                                                                                                           | macOSサポート    | バイナリ提供 |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- | ------------ |
| Foxy以前                                                                                                                                                                                                                                           | **Tier 1** | あり         |
| Galactic〜                                                                                                                                                                                                                                         | **Tier 3** | なし         |
| Jazzy/Kilted (現行)                                                                                                                                                                                                                                | **Tier 3** | なし         |
| Foxy Fitzroyまでは、macOSはTier 1プラットフォームとして扱われていた[Openrobotics](https://discourse.openrobotics.org/t/macos-support-in-ros-2-galactic-and-beyond/17891)。しかしGalactic以降、macOSは正式サポートから外れ、現在はTier 3となっている。 |                  |              |

### Apple Silicon (M1/M2/M3) 対応状況

公式にはサポートされていないが、コミュニティメンバーがROS 2 KiltedをmacOS Apple Siliconでネイティブ動作させたデモを公開 [Openrobotics](https://discourse.openrobotics.org/t/ros-2-kilted-running-natively-on-macos-apple-silicon-first-demo/51335)しました。このデモでは、Gazebo Ionic、ros2_control、MoveIt 2、Navigation2が動作している。

が、通信関係がクソ怖いのと、諸々インストールせねばならないのでネイティブで動作させるのは諦めましょう
デモを見る限り諸々をpythonでインスコしている。筆者はpythonが好きではないのでやらない。物好きだけやってくれ

## 今回のキモ

RC26はスマホコントローラからfastddsの通信を行う必要がある。UDP通信を行うためには、macで受信した通信をそのままubuntuに渡す必要がある。
そのため、ブリッジモードが存在する仮想化ソフトを使用する必要がある。
今回は、UTMを使用する。

# 環境

| 項目           | 内容                                 |
| -------------- | ------------------------------------ |
| モデル         | MacBook Air (M4, 2024)               |
| OS             | macOS 26.2 (Build 25C56)             |
| チップ         | Apple M4 (10コア: 4高性能 + 6高効率) |
| メモリ         | 24 GB                                |
| アーキテクチャ | arm64                                |

# インストール方法

## 1.仮想マシンの作成

UTMを起動します。
起動時の画面で「新規仮想マシンを作成」を選択します。
![](https://devio2023-media.developers.io/wp-content/uploads/2024/01/Group-55-640x483.png)

「仮想化」を選択します。
![](https://devio2023-media.developers.io/wp-content/uploads/2024/01/Group-54-640x483.png)

今回はubuntuを起動するので「Linux」を選択します。
![](https://devio2023-media.developers.io/wp-content/uploads/2024/01/Group-53-640x483.png)

先ほどダウロードしたUbuntuのイメージを選択します。
![](https://devio2023-media.developers.io/wp-content/uploads/2024/01/Group-56-640x483.png)

仮想マシンに割り当てるメモリとCPUコア数を指定します。
今回はメモリ4096, CPUコア数4としています。
![](https://devio2023-media.developers.io/wp-content/uploads/2024/01/Group-57-640x483.png)

仮想マシンに割り当てるストレージを指定します。
今回は後ほどDesktop版をインストールする予定なので32GBを割り当てています。
![](https://devio2023-media.developers.io/wp-content/uploads/2024/01/Group-58-640x483.png)

仮想マシンの名前を付けて保存します。
![](https://devio2023-media.developers.io/wp-content/uploads/2024/01/Group-59-640x483.png)

保存が完了すると作成した仮想マシンが一覧に表示されます。
起動ボタンを押します。
![](https://devio2023-media.developers.io/wp-content/uploads/2024/01/Group-51-640x483.png)

## 2. Ubuntuのインストール

Ubuntuのインストールは通常のインストールと同じ手順で実施します。

OSの設定後Rebootを行います。
この際、UTMのランチャー画面でCD/DVDのプルダウンから削除を選択します。
その後、再度ubuntuを起動します。
![](https://devio2023-media.developers.io/wp-content/uploads/2024/01/Group-50-640x495.png)

今回はUbuntu Desktop版が必要だったので以下のコマンドでインストールを行いました。

```bash
$ sudo apt install ubuntu-desktop
```

無事にインストールすることができました。

## 3. ネットワーク設定（ブリッジ接続）

- UTM で対象の Ubuntu VM を右クリック →「編集」
- 「デバイス」→「ネットワーク」を選択
- ネットワークモードを**ブリッジ（詳細）** に変更する

  - 同一ネットワーク上から直接アクセス可能にするため

---

## 4. Ubuntu 22.04 の日本語化

- 日本語ロケール・日本語入力環境を設定

```bash
### 日本語言語パック、フォント、入力メソッドなどをインストール
sudo apt update
sudo apt install -y task-japanese-gnome-desktop language-pack-gnome-ja-base language-pack-gnome-ja fonts-noto-cjk-extra

# ロケールを日本語に設定
sudo localectl set-locale LANG=ja_JP.UTF-8 LANGUAGE="ja_JP:ja"

# タイムゾーンを日本時間に設定
sudo timedatectl set-timezone Asia/Tokyo
```

- デスクトップ環境導入後に日本語化する想定
  - やり方忘れた　各自ググってくれ

## 5.ROS 2 Humble のインストール

- Ubuntu 22.04 向けの ROS 2 Humble をインストール

```bash
# universeリポジトリを有効化
sudo apt install software-properties-common
sudo add-apt-repository universe

# ROS 2 GPGキーを追加
sudo apt update && sudo apt install curl
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

# ソースリストにROS 2リポジトリを追加
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# インストール
sudo apt update
sudo apt upgrade
sudo apt install ros-humble-desktop

# 環境設定
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

- あとは各ノードで使用しているツールをaptでちまちま追加したら良い

---

## 6. Git の初期設定

- ユーザー名・メールアドレスを設定

`git config --global user.name "Your Name"`
`git config --global user.email "your@email"`

---

## 7. VS Code のインストール

- Ubuntu 向け VS Code をインストール

```bash
sudo snap install code --classic
```

---

## 8. bashrc の移植

- 既存の Ubuntu 22.04 環境から `.bashrc` をコピー
- ROS 環境変数やエイリアスを再利用

---

## 9. 動作確認

- ターミナル起動・日本語入力・ROS 2 ノード起動などを確認
- 問題なければ環境構築完了 お疲れ様でした

# あとがき

orbstack使いたかったけど、ブリッジ機能がないので環境消した。どうやらブリッジ機能追加のやつが異臭に投げられてるらしいがガン無視されてるらしい。
UTM以外にも、vmwareやらparallelsやらvirtualboxでも動くと思う。知らんけど。まあ好きなの使えばいいと思いますよ

ubuntu起動時は以下くらいの使用率
![image.png](/attachment/694152d87a944b7056e5664c)
![image.png](/attachment/694152ed7a944b7056e5669b)

ros2起動したら以下くらいの使用率
![image.png](/attachment/6941532c7a944b7056e566f3)
![image.png](/attachment/694153417a944b7056e56743)

GUIなのでクソあちーです
せっかく軽いのに持てへんや内科医

一応こんな感じに動いております
![image.png](/attachment/694153c07a944b7056e56816)

## 林檎ちゃんの良いところ

- 美しい
- 軽い
- 画面が綺麗
- スピーカーの音質がずば抜けている
- イヤホンジャックの音質がかなり良い
- brewで全部インストールできる
- zshが標準で使える
- 何もインストールしなくてもadbが使える
- androidのエミュレータがネイティブに動作する
- andoridのビルドが爆速
- トラックパッドの使い勝手が直感的
  - スクロールぬるぬる
- バッテリー持ちがエグい
  - 普通にフタ閉じてたら1週間持つ
- なんとなくモチベ上がる

## 林檎ちゃんの良くないところ

- androidを物理的に繋げてもファイル転送できない
- androidを物理的に繋げてテザリングできない
  - pixelならいける。テザリングのプロトコルが他スマホと違うらしい
- 学内ネットに繋げた際に900sの認証で強制切断される
- 学内ネットに繋げてもプリンターにアクセスできない←これ一番クソ
- トラックポイントがない
- トラックパッドの文字選択がクソ
- linux,windowsにRDPで繋げた際に右のcommand邪魔
- 高い！！！！！！！！！こんなん買うやつアホか信者かイキり野郎しかおらん
- 高くて使いたおすのに心理的負担がある
- ポートが少ない
  - USB-Aが無い
  - HDMIが無い

総合的に使い勝手はクソです。学内ネット強切断、プリンター使えない、トラックポイントがないのが致命的
普通にThinkPad買って、2スロット目にubuntu入れるのがいっちゃん楽です。

mac買ってもユーザーが知識なくてカスならmacの性能はフルに使用できません。←これ重要
