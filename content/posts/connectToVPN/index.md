---
title: "ubuntuからVPN経由で学内ネットに繋げる"
date: 2026-04-04T22:31:24+09:00
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

# Remote-VPN（Linux/Ubuntu）

接続先: `ras2.接続先:ポート`　認証: `a学籍番号`

## セットアップ
```bash
sudo apt install openfortivpn
```

## 設定ファイル
`vim /etc/openfortivpn/config`
```
host = URL
port = ポート番号
username = a学籍番号
```
```bash
sudo chmod 600 /etc/openfortivpn/config
```

## 接続・切断
```bash
sudo openfortivpn   # 接続
Ctrl+C              # 切断
```

> 外部回線から使用すること。