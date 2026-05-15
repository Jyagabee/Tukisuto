---
title: "学内ネットワークでTailscaleを使うときの備忘録"
date: 2026-05-15T19:50:24+09:00
draft: false
featured: false
showHero: false  # ヒーロー画像を表示するか
heroStyle: "basic"  # ヒーロー画像のスタイル（basic, background, thumbなど）
heroImage: "hero.jpg"  # 記事のメイン画像
thumbnail: "thumbnail.jpg"  # 一覧表示用画像
tags: ["Tailscale", "学内ネットワーク", "VPN", "Windows", "Linux", "macOS", "Exit Node", "DNS"]
summary: "学内ネットワークからTailscaleを使うときに遭遇したNoState、DERP経由、Exit Node、DNSまわりの挙動をOS別に整理した備忘録。"
---
# 学内ネットワークでTailscaleを使うときの備忘録

## 概要

学内ネットワークからTailscaleを使おうとしたときに、WindowsとLinuxで少し違う挙動が出たので整理しておく。

特に重要なのは、以下の3点。

- Windowsでは `NoState` になり、Tailscaleが起動していても正常に初期化できないことがあった
- Linuxでは最初 `DERP(tok)` 経由だったが、しばらくすると直接接続に切り替わった
- Exit NodeやDNS設定を有効にすると、経路問題とDNS問題が混ざって原因が分かりにくくなる

学内ネットだからTailscaleが完全に使えないというより、プロキシ、DNS、DERP、Exit Nodeの設定が絡んで不安定に見える、という理解が近そう。

---

# 共通事項

## 学内ネットで見えていたプロキシ

Windowsで `tailscale netcheck` を実行したところ、Tailscaleが学内プロキシを認識していた。

```text
http://wwwproxy-a10.***********.ac.jp:8080
````

ログでは以下のような表示が出ていた。

```text
tshttpproxy: using proxy "http://wwwproxy-a10.***********.ac.jp:8080" for URL: "https://controlplane.tailscale.com/"
Failed to fetch a DERP map
```

つまり、Tailscaleはプロキシ経由で外へ出ようとしているが、DERP mapの取得に失敗していた。

DERP mapは、Tailscaleが中継サーバーを使うための情報。これが取れないと、ネットワーク診断や接続確立がうまく進まない可能性がある。

---

## Tailscaleの接続経路の見方

`tailscale ping` を使うと、相手端末へどの経路で接続しているか確認できる。

例：

```bash
tailscale ping st170e
```

表示の意味は以下。

| 表示                       | 意味           | 状態              |
| ------------------------ | ------------ | --------------- |
| `via DERP(tok)`          | 東京のDERPリレー経由 | つながるが遅め         |
| `via 172.xx.xx.xx:41641` | 直接接続         | 良好              |
| `no response`            | 応答なし         | 接続不可または相手がオフライン |

今回Linux側では、最初はDERP経由だった。

```text
pong from st170e via DERP(tok) in 36ms
pong from st170e via DERP(tok) in 34ms
pong from st170e via DERP(tok) in 32ms
pong from st170e via DERP(tok) in 28ms
```

その後、直接接続に切り替わった。

```text
pong from st170e via 172.26.160.101:41641 in 8ms
```

この状態はかなり良い。

最初にDERP経由でも、少し待つと直接接続に切り替わる場合があるので、`tailscale ping` は1回だけで判断しない方がよい。

---

# Windows

## 発生した状態

Windowsでは、Tailscaleサービス自体は起動できた。

```powershell
net start Tailscale
```

実行結果：

```text
Tailscale サービスは正常に開始されました。
```

しかし、状態確認では以下のようになった。

```powershell
tailscale status
```

```text
# Health check:
#     - Tailscale is starting. Please wait.

unexpected state: NoState
```

## 状況の解釈

`NoState` は、Tailscaleのサービスは動いているが、ログイン状態や初期化状態が整っていないような状態だと考えられる。

このときは、単にサービスを起動しただけでは不十分で、以下を確認する必要がある。

* `tailscale up` が通るか
* ログイン状態が有効か
* 学内プロキシ経由でTailscaleの制御サーバーへ到達できるか
* DNSやHTTPS通信が邪魔されていないか

---

## Windowsで使う基本コマンド

### サービス起動

```powershell
net start Tailscale
```

### サービス停止

```powershell
net stop Tailscale
```

### サービス再起動

```powershell
Restart-Service Tailscale
```

または、

```powershell
net stop Tailscale
net start Tailscale
```

### Tailscaleに接続

```powershell
tailscale up
```

### Tailscaleを切断

```powershell
tailscale down
```

### 状態確認

```powershell
tailscale status
```

### ネットワーク診断

```powershell
tailscale netcheck
```

### 相手端末への疎通確認

```powershell
tailscale ping st170e
```

### 自分のTailscale IP確認

```powershell
tailscale ip -4
tailscale ip -6
```

---

## WindowsでExit Nodeを使う

Exit Nodeを使う場合は、以下のように指定する。

```powershell
tailscale up --exit-node=st170e --exit-node-allow-lan-access
```

Tailscale IPで直接指定してもよい。

```powershell
tailscale up --exit-node=100.x.y.z --exit-node-allow-lan-access
```

`--exit-node-allow-lan-access` は、Exit Node使用中でもローカルLANへのアクセスを許可するオプション。

これを付けないと、学内LANや手元のルーター管理画面にアクセスしづらくなる可能性がある。

### Exit Nodeを解除

```powershell
tailscale up --exit-node=
```

完全に切る場合は以下。

```powershell
tailscale down
```

---

## Windowsでルートを確認する

Exit NodeやDNSまわりがおかしいときは、デフォルトルートを見る。

### IPv4デフォルトルート確認

```powershell
Get-NetRoute -DestinationPrefix "0.0.0.0/0" |
  Sort-Object RouteMetric |
  Format-Table ifIndex, InterfaceAlias, NextHop, RouteMetric, InterfaceMetric, PolicyStore
