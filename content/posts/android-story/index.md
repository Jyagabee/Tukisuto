---
title: "あんどろいどのお話"
date: 2026-02-13T16:02:27+09:00
draft: false
featured: false
showHero: false
categories: ["Android", "技術解説"]
tags: ["Android", "iOS", "アプリ開発", "Kotlin", "Activity", "Fragment", "カスタムROM", "Pixel", "HyperOS", "Bootloader Unlock"]
summary: "iOS開発者向けAndroid入門ガイド。AndroidとiOSの違い、アプリ構造、ライフサイクル、バージョン別の変更点、特有の「癖」について詳しく解説。"
---opus兄貴にぶん投げたやつ  
ぽみゃーらopus兄貴に感謝せーよ

## 目次

1. [Androidとは何か](#1-androidとは何か)
2. [iOSとの根本的な違い](#2-iosとの根本的な違い)
3. [アプリ構造とライフサイクル](#3-アプリ構造とライフサイクル)
4. [バージョン別の歴史と現在のターゲット](#4-バージョン別の歴史と現在のターゲット)
5. [Android 12以降の重要な変更点](#5-android-12以降の重要な変更点)
6. [Android特有の「癖」と落とし穴](#6-android特有の癖と落とし穴)
7. [配信とテスト](#7-配信とテスト)

---

## 1. Androidとは何か

### 1.1 Androidの基本情報

| 項目 | 内容 |
|------|------|
| 開発元 | Google（元Android Inc.、2005年に買収） |
| 初版リリース | 2008年9月 |
| ベースOS | Linuxカーネル |
| ライセンス | Apache License 2.0（オープンソース） |
| 現在の最新版 | Android 15（2024年〜） |

### 1.2 Androidの思想

```
┌─────────────────────────────────────────────────────────┐
│                    Android の思想                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   🔓 オープン        誰でもソースコードを閲覧・改変可能    │
│                                                         │
│   🎨 カスタマイズ    メーカーが自由にUIや機能を追加        │
│                                                         │
│   🔗 連携性          アプリ間のデータ共有が容易           │
│                                                         │
│   📱 多様性          様々な画面サイズ・性能に対応          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 1.3 Androidアーキテクチャ

```
┌─────────────────────────────────────────────────────────┐
│                   Applications                          │
│        (ホーム、連絡先、ブラウザ、あなたのアプリ)          │
├─────────────────────────────────────────────────────────┤
│                Application Framework                     │
│   (Activity Manager, Window Manager, Content Provider)   │
├─────────────────────────────────────────────────────────┤
│        Libraries          │      Android Runtime        │
│   (SQLite, OpenGL, SSL)   │    (ART, Core Libraries)    │
├─────────────────────────────────────────────────────────┤
│              Hardware Abstraction Layer (HAL)            │
├─────────────────────────────────────────────────────────┤
│                     Linux Kernel                         │
│      (ドライバ、電源管理、メモリ管理、セキュリティ)         │
└─────────────────────────────────────────────────────────┘
```

---

## 2. iOSとの根本的な違い

### 2.1 プラットフォームの比較

```
        iOS                              Android
    ┌─────────┐                      ┌─────────┐
    │  Apple  │ ← 1社で完結          │ Google  │ ← OSを提供
    │         │                      └────┬────┘
    │ ハード  │                           │
    │   +     │                    ┌──────┼──────┐
    │ ソフト  │                    ▼      ▼      ▼
    │   +     │               Samsung  Xiaomi  Pixel ...
    │ ストア  │                    │      │      │
    └─────────┘               独自UI  独自UI  純正
                              (One UI)(MIUI) (Stock)
```

### 2.2 詳細比較表

| 観点 | iOS | Android |
|------|-----|---------|
| **開発元** | Apple（クローズド） | Google（オープンソース） |
| **端末メーカー** | Appleのみ | 多数（Samsung, Xiaomi, OPPO等） |
| **カスタマイズ性** | 制限的 | 高い自由度 |
| **アプリ配布** | App Store のみ | Play Store + 野良APK可 |
| **開発言語** | Swift / Objective-C | Kotlin / Java |
| **IDE** | Xcode | Android Studio |
| **UI フレームワーク** | SwiftUI / UIKit | Jetpack Compose / XML |
| **端末の断片化** | 少ない | 非常に多い |
| **OSアップデート** | 一斉配信 | メーカー依存（遅延あり） |

### 2.3 「断片化」問題とは

iOSでは経験しない、Android特有の課題

```
iOS の世界                      Android の世界
─────────────                   ─────────────────────────
                               
  iPhone 14 ─┐                   Galaxy S24 (Android 14)
  iPhone 15 ─┼─▶ 同じiOS        Pixel 8 (Android 14)
  iPhone 16 ─┘                   Xperia (Android 13)      ← バージョンがバラバラ
                                 AQUOS (Android 12)
  ほぼ同じ挙動                    OPPO (Android 11)
                                 古い端末 (Android 9)
                               
                                 さらに...
                                 同じAndroid 14でも
                                 メーカーごとにUIが違う
```

**実際の市場シェア（2024年時点の例）**

```
Android バージョン別シェア
─────────────────────────────────────────────
Android 14  ████████████░░░░░░░░░░░  25%
Android 13  ██████████████████░░░░░  38%
Android 12  ████████████░░░░░░░░░░░  22%
Android 11  ████░░░░░░░░░░░░░░░░░░░   8%
それ以下    ███░░░░░░░░░░░░░░░░░░░░   7%
─────────────────────────────────────────────
※ つまり、複数バージョンへの対応が必須
```

### 2.4 開発言語の対比

**Swift と Kotlin は似ている部分が多い**

```kotlin
// iOS (Swift)                    // Android (Kotlin)
// ─────────────                  // ─────────────────

// 変数宣言
let name = "iOS"                  val name = "Android"
var count = 0                     var count = 0

// 関数
func greet(name: String) {        fun greet(name: String) {
    print("Hello, \(name)")           println("Hello, $name")
}                                 }

// Null安全
var optional: String? = nil       var nullable: String? = null
optional?.count                   nullable?.length

// クラス
class Person {                    class Person(
    let name: String                  val name: String,
    var age: Int                      var age: Int
}                                 )
```

>  **ポイント**: Swift経験者はKotlinに馴染みやすい。構文が似ている。

---

## 3. アプリ構造とライフサイクル

### 3.1 プロジェクト構造の比較

```
iOS (Xcode)                        Android (Android Studio)
─────────────                      ────────────────────────

MyApp/                             app/
├── MyApp.xcodeproj                ├── build.gradle.kts
├── Info.plist        ←────────▶   ├── src/main/
├── Assets.xcassets                │   ├── AndroidManifest.xml
├── AppDelegate.swift              │   ├── java/com/example/
├── SceneDelegate.swift            │   │   └── MainActivity.kt
├── ContentView.swift              │   └── res/
└── Models/                        │       ├── layout/
                                   │       ├── drawable/
                                   │       ├── values/
                                   │       └── mipmap/
                                   └── libs/
```

### 3.2 重要ファイルの対応関係

| iOS | Android | 役割 |
|-----|---------|------|
| Info.plist | AndroidManifest.xml | アプリの設定・権限宣言 |
| AppDelegate | Application クラス | アプリ全体のライフサイクル |
| UIViewController | Activity / Fragment | 画面の単位 |
| Storyboard / SwiftUI | XML Layout / Compose | UI定義 |
| Assets.xcassets | res/drawable, mipmap | 画像リソース |
| .xcconfig | build.gradle.kts | ビルド設定 |

### 3.3 AndroidManifest.xml の例

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">

    <!-- 権限の宣言（iOSのInfo.plistより細かい） -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" />

    <application
        android:name=".MyApplication"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.MyApp">

        <!-- 画面（Activity）の宣言 -->
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <!-- ランチャーから起動可能 -->
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity android:name=".DetailActivity" />

    </application>
</manifest>
```

### 3.4 Activity ライフサイクル（最重要）

iOSの `UIViewController` に相当しますが、より複雑

```
                    ┌─────────────┐
                    │  onCreate() │ ← 初期化（viewDidLoadに近い）
                    └──────┬──────┘
                           ▼
                    ┌─────────────┐
                    │  onStart()  │ ← 画面が見え始める
                    └──────┬──────┘
                           ▼
                    ┌─────────────┐
        ┌──────────▶│  onResume() │ ← フォアグラウンド（操作可能）
        │           └──────┬──────┘
        │                  │
        │           ユーザーが操作中...
        │                  │
        │                  ▼
        │           ┌─────────────┐
        │           │  onPause()  │ ← 一部隠れる/フォーカス失う
        │           └──────┬──────┘
        │                  │
        │    ┌─────────────┼─────────────┐
        │    │             ▼             │
        │    │      ┌─────────────┐      │
        │    │      │  onStop()   │      │ ← 完全に見えなくなる
        │    │      └──────┬──────┘      │
        │    │             │             │
        │    ▼             ▼             ▼
        │  戻る      ┌─────────────┐   システムが
        │           │ onDestroy() │   強制終了
        │           └─────────────┘
        │                  │
        └──────────────────┘
             onRestart() → onStart()
```

### 3.5 iOS との ライフサイクル対比

```
iOS (UIViewController)              Android (Activity)
──────────────────────              ──────────────────

viewDidLoad()          ─────────▶   onCreate()
viewWillAppear()       ─────────▶   onStart()
viewDidAppear()        ─────────▶   onResume()
viewWillDisappear()    ─────────▶   onPause()
viewDidDisappear()     ─────────▶   onStop()
deinit                 ─────────▶   onDestroy()

                       ※ Android の方が状態が多く複雑
```

### 3.6 Kotlin での Activity 実装例

```kotlin
class MainActivity : AppCompatActivity() {

    // onCreate: 画面の初期化（1回だけ呼ばれる）
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)  // XMLレイアウトを適用
        
        // 初期化処理
        setupUI()
        loadData()
    }

    // onResume: 画面がアクティブになるたび呼ばれる
    override fun onResume() {
        super.onResume()
        // センサー開始、アニメーション再開など
    }

    // onPause: 画面が非アクティブになる直前
    override fun onPause() {
        super.onPause()
        // センサー停止、データ一時保存など
    }

    // onDestroy: 画面が破棄される
    override fun onDestroy() {
        super.onDestroy()
        // リソース解放
    }
}
```

### 3.7 Intent（画面遷移とデータ受け渡し）

iOSの `Segue` や `NavigationController` に相当しますが、仕組みが大きく異なります。

```kotlin
// 画面遷移の基本
val intent = Intent(this, DetailActivity::class.java)
startActivity(intent)

// データを渡す場合
val intent = Intent(this, DetailActivity::class.java).apply {
    putExtra("USER_ID", 123)
    putExtra("USER_NAME", "田中太郎")
}
startActivity(intent)

// 受け取り側（DetailActivity）
class DetailActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val userId = intent.getIntExtra("USER_ID", -1)
        val userName = intent.getStringExtra("USER_NAME") ?: ""
    }
}
```

**Intent の概念図**

```
┌─────────────────┐    Intent     ┌─────────────────┐
│  MainActivity   │──────────────▶│ DetailActivity  │
│                 │   (データ付き)  │                 │
│  userId = 123   │               │  受け取って表示   │
└─────────────────┘               └─────────────────┘

Intent は「意図」を表すオブジェクト
- 明示的 Intent: 具体的なActivityを指定
- 暗黙的 Intent: 「写真を撮りたい」等の意図を指定 → OSが適切なアプリを選択
```

### 3.8 Fragment（再利用可能なUI部品）

```
┌─────────────────────────────────────────────┐
│                  Activity                    │
│  ┌─────────────────┐  ┌─────────────────┐   │
│  │   Fragment A    │  │   Fragment B    │   │
│  │   (リスト表示)   │  │   (詳細表示)    │   │
│  │                 │  │                 │   │
│  │                 │  │                 │   │
│  └─────────────────┘  └─────────────────┘   │
│                                             │
│  タブレット: 2つ同時表示                      │
└─────────────────────────────────────────────┘

┌──────────────────┐    ┌──────────────────┐
│     Activity     │    │     Activity     │
│  ┌────────────┐  │    │  ┌────────────┐  │
│  │ Fragment A │  │───▶│  │ Fragment B │  │
│  │            │  │    │  │            │  │
│  └────────────┘  │    │  └────────────┘  │
│                  │    │                  │
│  スマホ: 1つずつ   │    │                  │
└──────────────────┘    └──────────────────┘
```

---

## 4. バージョン別の歴史と現在のターゲット

### 4.1 Android バージョンの歴史（概要）

```
2008 ──── Android 1.0 (API 1)      黎明期
  │
2010 ──── Android 2.x              スマホ普及期
  │                                ※ 現在はサポート対象外
2011 ──── Android 4.0 (API 14)     タブレット統合
  │
  │       ─────────────────────────────────────
  │       ↑ ここより古いバージョンは考慮不要 ↑
  │       ─────────────────────────────────────
  │
2014 ──── Android 5.0 (API 21)     Material Design 導入
  │       Lollipop                 ART ランタイム標準化
  │                                🎯 最低サポートラインの目安
  │
2015 ──── Android 6.0 (API 23)     実行時パーミッション導入
  │       Marshmallow              ⚠️ 権限管理の大転換点
  │
2016 ──── Android 7.0 (API 24)     マルチウィンドウ
  │
2017 ──── Android 8.0 (API 26)     通知チャンネル必須化
  │
2018 ──── Android 9.0 (API 28)     ノッチ対応
  │
2019 ──── Android 10 (API 29)      ダークテーマ、Scoped Storage開始
  │                                ⚠️ ストレージアクセスの転換点
  │
2020 ──── Android 11 (API 30)      1回限りの権限、Scoped Storage強化
  │
  │       ═══════════════════════════════════════
  │       ║  ここから大きな変更（Android 12）  ║
  │       ═══════════════════════════════════════
  │
2021 ──── Android 12 (API 31)      Material You、おおよその位置情報
  │                                🎯 現在の推奨ターゲット
  │
2022 ──── Android 13 (API 33)      通知権限の追加、言語設定
  │
2023 ──── Android 14 (API 34)      写真・動画の部分アクセス
  │
2024 ──── Android 15 (API 35)      最新
  │
現在
```
#### 雑談
個人的バージョンの思い入れ  

1.0: なし。最初期は~赤乳首~トラックポイントとキーボードが付いていたらしい。titanみたいやね  
2.0: ワイがクソガキの頃に007shを使ってたのでだいぶ思い入れあるね。クルリンパしてスマホーってやってた。正直ショボいとは思えないくらい動作サクサクしてた記憶がある  
3.0: なし。影が薄いって某youtubeｒも言ってた
4.0: 202sh？に入ってたかなあ。あんまり知らない。202sh使ってた時はケータイ係長を壁紙にしていた  
4.4: くら寿司のタブレットとか、チャレンジタブレットで使われてるね。個人的に歴代OSでいっちゃんかっこいいUIだと思う。  
5.0: これ確かツレから買ったnexus5のプリインストールOSだった気がする。速攻12に上げたので覚えてない。  
6.0: 高校生の時に、塾の先生に貰ったxperia x compactに入ってたos.この端末、sonyが配布してたテーマってのを反映させることが出来て、設定画面の壁紙を初音ミクにして遊んでた記憶  
7.0: nexus7(2012)のカスromで7.0に上げて遊んでた記憶がある。2013なら12が入るらしいが、2012なので無理だった。なお当時遊んでたnexus7は劣悪なケーブルで充電したため充電端子が溶けて文鎮化した。  
8.0: ワイが初めて触ったスマホ(s3-sh)のプリインストールosだった。当時は何も知らないクソガキだったので、開発者ツールの設定項目の意味を全てググった記憶がある。s3-shだとイースターエッグの挙動がおかしかった  
9.0: s3-shのアプデを行った時に正座して1時間半ほどプログレスバーを見ていた記憶がある。このタイミングでandroid oneのロゴが変わった。google感消えてキモかったので前のロゴに戻して欲しいと思った。設定項目あんま変わらなかったのでちょっと悲しかった記憶。自分で初めて買ったスマホ(so-01k)のOSだった。  
10: s3-shのアプデを行った際に正座して1時間ほどプログレスバーを見ていた記憶がある。親が持っていたxperia 1のアプデが10でリリースされてたのに10でサポート切られててドン引きした。  
11: 個人的に一番好きなOS.UIも使い勝手も過去一ベストだと思う。一度lineageOSで触ってほしい。乗り換えたpixel5にプリインストールされていた。塾の先生の貰ったgalaxy a51に入ってたos.海外版z3にa11なcarbonROMを焼いた記憶。イースターエッグでねこあつめできる。これマジで好き  
12: pixel5でandroidベータプログラムに登録し、12に上げた。最初のプレスリリース見た時はiosやん！！！ってなってたのに、蓋開けてみたら物凄く使い勝手悪くて11に戻してくれと強く願った記憶がある。12Lで多少マシになった。12Lがあるおかげで今の中華タブが跋扈していると勝手に思っている。  
13: ベータプログラムの継続を行い、13に上げた。ちょっと変わったかな？程度の認識。~通知バーを下にぐわんってしたときに出る再生バーが精〇って一部で話題になっていた~.12turbo買った時に入ってたmiui14ベースOS.  
14: pixel5のベータプログラムが終了したため、このバージョンで安定版へ移行した。正直この辺からあんま変わってない？感ある。~通知バーを下にぐわんってしたときに出る再生バーが縦棒に修正されてて一部で話題になっていた~. 12turboちゃんをeuROMにしたときに14に上がったけど、UIがコテコテすぎて全く面影が無かった。  
15: 妹のpixel5aのリコールで7proをもらったので、15から7proをベータプログラムに登録した。イースターエッグを最初見た時、何をすべきか全くわからなかった。12turboちゃんをhyperos2に上げたら、15になったが相変わらずコテコテなので面影がない  
16: 16のベータプログラムを受け取った時はまあなんも変わらんなってイメージあったが、qpr2(だっけ？)のアプデを適用したらめっちゃUI変わった。最初見た時はキモかったが、慣れたらかなり使いやすかった。12の時にやりたかったことだろうなと感じた。イースターエッグが15と変わらなくてマジかって思った記憶


### 4.2 現在の推奨設定

```kotlin
// build.gradle.kts (app)

android {
    compileSdk = 35          // ビルドに使用するSDK（最新推奨）
    
    defaultConfig {
        minSdk = 24          // 最低サポートバージョン
                             // → Android 7.0 以上（市場の約95%カバー）
        targetSdk = 35       // 動作確認済みの最新バージョン
    }
}
```

**設定値の意味**

```
compileSdk (35)     最新のAPIを使ってビルド
     │
     ▼
targetSdk (35)      「Android 15の挙動でテスト済み」宣言
     │              新しいセキュリティ制限が適用される
     ▼
minSdk (24)         これより古い端末では動作しない
                    ※ 小さいほどカバー範囲は広いが、
                      古いAPIへの対応コストが増える
```

### 4.3 なぜ minSdk 24（Android 7.0）が現実的か

| minSdk | Android | 市場カバー率 | 考慮事項 |
|--------|---------|-------------|----------|
| 21 | 5.0 | ~99% | 古いAPIへの対応が大変 |
| 23 | 6.0 | ~98% | 実行時権限に対応必要 |
| **24** | **7.0** | **~95%** | **バランスが良い（推奨）** |
| 26 | 8.0 | ~90% | 新しめのアプリ向け |
| 29 | 10 | ~80% | Scoped Storage対応済み |

---

## 5. Android 12以降の重要な変更点

### 5.1 Android 12 (API 31) の変更点

#### Material You（ダイナミックカラー）

```
ユーザーの壁紙から色を抽出し、システム全体に適用

┌─────────────────────────────────────────────┐
│  壁紙（青い海の写真）                         │
│                                             │
│    ┌───┐                                    │
│    │ 🌊 │ ─── 色を抽出 ────┐                │
│    └───┘                   │                │
│                            ▼                │
│         ┌────────────────────────┐          │
│         │  Primary: #1A73E8     │          │
│         │  Secondary: #4285F4   │          │
│         │  Surface: #E8F0FE     │          │
│         └────────────────────────┘          │
│                    │                        │
│         ┌─────────┴─────────┐               │
│         ▼                   ▼               │
│    ボタンの色            アイコンの色        │
└─────────────────────────────────────────────┘
```

```kotlin
// ダイナミックカラーの適用
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        // Android 12以上でダイナミックカラーを適用
        DynamicColors.applyToActivitiesIfAvailable(this)
    }
}
```

#### おおよその位置情報

```
Android 11以前                  Android 12以降
─────────────────              ─────────────────────────

位置情報の許可                  位置情報の許可
    │                              │
    ▼                              ▼
┌─────────┐                   ┌─────────────┐
│ 許可する │                   │ 正確な位置   │  ← 新規追加
└─────────┘                   ├─────────────┤
                              │ おおよそ     │  ← 新規追加
                              ├─────────────┤
                              │ 許可しない   │
                              └─────────────┘

※ ユーザーが「おおよそ」を選ぶと、
   約1.6km程度の精度しか得られない
```

```kotlin
// 位置情報の権限リクエスト（Android 12対応）
private val locationPermissions = if (Build.VERSION.SDK_INT >= 31) {
    arrayOf(
        Manifest.permission.ACCESS_FINE_LOCATION,
        Manifest.permission.ACCESS_COARSE_LOCATION
    )
} else {
    arrayOf(Manifest.permission.ACCESS_FINE_LOCATION)
}
```

#### SplashScreen API の強制

```
Android 11以前                  Android 12以降
─────────────────              ─────────────────────────

自由にスプラッシュを             システム標準のスプラッシュが
実装可能                        自動で表示される
                               
┌─────────────┐                ┌─────────────┐
│             │                │     ┌─┐     │
│    LOGO     │                │     │●│     │  ← アイコンのみ
│             │                │     └─┘     │
│  Loading... │                │             │
└─────────────┘                └─────────────┘
                               
                               ※ カスタマイズは可能だが
                                 制約がある
```

### 5.2 Android 13 (API 33) の変更点

#### 通知の実行時権限

```
Android 12以前                  Android 13以降
─────────────────              ─────────────────────────

通知は                         通知を送る前に
デフォルトで許可                許可を求める必要あり

アプリ起動                     アプリ起動
    │                              │
    ▼                              ▼
すぐ通知OK                     ┌─────────────────────┐
                              │ 通知を許可しますか？  │
                              │  [許可] [許可しない] │
                              └─────────────────────┘
                                       │
                                       ▼
                              許可されたら通知OK
```

```kotlin
// 通知権限のリクエスト（Android 13以降で必要）
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
    if (checkSelfPermission(Manifest.permission.POST_NOTIFICATIONS) 
            != PackageManager.PERMISSION_GRANTED) {
        requestPermissions(
            arrayOf(Manifest.permission.POST_NOTIFICATIONS),
            NOTIFICATION_PERMISSION_CODE
        )
    }
}
```

### 5.3 Android 14 (API 34) の変更点

#### 写真/動画の部分的なアクセス

```
Android 13以前                  Android 14以降
─────────────────              ─────────────────────────

すべての写真に                  ┌───────────────────────┐
アクセス許可                    │  アクセス許可の選択     │
  or                          │                       │
アクセス拒否                    │  ○ すべての写真       │
                              │  ○ 選択した写真のみ    │  ← 新規
                              │  ○ 許可しない         │
                              └───────────────────────┘
                              
                              ユーザーが特定の写真だけを
                              選んで許可できる
```

### 5.4 バージョン別対応のパターン

```kotlin
// バージョンによって処理を分岐する基本パターン
when {
    Build.VERSION.SDK_INT >= Build.VERSION_CODES.UPSIDE_DOWN_CAKE -> {
        // Android 14 (API 34) 以上の処理
    }
    Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU -> {
        // Android 13 (API 33) 以上の処理
    }
    Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
        // Android 12 (API 31) 以上の処理
    }
    else -> {
        // Android 11 以下の処理
    }
}
```

---

## 6. Android特有の「癖」と落とし穴

### 6.1 パーミッション（権限）の複雑さ

#### iOS との比較

```
iOS                              Android
─────                            ───────

初回アクセス時に                  2段階の仕組み
1回だけ確認                      
                                1. Manifest で宣言
┌────────────────┐                 （宣言しないと絶対に使えない）
│ 位置情報を      │              
│ 許可しますか？  │              2. 実行時に許可を求める
└────────────────┘                 （Android 6.0以降）

                                ┌─────────────────────────┐
                                │ AndroidManifest.xml     │
                                │ <uses-permission .../>  │
                                └───────────┬─────────────┘
                                            │
                                            ▼
                                ┌─────────────────────────┐
                                │ 実行時にダイアログ表示   │
                                └─────────────────────────┘
```

#### 権限の種類

```
┌─────────────────────────────────────────────────────────┐
│                    権限の分類                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🟢 Normal（通常）                                      │
│     └─ インストール時に自動付与                          │
│     └─ 例: INTERNET, BLUETOOTH                          │
│                                                         │
│  🟡 Dangerous（危険）                                   │
│     └─ ユーザーの明示的な許可が必要                      │
│     └─ 例: CAMERA, LOCATION, CONTACTS                   │
│                                                         │
│  🔴 Special（特殊）                                     │
│     └─ 設定画面からの手動許可が必要                      │
│     └─ 例: SYSTEM_ALERT_WINDOW（オーバーレイ）          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### 実装例

```kotlin
class MainActivity : AppCompatActivity() {

    private val cameraPermissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted ->
        if (isGranted) {
            // 許可された → カメラを起動
            openCamera()
        } else {
            // 拒否された → 説明を表示
            showPermissionRationale()
        }
    }

    private fun requestCameraPermission() {
        when {
            // すでに許可済み
            checkSelfPermission(Manifest.permission.CAMERA) 
                == PackageManager.PERMISSION_GRANTED -> {
                openCamera()
            }
            // 以前拒否された → 説明が必要
            shouldShowRequestPermissionRationale(Manifest.permission.CAMERA) -> {
                showRationaleDialog()
            }
            // 初回 or 「今後表示しない」選択後
            else -> {
                cameraPermissionLauncher.launch(Manifest.permission.CAMERA)
            }
        }
    }
}
```

### 6.2 Scoped Storage（ストレージアクセス制限）

#### 変遷

```
Android 9以前     Android 10        Android 11以降
────────────     ──────────        ─────────────────

自由にアクセス    オプトアウト可能    完全強制
                 (一時的に回避可)    

/storage/        
├── DCIM/        ← 読み書き自由     ← 条件付き      ← メディアAPIのみ
├── Download/    ← 読み書き自由     ← 条件付き      ← SAFのみ
├── Music/       ← 読み書き自由     ← 条件付き      ← メディアAPIのみ
└── MyApp/       ← 自由             ← 自由          ← 自由（アプリ専用）
```

#### 対応方法

```kotlin
// ❌ 古い方法（Android 10以降では動作しない）
val file = File("/storage/emulated/0/Download/test.txt")
file.writeText("Hello")

// ✅ 正しい方法1: アプリ専用領域を使う
val file = File(context.filesDir, "test.txt")
file.writeText("Hello")

// ✅ 正しい方法2: MediaStore API（写真・動画・音楽）
val contentValues = ContentValues().apply {
    put(MediaStore.Images.Media.DISPLAY_NAME, "my_image.jpg")
    put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")
}
val uri = contentResolver.insert(
    MediaStore.Images.Media.EXTERNAL_CONTENT_URI, 
    contentValues
)

// ✅ 正しい方法3: Storage Access Framework（ユーザーが選択）
val intent = Intent(Intent.ACTION_CREATE_DOCUMENT).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TITLE, "document.txt")
}
startActivityForResult(intent, CREATE_FILE_REQUEST)
```

### 6.3 バックグラウンド制限

#### iOS との違い

```
iOS                              Android
─────                            ───────

バックグラウンド                  複数の制限レイヤー
モードで比較的                   
安定して動作                      ┌─────────────────────────┐
                                 │ Doze モード              │
                                 │ (画面OFF + 静止で発動)   │
                                 └────────────┬────────────┘
                                              ▼
                                 ┌─────────────────────────┐
                                 │ App Standby             │
                                 │ (使われないアプリを制限) │
                                 └────────────┬────────────┘
                                              ▼
                                 ┌─────────────────────────┐
                                 │ メーカー独自の制限       │
                                 │ (Samsung, Xiaomi等)     │
                                 └─────────────────────────┘
                                 
                                 ※ 通知が届かない、同期しない
                                   などの問題が発生しやすい
```

### 6.4 メーカーカスタマイズの罠

```
Google Pixel（素のAndroid）で動作確認
              │
              ▼
         動作OK！ ✅
              │
              │   でも...
              ▼
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Samsung (One UI)     → タスクキラーが強力              │
│                         バックグラウンド処理が止まる     │
│                                                         │
│  Xiaomi (MIUI)        → 自動起動の許可が必要            │
│                         バッテリー最適化から除外必要     │
│                                                         │
│  OPPO (ColorOS)       → 独自の省電力機能                │
│                         アプリのロックが必要            │
│                                                         │
│  Huawei (EMUI)        → 独自の権限管理                  │
│                         Protected Apps への登録必要     │
│                                                         │
└─────────────────────────────────────────────────────────┘

⚠️ 「Pixelで動いたから大丈夫」は危険！
   主要メーカーの実機でテストすること
```

### 6.5 dp / sp と画面サイズ

#### 単位の違い

```
iOS                              Android
─────                            ───────

pt (ポイント)                    dp (density-independent pixel)
  └─ 解像度非依存                  └─ 解像度非依存

                                 sp (scale-independent pixel)
                                   └─ フォントサイズ専用
                                      ユーザーの設定で拡大縮小
```

#### 画面密度の多様性

```
画面密度 (dpi)     倍率      代表的な端末
───────────────────────────────────────────
ldpi    (~120)    0.75x     ほぼ絶滅
mdpi    (~160)    1.0x      古い端末
hdpi    (~240)    1.5x      古い端末
xhdpi   (~320)    2.0x      多くのスマホ
xxhdpi  (~480)    3.0x      高解像度スマホ
xxxhdpi (~640)    4.0x      最新フラッグシップ

※ iOSは @1x, @2x, @3x の3種類だが
  Androidはより細かい区分
```

```kotlin
// レイアウトでの使用例
// ✅ 正しい
android:layout_width="200dp"
android:textSize="16sp"

// ❌ 間違い（画面サイズによって見た目が変わる）
android:layout_width="200px"
android:textSize="16px"
```

---

## 7. 配信とテスト

### 7.1 ビルド成果物の比較

```
iOS                              Android
─────                            ───────

.ipa                             .apk (従来)
 └─ 1ファイル                     └─ 全リソース含む
                                   └─ サイズが大きくなりがち
                                 
                                 .aab (推奨)
                                   └─ Android App Bundle
                                   └─ Play Storeが端末に
                                      最適化して配信
```

### 7.2 配布方法の違い

```
┌─────────────────────────────────────────────────────────┐
│                      iOS                                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│    App Store ─────── 唯一の公式配布手段                  │
│         │                                               │
│         ├─ TestFlight (テスト配布)                      │
│         │                                               │
│         └─ Ad Hoc / Enterprise (限定的)                 │
│                                                         │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    Android                              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│    Google Play ────── 主要な配布手段                    │
│         │                                               │
│         ├─ 内部テスト (最大100人)                       │
│         ├─ クローズドテスト (人数制限なし)               │
│         └─ オープンテスト (誰でも参加)                   │
│                                                         │
│    その他の方法 ──── Androidの自由度                    │
│         │                                               │
│         ├─ APKを直接インストール（サイドローディング）   │
│         ├─ 自社サーバーから配布                         │
│         ├─ Amazon App Store                            │
│         └─ その他のストア                               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 7.3 署名の仕組み

```
iOS                              Android
─────                            ───────

証明書 + Provisioning           Keystore ファイル
Profile の複雑な管理             (.jks または .keystore)

Apple Developer                 ┌─────────────────────┐
Program 必須                    │  keystore.jks       │
                                │  ├─ 秘密鍵          │
                                │  └─ 証明書          │
                                └─────────────────────┘
                                
                                ⚠️ 紛失すると
                                   アプリの更新不可能！
```

```kotlin
// build.gradle.kts での署名設定
android {
    signingConfigs {
        create("release") {
            storeFile = file("my-release-key.jks")
            storePassword = System.getenv("KEYSTORE_PASSWORD")
            keyAlias = "my-key-alias"
            keyPassword = System.getenv("KEY_PASSWORD")
        }
    }
    
    buildTypes {
        getByName("release") {
            signingConfig = signingConfigs.getByName("release")
            isMinifyEnabled = true
        }
    }
}
```

### 7.4 テスト時の注意点

```
┌─────────────────────────────────────────────────────────┐
│              テストで確認すべきこと                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  📱 複数の画面サイズ                                     │
│     └─ 小型スマホ、大型スマホ、タブレット                │
│                                                         │
│  🤖 複数のAndroidバージョン                              │
│     └─ minSdk、中間、最新の3パターン以上                 │
│                                                         │
│  🏭 複数のメーカー                                       │
│     └─ Pixel（基準）、Samsung、Xiaomi 等                 │
│                                                         │
│  🔋 バッテリー最適化の影響                               │
│     └─ バックグラウンド処理、通知、位置情報               │
│                                                         │
│  🌐 ネットワーク状態                                     │
│     └─ オフライン、低速回線                              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 7.5 エミュレータ vs 実機

```
エミュレータ                     実機
────────────                     ────

✅ 複数バージョンを              ✅ 実際の性能
   簡単に切り替え                
                                ✅ メーカー独自機能の
✅ 画面サイズを自由に               テスト
   設定可能                      
                                ✅ センサー類
❌ パフォーマンスは                 （GPS、カメラ、加速度）
   参考にならない                
                                ✅ バッテリー挙動
❌ メーカー独自UIは              
   再現できない                  ❌ 全パターン揃えるのは
                                   コストがかかる
❌ 一部センサーは                
   シミュレートのみ              

💡 結論: 両方使い分ける
   開発中 → エミュレータ
   最終確認 → 実機（主要メーカー複数）
```

---

## まとめ

### iOS開発者がAndroidで戸惑うポイント

| ポイント | iOS | Android |
|----------|-----|---------|
| 端末の多様性 | 少ない | 非常に多い（断片化） |
| OS更新 | 一斉配信 | メーカー依存で遅延 |
| 権限管理 | シンプル | 複雑（Manifest + 実行時） |
| ファイルアクセス | 比較的自由 | 制限厳しい（Scoped Storage） |
| バックグラウンド | 安定 | 制限多い（Doze等） |
| アプリ配布 | App Store のみ | 複数の選択肢 |
| メーカー差異 | なし | 大きい（UI、省電力等） |

### 最初に覚えるべきこと

1. **Activity ライフサイクル** — すべての基本
2. **実行時パーミッション** — ユーザー体験に直結
3. **バージョン分岐** — `Build.VERSION.SDK_INT` の使い方
4. **複数端末でのテスト** — 「Pixelで動く」だけでは不十分

### 参考リンク

- [Android Developers 公式](https://developer.android.com/)
- [Kotlin 公式ドキュメント](https://kotlinlang.org/docs/home.html)
- [Material Design 3](https://m3.material.io/)

---

## あとがき
### 個人的に泥が好きなところ
なんと言ってもやっぱり自由！！！！！！広告ブロックさせてくれるわゲームのアセットぶっこ抜けるわ環境をそっくりそのまま別端末に移せるわ……  
こう…アンダーグラウンドな人間にとってとても都合がよろしいんですよね。例えばエミュの開発、配布がしやすかったりpcライクに使えたり(頑張ればgitが使える)  

まあ使っている人全員悪人とかアンダーグラウンドな知識あるわけじゃないですけどね。ワイみたいなキモオタなんてごく一部ですよ。~OB会の時にOBに対して「そのスマホ、もしかしてGalaxy S23 Ultraですか？」って聞いてなんだこいつとか言われた~  

### 気をつけてほしいところ
なんと言ってもOSが端末によって全然ちゃうとこですね。

Pixel,Sumsung,Xperia,Xiaomi,Oppo,エミュ…全部ちゃいますからね。大手企業ですら実機用意して実機デバッグしてるらしいので実機動作確認ちゃんとしてほしいですわ。  
25の大会2週間前くらいに突然、両方のRedmi Note11だけ機体との接続にグズりやがったこともあります。Pixel6a,7aは接続エラーなかったのに。あの時は絶対HyperOSなんて使うかと思いましたわ。  
慢心なんてしちゃダメですよ。アプリ動く上での通信で絶対とかないんで。信じていいのjetpack composeで構成された画面の表示だけですよ  

#### HyperOSについて
hyperosはｶﾅｰﾘ厄介なのでメモっておきます。  

hyperosは、MIUIという名前のOSのバージョンアップ版です。ググればわかりますが、かなりUIが変わってます。要はiOSらしくなったということですね。あとめっちゃ重くなった。しょぼいSoC積んでるスマホだったら火吹きますよ。Redmi Note 11も火吹いてますよ。  

アカンところは、すぐタスクキルするところです。要するにすぐアプリ終了させて接続切りやがるということです。うんちです。ワイが嫌いなとこNo.1です。こいつのせいでadguardのタスクどれだけ切られたか。しばき回しますよほんと  
こればっかりはどうしようもないので、大会で使用するのはおすすめしません。使うとしてもsrcくらいかなー。

良いところとしては、Bootloader unlockさせてくれるところでしょうか。まあこれも実質改悪されてるんで出来ないようなもんなんですけどね。  
↑毎日日本時間午前1時にbootloader unlockさせれる枠が開くので、全世界で取り合いです。ゴミマジで。HyperOSなんてうんこOS消させてくれ

#### Pixelについて
個人的にｶﾅｰﾘ好きな部類のOSなのでメモっておきます。
ロボプロに置いてあるやつは、Pixel6a(カメラバーが細くて黒いやつ)と、Pixel7a(カメラバーが太くてシルバーっぽいやつ)の2機種3台です。  

この娘たちの何が良いってマジで開発者に優しいところです。ワイが惚れた理由を以下に列挙します。
 - bootloader unlockさせてくれる
 - androidの最新プログラムをいち早く受け取れる
 - ~リコールが多い~
 - 国内代理修理店が多い
 - パーツのモジュール化がちゃんとしてる
     - 画面割れても、内カメ生きてたり顔認証生きてたりマウス操作できたり...
     - 中バラして指紋だけぶっこ抜いて起動すると、ちゃんと(?)指紋認証設定項目だけ消えた状態で立ち上がる
 - 最悪自分で直せる
     - アリエクでパーツが安く売ってる
     - 深圳で中古パーツで組まれたジャンクpixelが売ってるらしい
 - カメラが安い割にキレイ
 - 廉価モデルでもそれなりのSoC積んでる
 - logcatに素直なログを出してくれる
     - HyperOSは見習えよマジ。画面に指置いた瞬間わけわからんログ出すなks
     - ~xiaomi.eu romお前もやぞ。おま国しやがって調子乗るな~
 - OS吹き飛ばしても、ファクトリーイメージを公開してくれてる
 - ファクトリーイメージを**PCのchromeから**インスコできる
     - (しゅきしゅきだいしゅき〜っ)
     - https://flash.android.com/welcome?continue=%2F%3Fhl%3Dja
 - 基本的にlineageOSが公開されてるので、どんだけ古いOSでも最新OSを**動かす**ことはできる
     - ↑動くとは言ってない

まあこんなもんですかね。言い換えれば、ユーザーフレンドリーではないというOSですね。わかってる人向きでバニラな使い勝手です。  
最近、レガシーな端末(Pixel7以前)のサポートが雑になってアプデQPRごとになった(3ヶ月に1回)のでうんちですね。  
あとレガシーな端末？のAOSP公開サイクルが遅れがちになっててカスROM開発者・ユーザーも地味に影響受けております。googleなんとかしてくれ　　

#### bootloader unlockについて
android弄ってるならbluくらいすると面白いと思いますよ。  
ちゃんとした説明は[あっち](https://source.android.com/docs/core/architecture/bootloader/locking_unlocking?hl=ja)に任せるとして、こっちでは雑に解説しようと思います。  

##### なにが出来るか
メリットを以下に列挙します

- OSを書き換えられる
    - 頑張れば自分で作成できる
    - 有志が作ってくれたOSを入れることができ、なんちゃって顔認証などを入れたりセキュリティ強化したOSを入れたりできる
    - bluするのは実質これのため
- アプリ改変せずに広告ブロックがちゃんとできる  
    - 例: ようつべのアプリに手を入れずに広告ブロックできる
- 電話の録音ができる
    - これは最近のPixelでもできるようになっている
- ゲームのデータをぶっこ抜ける
    - セーブデータ、アセット、通信パケット全て丸見えですわ
- ゲーム(unity)の品質を上げることができる
    - 上限60fpsから120fpsに上げることもできる
    - ~ｺﾞﾆｮｺﾞﾆｮしたら学マスキャラのπ乙を盛ることもできる。banされたら怖いのでワイはやらんが~

まー一旦こんなもんです。OSを書き換えられるってのがいっちゃんデカい。  
有名なOSとして、lineageOS,EvolutionX,crDroid,xiaomi.euがあります。   
とりまlineage入れときゃバニラな使い勝手を感じられると思います。個人的な推しはcrDroidです。設定から端末偽装できてgoogleフォトに無制限アップロード出来て良い。evoxもcrdroidとほぼ同じだけど、なんとなく不安定だったので嫌い。  
xiaomi.euは、miuiだったりhyperosのカスいところ(プリインストールアプリなのに広告出てくる,中国の怪しいサーバーへの情報送信の停止)が行われて、かつUIはそのままって感じのちゃんとしたOSです。xiaomiスマホ買ったらbluしてみてほしい。けど最近日本からxiaomi.euサイトへアクセスできなくなったのでカスです。~早苗ちゃんなんとかしてくれ~
 
OSを書き換えるということは、本来であればandroid11で止まっていたバージョンを無理やりandroid12,13...へ上げられるということです。  
レガシーなOSだと、セキュリティアップデートが来なかったり、サポート終了されるアプリが増えて物理的に動かなくなります。けど、バージョン上げたOSを焼くだけで、それらのアプリが動くようになると。まだまだレガシーな端末使えるとか最高じゃないですか。ワイはこのロマンが好きでandroid使ってる感ある。  

これのデメリットとして、以下の要素があります。
- bootloader unlockしたら無条件にデータが初期化される
- OSを焼き直すときもデータが初期化される
- 焼き方ミスったら文鎮化する
    - ↑これいっちゃんデメリット
- 銀行系アプリが使えんなってきている
    - ↑これいっちゃんデメリット2
- 色々グレー
    - 電波法、端末によっては技適諸々
    - ~法に触れているからって摘発されることはないが~

まあこんなもんです。  
特にヤバいやつだけ解説しておきます。  
blu,os焼き直しでデータ初期化はセキュリティ面のアレなのでそういうものです。諦めてください。  
焼き方ミスったら文鎮化するのは、そういうものなんで諦めてください。  
[ここらへん](https://mitanyan98.hatenablog.com/)見とけばとりあえず大丈夫だと思います。最悪この人らのディスコ鯖凸ればなんとかなります。1台スマホを潰してからが本番ですよ  

銀行系アプリが使えんなるのは、主に[ゴミカスアホボケ詩ねクソPlay Integrity API](https://developer.android.com/google/play/integrity/overview?hl=ja)のせいです。こいつを呪ってください。  
android10,11までのOSってSafetyNetってのが入ってたんですけど、めっちゃセキュリティザルでそれはそれは何でもできたわけです。android12に上げるタイミングで問題視されて、セキュリティ上げないとアカンやん！ってなりました。android13~14あたりで実装されて、カスrom入れたワイらは毎日涙をちょちょぎれています。  
こいつがいるとどうなるかというと、まずエミュ等変なosか、osが改変されているか見て、bluされているか見て、通った端末のみbasic,device,strongのラベルが貼られます。  
![Screenshot_2026-01-17-12-20-45-343_gr.nikolasspyr.integritycheck.jpg](/attachment/6978632b27c46924a4088d71)  
↑こんな感じ  
 
大体銀行系のアプリはそのラベルを見て、strongまで通ってないと動かないです。  
basicは割と簡単に取れて、deviceはデバイスを偽装したら通ります。strongは弄ってたらもう無理です。googleがうんちすぎて取れない。  
![Screenshot_2026-01-16-16-42-44-575_com.revolut.revolut.jpg](/attachment/6978635627c46924a4088dc9)  
↑マジカス氏ね  

これの何がアカンって、googleのサーバーで判定するとこなんですよ。なんでぽみゃーらに申請してやらんといけんねんぽみゃーらにやる個人情報なんてねえよ！！！ってブチ切れている人がごく一部に存在しています。  

playintegrityに通ってるかどうかは、以下の方法で確認できます。
1. プロフィールアイコン→設定→基本情報 を開き、Play ストアのバージョン の行を7回タップして、開発者向けオプションを有効化する。
2. プロフィールアイコン→設定→全般→開発者向けオプション を開き、Play Integrity の 完全性の確認 を押すと判定できる

まあそんなところです。色々とgoogle対有志のバトルがあるわけですが、最近はgoogleがゴリ押そうとしててアカンですね。邪悪になるなって教わらなかったんでしょうか。  

まあこれ見て興味持った変な人は、メルカリとかラクマとかイオシスとかワーモバとかでスマホ買って触ってみると良いと思います。端末によって個性があって面白いんです。  
androidはね、設定が一番個性出ているんですよ。誰に対して、何を触ってほしいか、何を弄られてほしくないか考えながら見るとなかなか面白いです。ぜひやってみてほしい。  

