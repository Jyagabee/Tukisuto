---
title: 週報メモ
date: 2025-03-03T23:20:35+09:00
draft: false
featured: false
---
## memo
個人的に忘れそうだったのでメモ

# Hugoで記事に画像を添付する方法

Hugoで記事に画像を添付するには、主に2つの方法があります：

## 1. Page Bundle方式（推奨）

記事と画像を同じフォルダに配置する方法です。これが最も管理しやすい方法です：
```
content/
└── posts/
    └── my-post/
        ├── index.md  # 記事本文
        ├── image1.jpg  # 画像ファイル
        └── image2.png  # 画像ファイル
        
```
### 手順

1. まず記事用のディレクトリを作成
	```
	hugo new content/posts/my-post/index.md
	```
2. 同じディレクトリ内に画像を配置
3. Markdown内で相対パスで参照
## 本文

以下は画像です：
```
![画像の説明](image1.jpg "画像のタイトル")  

サイズ指定もできます：

![画像の説明](image2.png "画像のタイトル"){width=300px}
```
## 2. 静的ファイル方式

共通で使いたい画像を [static](vscode-file://vscode-app/c:/Users/ameri/AppData/Local/Programs/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html) ディレクトリに配置する方法：
```
# static/images/ ディレクトリに画像を配置
```
Markdown内での参照：
```
![画像の説明](/images/sample.jpg "画像のタイトル")
```
## 注意事項

1. 画像ファイル名には日本語や特殊文字を避け、英数字とハイフンを使用するのが安全です
2. 大きな画像は最適化してからアップロードしましょう
3. 常に代替テキスト（alt属性）を設定しましょう

Blowfishテーマでは、Page Bundle方式が特に推奨されており、画像処理機能なども利用できます。