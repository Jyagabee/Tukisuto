---
title: "ubuntuからVPN経由で学内ネットに繋げる"
date: 2026-04-04T22:31:24+09:00
draft: false
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


終わり。
