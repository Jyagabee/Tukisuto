---
title: "Ros2vscodesettings"
date: 2026-03-06T23:19:14+09:00
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

# VSCode 開発環境セットアップ手順

ROS2 (Humble) + clangd を使った開発環境の構築手順と設定解説です。

やーなんか、ツレとお話してたら「ros2使ってるならclangd使うといいよん」って言われたんでclaude-code 4.6に書かせました


なんかros2の設定文献少ない…少なくない？って思ったんでメモるだけメモるンゴ

ッぱclaude-code 4.6ニキエグいわ　ワイがゴにョる必要性ないなあ…

---

## 目次

1. [前提条件](#前提条件)
2. [拡張機能のインストール](#拡張機能のインストール)
3. [clangd のインストール](#clangd-のインストール)
4. [ビルド手順](#ビルド手順)
5. [設定ファイル解説](#設定ファイル解説)
   - [extensions.json](#extensionsjson)
   - [settings.json](#settingsjson)
   - [tasks.json](#tasksjson)
   - [launch.json](#launchjson)
   - [c_cpp_properties.json](#c_cpp_propertiesjson)
   - [.clangd](#clangd-1)
6. [clangd とは](#clangd-とは)
7. [デバッグの使い方](#デバッグの使い方)
8. [ショートカットキー一覧](#ショートカットキー一覧)
9. [よくあるトラブル](#よくあるトラブル)

---

## 前提条件

- Ubuntu 22.04
- ROS2 Humble インストール済み
- VSCode インストール済み

---

## 拡張機能のインストール

`.vscode/extensions.json` に推奨拡張機能が定義されています。
VSCodeでこのワークスペースを開くと、インストールを促す通知が表示されます。

一括インストールする場合：

```bash
code --install-extension ms-vscode.cpptools
code --install-extension ms-vscode.cpptools-extension-pack
code --install-extension ms-python.python
code --install-extension ms-python.debugpy
code --install-extension ms-vscode.cmake-tools
code --install-extension twxs.cmake
code --install-extension llvm-vs-code-extensions.vscode-clangd
code --install-extension usernamehw.errorlens
```

---

## clangd のインストール

```bash
sudo apt install clangd
clangd --version   # バージョン確認
```

---

## ビルド手順

clangdはビルド情報（`compile_commands.json`）を必要とするため、**初回および新しいファイルを追加した後**は以下の手順を実行してください。

### ステップ 1: ビルド

`Ctrl+Shift+B` でデフォルトビルドタスク（`colcon build`）が実行されます。

またはターミナルで：

```bash
source /opt/ros/humble/setup.bash
colcon build --symlink-install --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```

### ステップ 2: compile_commands.json の更新

`Ctrl+Shift+P` → `Tasks: Run Task` → `make compile_commands` を選択

またはターミナルで：

```bash
bash scripts/make_compile_commands.sh
```

### ステップ 3: clangd の再起動

`Ctrl+Shift+P` → `clangd: Restart language server`

---

## 設定ファイル解説

### extensions.json

**役割**: チーム全員に推奨する拡張機能を定義します。

ワークスペースを開いたとき「推奨拡張機能をインストールしますか？」と通知が出ます。

| 拡張機能 | 用途 |
|---|---|
| `ms-vscode.cpptools` | C/C++基本サポート（IntelliSenseは無効化してclangdを使用） |
| `ms-vscode.cpptools-extension-pack` | C++関連ツール一式 |
| `ms-python.python` | Pythonサポート |
| `ms-python.debugpy` | Pythonデバッガ |
| `ms-vscode.cmake-tools` | CMakeサポート |
| `twxs.cmake` | CMakeファイルのシンタックスハイライト |
| `llvm-vs-code-extensions.vscode-clangd` | clangd（C++補完・解析のメイン） |
| `usernamehw.errorlens` | エラー・警告をコード行の右に表示 |

---

### settings.json

**役割**: プロジェクト共通のVSCode設定です。チームメンバー全員に同じ設定が適用されます。

#### ROS2設定

```json
"ROS2.distro": "humble"
```
ROS拡張機能にROS2のディストリビューションを伝えます。`ros2 topic list` 等のROS2コマンドをVSCode内から使う際に参照されます。

#### Python補完パス

```json
"python.autoComplete.extraPaths": [ ... ]
"python.analysis.extraPaths": [ ... ]
```
ROS2のPythonパッケージをPylanceが認識できるよう、インストール先のパスを追加しています。
- `/opt/ros/humble/...` — ROS2本体のPythonパッケージ
- `${workspaceFolder}/install/...` — このワークスペースでビルドしたカスタムメッセージ等

#### ファイル設定

```json
"files.associations": {
  "*.hpp": "cpp",
  "*.cpp": "cpp"
}
```
`.hpp` / `.cpp` ファイルをC++として扱うようVSCodeに指示します。これにより適切なシンタックスハイライトと補完が機能します。

```json
"editor.formatOnSave": false
```
保存時の自動フォーマットを無効化しています。意図しないコード変更を防ぐためです。

#### C/C++拡張の無効化

```json
"C_Cpp.intelliSenseEngine": "disabled"
"C_Cpp.errorSquiggles": "disabled"
```
Microsoft C/C++拡張のIntelliSenseとエラー表示を無効化しています。
clangdと同時に有効にするとエラー表示が二重になって混乱するため、clangdに一本化しています。

#### clangd設定

```json
"clangd.path": "clangd"
```
clangdの実行ファイルのパスです。`apt install clangd` でインストールした場合はこのままで動作します。

```json
"clangd.arguments": [...]
```

| 引数 | 説明 |
|---|---|
| `--background-index` | バックグラウンドでインデックスを構築。起動直後から補完が使えるようになる |
| `--compile-commands-dir` | `compile_commands.json` の場所を指定 |
| `--header-insertion=never` | 補完時にヘッダを自動挿入しない。意図しない `#include` の追加を防ぐ |
| `--completion-style=detailed` | 補完候補に型情報などの詳細を表示する |
| `--function-arg-placeholders=false` | 関数補完時に引数のプレースホルダを挿入しない |
| `--clang-tidy` | clang-tidyによる静的解析を有効化。バグになりやすいコードパターンを検出する |
| `--query-driver` | clangdにGCCのパスを教える。これによりC++標準ライブラリ（`<string>`, `<vector>` 等）のヘッダパスを自動取得できる。**これがないとC++標準ライブラリが見つからないエラーが大量に出る** |

---

### tasks.json

**役割**: VSCode内から実行できるシェルコマンドを定義します。`Ctrl+Shift+B` や `Tasks: Run Task` から呼び出せます。

#### colcon build（デフォルトビルドタスク）

```json
{
  "label": "colcon build",
  "command": "colcon build --symlink-install --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON",
  "group": { "kind": "build", "isDefault": true }
}
```

- `--symlink-install`: インストール時にファイルをコピーせずシンボリックリンクを作成。Pythonスクリプト等を編集した際にリビルド不要になる
- `--cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON`: 各パッケージのビルドディレクトリに `compile_commands.json` を生成する（clangdに必要）
- `"isDefault": true`: `Ctrl+Shift+B` でこのタスクが実行される

#### make compile_commands

```json
{
  "label": "make compile_commands",
  "command": "bash scripts/make_compile_commands.sh",
  "dependsOn": "colcon build"
}
```

- 各パッケージに散らばった `compile_commands.json` をワークスペースルートの1ファイルにマージする
- `"dependsOn": "colcon build"`: このタスク実行前に `colcon build` が自動実行される

---

### launch.json

**役割**: デバッグ実行の設定を定義します。`F5` または左サイドバーの「実行とデバッグ」から使います。

#### GDB: main_exec（C++デバッガ）

```json
{
  "name": "GDB: main_exec",
  "type": "cppdbg",
  "program": "${workspaceFolder}/build/main_executor/main_exec"
}
```

GDBを使ってC++実行ファイルを直接デバッグします。
- ブレークポイントで停止、変数の中身を確認できます
- `--ros-args --params-file` でyamlパラメータを渡した状態で起動します
- `LD_LIBRARY_PATH` に各パッケージの共有ライブラリパスを設定しているため、ライブラリが正しくロードされます

> **注意**: デバッグビルド（`-DCMAKE_BUILD_TYPE=Debug`）でビルドしないと、最適化によりブレークポイントが正確に機能しない場合があります。

#### ROS: Launch (main_exec)（launchファイルデバッガ）

```json
{
  "name": "ROS: Launch (main_exec)",
  "type": "ros2",
  "request": "launch",
  "target": "${workspaceFolder}/main_executor/launch/main_exec.launch.py"
}
```

ROS2のlaunchファイルを使ってデバッグ起動します。複数ノードをまとめて起動でき、ROS2の通常の起動方法に近い形でデバッグできます。

#### ROS: Attach to node（実行中ノードにアタッチ）

```json
{
  "name": "ROS: Attach to node",
  "type": "ros2",
  "request": "attach"
}
```

すでに起動中のROS2ノードにデバッガをアタッチします。通常のlaunchコマンドで起動したノードをあとからデバッグしたい場合に使います。

#### Python: Current File（Pythonデバッガ）

```json
{
  "name": "Python: Current File",
  "type": "debugpy",
  "program": "${file}"
}
```

現在開いているPythonファイルをデバッグ実行します。`PYTHONPATH` にROS2のパスが設定されているため、ROS2のメッセージ型等をインポートした状態で動作します。

---

### c_cpp_properties.json

**役割**: Microsoft C/C++拡張向けのインクルードパス設定です。

現在はclangdを使用しているため、この設定はIntelliSenseには使われません。ただし、C/C++拡張が提供する一部の機能（コード閲覧など）では参照されます。

`compileCommands` に `compile_commands.json` のパスが設定されており、clangdと同じビルド情報を参照するようになっています。

---

### .clangd

**役割**: clangd本体の動作設定ファイルです。ワークスペースルートに置きます。

```yaml
CompileFlags:
  Add:
    - "-std=c++17"
    - "-I/opt/ros/humble/include"
```
`compile_commands.json` に加えて補足するコンパイルフラグです。ROS2 Humbleは C++17 を使用します。

```yaml
Diagnostics:
  UnusedIncludes: None
```
未使用インクルードの警告を無効化しています。ROS2のヘッダは内部で多くのものをインクルードしており、警告が多くなりすぎるためです。

```yaml
  ClangTidy:
    Remove:
      - readability-*
      - modernize-*
```
readability（命名規則等）とmodernize（新C++スタイルへの書き換え提案）を無効化しています。既存コードへの大量の警告を抑制するためです。

```yaml
InlayHints:
  Enabled: Yes
  ParameterNames: Yes
  DeducedTypes: Yes
```
関数呼び出しの引数名や、`auto` で推論された型をコード上に薄く表示します。

---

## clangd とは

clangdは **LLVMプロジェクトが開発するC/C++用のLanguage Server** です。
コードを書きながら以下の機能をリアルタイムで提供します。

| 機能 | 説明 |
|---|---|
| コード補完 | 変数名・関数名・メンバを自動補完 |
| エラー検出 | コンパイルエラーをリアルタイムで波線表示 |
| 定義へジャンプ | `F12` で関数・クラスの定義へ移動 |
| 参照の検索 | `Shift+F12` でその変数・関数が使われている箇所を一覧表示 |
| ホバー情報 | 関数にカーソルを合わせると型・説明を表示 |
| InlayHints | 引数名・推論された型を薄く表示 |
| リファクタリング | シンボルの一括リネームなど |

### なぜ ROS2 では clangd が適しているか

VSCodeにデフォルトで入っているMicrosoft製の **C/C++ IntelliSense** と比べて、clangdには以下のメリットがあります。

**C/C++ IntelliSense の問題点：**
- インクルードパスを `c_cpp_properties.json` に手動で全列挙する必要がある
- ROS2はパッケージが多く、インクルードパスが膨大で管理が難しい
- パッケージを追加するたびに設定ファイルの更新が必要

**clangd のメリット：**
- `compile_commands.json` を読み込むため、**実際のビルドと同じインクルードパス・コンパイルフラグ**を使う
- パッケージを追加してもビルドさえすれば自動的に反映される
- 解析精度が高く、誤検知が少ない

> **補足**: `compile_commands.json` とは、各ソースファイルをどのコンパイルオプションでビルドするかを記録したファイルです。`-DCMAKE_EXPORT_COMPILE_COMMANDS=ON` を付けてビルドすると自動生成されます。

---

## デバッグの使い方

### C++ノードのデバッグ手順

1. **デバッグビルドに切り替える**（ブレークポイントを正確に機能させるため）

   ```bash
   colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Debug -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
   ```

2. **ブレークポイントを設置する**

   止めたいコードの行番号の左をクリックすると赤丸が表示されます。

3. **デバッグ開始**

   左サイドバーの「実行とデバッグ」アイコン（▷に虫のアイコン）をクリックし、
   上部のドロップダウンから構成を選んで `F5` を押します。

   | 構成名 | 使いどころ |
   |---|---|
   | `GDB: main_exec` | C++実行ファイルを直接起動してデバッグ |
   | `ROS: Launch (main_exec)` | launchファイル経由で全ノードを起動してデバッグ |
   | `ROS: Attach to node` | すでに起動中のノードにあとからデバッガをつなぐ |
   | `Python: Current File` | 開いているPythonスクリプトをデバッグ |

4. **デバッグ操作**

   | キー | 動作 |
   |---|---|
   | `F5` | 実行再開（次のブレークポイントまで進む） |
   | `F10` | 1行ずつ実行（関数の中には入らない） |
   | `F11` | 1行ずつ実行（関数の中に入る） |
   | `Shift+F11` | 現在の関数から抜け出す |
   | `Shift+F5` | デバッグ停止 |

5. **変数の確認**

   デバッグ中は左パネルの「変数」欄に現在のスコープの変数が表示されます。
   ウォッチ式を追加すれば特定の変数を常時監視できます。

### ROS2ノードのデバッグ上の注意

- ROS2ノードはデバッグ実行前に `ros2 daemon start` が動いている必要があります
- `ROS_DOMAIN_ID` が他のPCと一致していることを確認してください（`launch.json` では `14` に設定）
- 複数ノードが絡む問題のデバッグには `ROS: Attach to node` が有効です

---

## ショートカットキー一覧

### ビルド・タスク

| キー | 動作 |
|---|---|
| `Ctrl+Shift+B` | デフォルトビルドタスク（`colcon build`）を実行 |
| `Ctrl+Shift+P` → `Tasks: Run Task` | 任意のタスクを選んで実行（`make compile_commands` 等） |

### デバッグ

| キー | 動作 |
|---|---|
| `F5` | デバッグ開始 / 実行再開 |
| `Shift+F5` | デバッグ停止 |
| `F10` | ステップオーバー（1行実行・関数に入らない） |
| `F11` | ステップイン（1行実行・関数に入る） |
| `Shift+F11` | ステップアウト（現在の関数から抜ける） |
| `F9` | カーソル行にブレークポイントをトグル |

### コード編集（clangd）

| キー | 動作 |
|---|---|
| `F12` | 定義へジャンプ |
| `Alt+F12` | 定義をピークで表示（画面移動せずに確認） |
| `Shift+F12` | 参照を検索（その変数・関数が使われている箇所を一覧） |
| `F2` | シンボルのリネーム（一括置換） |
| `Ctrl+.` | クイックフィックス（エラーの修正候補を表示） |
| `Ctrl+Shift+O` | ファイル内のシンボル（関数・クラス）一覧 |
| `Ctrl+T` | ワークスペース全体のシンボル検索 |

### 一般

| キー | 動作 |
|---|---|
| `Ctrl+Shift+P` | コマンドパレット（VSCodeの全機能を検索・実行） |
| `Ctrl+P` | ファイルを素早く開く |
| `Ctrl+Shift+E` | エクスプローラーを開く |
| `Ctrl+Shift+G` | ソース管理（Git）を開く |
| `Ctrl+`` ` | ターミナルを開く/閉じる |

---

## よくあるトラブル

### C++標準ライブラリが見つからない（`'condition_variable' file not found` 等）

`--query-driver` の設定が効いていない可能性があります。

```bash
which g++   # /usr/bin/g++ が表示されることを確認
```

`settings.json` の `clangd.arguments` に以下が含まれているか確認：

```
"--query-driver=/usr/bin/g++,/usr/bin/gcc,/usr/bin/c++"
```

### エラーが消えない

以下を順番に試してください：

1. `bash scripts/make_compile_commands.sh` を再実行
2. `Ctrl+Shift+P` → `clangd: Restart language server`
3. VSCodeを再起動

### ビルドはできるのに clangd がエラーを出す

`compile_commands.json` が古い可能性があります。`make_compile_commands.sh` を再実行してください。

### 新しいパッケージを追加したら補完・解析が効かない

ビルドして `make_compile_commands.sh` を再実行すれば反映されます。