```

確認された例：

```text
ifIndex InterfaceAlias NextHop       RouteMetric InterfaceMetric PolicyStore
------- -------------- -------       ----------- --------------- -----------
     19 Wi-Fi 2        172.18.27.254           0              35
```

この場合、通常のIPv4デフォルトルートはWi-Fi側に向いている。

### IPv6デフォルトルート確認

```powershell
Get-NetRoute -DestinationPrefix "::/0" |
  Sort-Object RouteMetric |
  Format-Table ifIndex, InterfaceAlias, NextHop, RouteMetric, InterfaceMetric, PolicyStore
```

このときはIPv6のデフォルトルートが見つからなかった。

```text
No MSFT_NetRoute objects found
```

### インターフェースの優先度確認

```powershell
Get-NetIPInterface |
  Sort-Object InterfaceMetric |
  Format-Table InterfaceAlias, AddressFamily, InterfaceMetric, ConnectionState, NlMtu
```

TailscaleのMTUは `1280`、通常の物理NICは `1500` になっていた。

---

# Ubuntu / Linux

## 発生した状態

Ubuntu/Linuxでは、`tailscale ping ******` を実行した。

```bash
tailscale ping ******
```

最初はDERP経由だった。

```text
pong from st170e via DERP(tok) in 36ms
pong from st170e via DERP(tok) in 34ms
pong from st170e via DERP(tok) in 32ms
pong from st170e via DERP(tok) in 28ms
```

しかし、その後に直接接続へ切り替わった。

```text
pong from ****** via 172.26.160.101:41641 in 8ms
```

## 状況の解釈

これは正常寄りの挙動。

Tailscaleは最初にDERPリレー経由でつながり、その後UDP hole punching、つまりNAT越えの直接接続に成功すると、直接接続へ切り替わる。

今回の場合、最終的に8ms程度で直接接続できていたので、Linux側はかなり良い状態だった。

---

## Ubuntu/Linuxで使う基本コマンド

### tailscaledを起動

```bash
sudo systemctl start tailscaled
```

### tailscaledを停止

```bash
sudo systemctl stop tailscaled
```

### tailscaledを再起動

```bash
sudo systemctl restart tailscaled
```

### 自動起動を有効化

```bash
sudo systemctl enable tailscaled
```

### Tailscaleに接続

```bash
sudo tailscale up
```

### Tailscaleを切断

```bash
sudo tailscale down
```

### 状態確認

```bash
tailscale status
```

### ネットワーク診断

```bash
tailscale netcheck
```

### 疎通確認

```bash
tailscale ping st170e
```

### 自分のTailscale IP確認

```bash
tailscale ip -4
tailscale ip -6
```

---

## Ubuntu/LinuxでExit Nodeを使う

### Exit Nodeを指定

```bash
sudo tailscale up --exit-node=****** --exit-node-allow-lan-access
```

Tailscale IPで指定する場合：

```bash
sudo tailscale up --exit-node=100.x.y.z --exit-node-allow-lan-access
```

### Exit Nodeを解除

```bash
sudo tailscale up --exit-node=
```

### 完全に切断

```bash
sudo tailscale down
```

---

## LinuxでルーティングやDNSを見る

### デフォルトルート確認

```bash
ip route
```

### Tailscaleインターフェース確認

```bash
ip addr show tailscale0
```

### DNS確認

```bash
resolvectl status
```

または、

```bash
cat /etc/resolv.conf
```

---

# macOS

## 今回の扱い

今回の検証ログでは、macOS上での学内ネットTailscale検証はあまり多くない。

ただし、基本的な確認方法はLinuxと似ている。

macOSの場合はGUI版TailscaleとCLI版Tailscaleの状態が混ざる可能性があるので、CLIで変な挙動をした場合はメニューバーのTailscaleアプリ側も確認する。

---

## macOSで使う基本コマンド

### 状態確認

```bash
tailscale status
```

### ネットワーク診断

```bash
tailscale netcheck
```

### 疎通確認

```bash
tailscale ping ******
```

### 自分のTailscale IP確認

```bash
tailscale ip -4
tailscale ip -6
```

### Exit Nodeを指定

```bash
sudo tailscale up --exit-node=st170e --exit-node-allow-lan-access
```

### Exit Nodeを解除

```bash
sudo tailscale up --exit-node=
```

---

# DNS / NextDNS / Override DNS

## 問題の切り分け

Tailscaleでは、管理画面側でDNSサーバーを指定したり、`Override DNS servers` を使ったりできる。

ただし、学内ネットで問題が出ているときにDNSまでTailscale側に寄せると、問題の切り分けが難しくなる。

例えば以下の問題が混ざる。

* Tailscaleの接続問題
* Exit Nodeの経路問題
* DNSの名前解決問題
* NextDNSのフィルタリング問題
* CDNの地域判定問題
* 学内プロキシの問題

そのため、まずはTailscaleの疎通だけを確認するのがよい。

---

## おすすめの切り分け順

```text
Tailscaleだけ接続する
↓
Exit Nodeは使わない
↓
Override DNS serversも切る
↓
tailscale statusを見る
↓
tailscale netcheckを見る
↓
tailscale ping 相手端末 を実行する
↓
直接接続になるか確認する
↓
最後にExit NodeやDNS設定を足す
```

最初からExit NodeやDNSを全部有効にすると、何が原因で壊れているのか分からなくなる。

---

# トラブル時の確認フロー

```text
Tailscaleが変
  ↓
tailscale status
  ↓
NoState?
  ├─ Yes
  │   ↓
  │   tailscale up を実行
  │   サービス再起動
  │   ログイン状態を確認
  │   学内プロキシ経由で制御サーバーに到達できるか確認
  │
  └─ No
      ↓
      tailscale netcheck
      ↓
      DERP map取得失敗?
      ├─ Yes
      │   ↓
      │   学内プロキシ・DNS・HTTPS到達性を疑う
      │
      └─ No
          ↓
          tailscale ping st170e
          ↓
          via DERP?
          ├─ Yes
          │   ↓
          │   何回か待つ
          │   direct接続に切り替わるか見る
          │
          └─ No
              ↓
              direct接続なら良好
```

---

# OS別まとめ

## Windows

Windowsでは、サービスは起動していても `NoState` になることがあった。

```text
unexpected state: NoState
```

この状態では、Tailscaleがまだ正常にログイン・初期化できていない可能性がある。

また、`tailscale netcheck` では学内プロキシを使っていた。

```text
http://wwwproxy-a10.***********.ac.jp:8080
```

そのうえで、DERP mapの取得に失敗していた。

Windowsでは、Tailscaleの起動状態、ログイン状態、プロキシ、DNS、Exit Nodeの影響を分けて確認する必要がある。

---

## Ubuntu / Linux

Linuxでは、最初はDERP経由だった。

```text
via DERP(tok)
```

しかし、少し待つと直接接続へ切り替わった。

```text
via 172.26.160.101:41641 in 8ms
```

これはかなり良い状態。

Linuxでは、`tailscale ping` を1回だけ見るのではなく、数回流してdirect接続に切り替わるか確認するのがよい。

---

## macOS

今回の検証ではmacOSの情報は少なめ。

基本的にはLinuxと同じように、

```bash
tailscale status
tailscale netcheck
tailscale ping st170e
```

を見ればよい。

ただし、macOSではGUI版Tailscaleの状態も確認する。

---

# 最低限覚えておくコマンド

## Windows

```powershell
net start Tailscale
net stop Tailscale
Restart-Service Tailscale

tailscale up
tailscale down
tailscale status
tailscale netcheck
tailscale ping ******

tailscale up --exit-node=****** --exit-node-allow-lan-access
tailscale up --exit-node=
```

## Ubuntu / Linux

```bash
sudo systemctl start tailscaled
sudo systemctl stop tailscaled
sudo systemctl restart tailscaled
sudo systemctl enable tailscaled

sudo tailscale up
sudo tailscale down
tailscale status
tailscale netcheck
tailscale ping st170e

sudo tailscale up --exit-node=****** --exit-node-allow-lan-access
sudo tailscale up --exit-node=
```

## macOS

```bash
tailscale status
tailscale netcheck
tailscale ping ******
tailscale ip -4
tailscale ip -6

sudo tailscale up --exit-node=****** --exit-node-allow-lan-access
sudo tailscale up --exit-node=
```

---

# 結論

学内ネットからTailscaleを使う場合、単純に「Tailscaleが壊れている」と見るより、以下が絡んでいると考えた方がよい。

* 学内プロキシ
* DNS
* DERPリレー
* UDPの直接通信
* Exit Node
* OSごとのネットワーク実装

今回の重要なメモはこれ。

```text
WindowsではNoStateやDERP map取得失敗が出ることがある。
Linuxでは最初DERPでも、少し待つとdirect接続になることがある。
Exit NodeとDNS設定は、Tailscale本体の疎通確認が終わってから触る。
tailscale pingは1回だけで判断しない。
```

まずは、

```bash
tailscale status
tailscale netcheck
tailscale ping ******
```

この3つを見る。

そのうえで、DERP経由なのか、直接接続なのか、Exit NodeやDNSが絡んでいるのかを分けて考える。
