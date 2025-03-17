---
title: "1章 シンプルなC++パブリッシャーとサブスクライバーアプリケーションの作成"
draft: false
featured: false
categories: ["FastDDS_Manual"]
tags: [1章]
summary: ""
---
## 1.3 シンプルなC++パブリッシャーとサブスクライバーアプリケーションの作成

このセクションでは、C++APIを使用してシンプルなFastDDSアプリケーションをパブリッシャーとサブスクライバーで作成する方法を段階的に説明する。また、eProsima Fast DDS-Genツールを用いて、このセクションで実装されているものと同様の例を自動生成することも可能である。
この追加のアプローチは、「パブリッシュ/サブスクライバーアプリケーションの構築」で説明されている。

### 1.3.1 背景

DDSは、DCPSモデルを実装するデータ中心の通信ミドルウェアである。このモデルは、データを生成する要素であるパブリッシャーと、データを消費するサブスクライバーの開発に基づいている。
これらのエンティティは、トピックを介して通信する。トピックは両方のDDSエンティティを結びつける要素である。パブリッシャーはトピックの下で情報を生成し、サブスクライバーは同じトピックをサブスクライブして情報を受け取る。

### 1.3.2 前提条件

まず、eProsima FastDDSとそのすべての依存関係をインストールするために、インストールマニュアルに記載されている手順に従う必要がある。また、eProsima FastDDS-Genツールのインストールについても、インストールマニュアルに記載されている手順を完了している必要がある。
このチュートリアルで提供されている全てのコマンドは、Linux環境用に記述されている。

### 1.3.3 アプリケーションワークスペースの作成

プロジェクトの最後には、アプリケーションワークスペースは以下の構造を持つことになる。
``build/DDSHelloWorldPublisher ``と ``build/DDSHelloWorldSubscriber``は、それぞれパブリッシャーとサブスクライバーアプリケーションである。

```
.
└── workspace_DDSHelloWorld
    ├── build
    │   ├── CMakeCache.txt
    │   ├── CMakeFiles
    │   ├── cmake_install.cmake
    │   ├── DDSHelloWorldPublisher
    │   ├── DDSHelloWorldSubscriber
    │   └── Makefile
    ├── CMakeLists.txt
    └── src
        ├── HelloWorld.hpp
        ├── HelloWorld.idl
        ├── HelloWorldCdrAux.hpp
        ├── HelloWorldCdrAux.ipp
        ├── HelloWorldPublisher.cpp
        ├── HelloWorldPubSubTypes.cxx
        ├── HelloWorldPubSubTypes.h
        ├── HelloWorldSubscriber.cpp
        ├── HelloWorldTypeObjectSupport.cxx
        └── HelloWorldTypeObjectSupport.hpp
```

まず、ディレクトリツリーを作成する。

```
mkdir workspace_DDSHelloWorld && cd workspace_DDSHelloWorld
mkdir src build
```

### 1.3.4 リンクされたライブラリとその依存関係のインポート

DDSアプリケーションにはFastDDSとFastCDRライブラリが必要となる。
インストール手順によって、これらのライブラリをDDSアプリケーションで利用可能にするプロセスが若干異なる。

#### 1.3.4.1 バイナリからのインストールと手動インストール

バイナリからのインストールまたは手動インストールを行った場合、これらのライブラリはすでにワークスペースからアクセス可能である。
Linuxでは、ヘッダーファイルはFastDDSとFastCDRについてそれぞれ ``/usr/include/fastdds/``と ``/usr/include/fastcdr/``ディレクトリにある。
両方のコンパイル済みライブラリは ``/usr/lib/``ディレクトリにある。

#### 1.3.4.2 Colconインストール

Colconインストールからライブラリをインポートする方法はいくつかある。
ライブラリを現在のセッションでのみ利用可能にする必要がある場合は、次のコマンドを実行する。

```
source <path/to/Fast-DDS/workspace>/install/setup.bash

```

シェル設定ファイルにFastDDSインストールディレクトリを$PATH変数に追加することで、任意のセッションからアクセス可能にすることができる。現在のユーザーに対して以下のコマンドを実行する。

```
echo 'source <path/to/Fast-DDS/workspace>/install/setup.bash' >> ~/.bashrc

```

これにより、このユーザーのログイン後毎回環境が設定される。

### 1.3.5 Cmakeプロジェクトの設定

プロジェクトのビルドを管理するためにCmakeツールを使用する。お好みのテキストエディタで、CMakeLists.txtという新しいファイルを作成し、以下のコードスニペットをコピーアンドペーストする。
このファイルをワークスペースのルートディレクトリに保存する。
これらの手順に従った場合、workspace_DDSHelloWorldになるはずである。

```
cmake_minimum_required(VERSION 3.20)

project(DDSHelloWorld)

# Find requirements
if(NOT fastcdr_FOUND)
    find_package(fastcdr 2 REQUIRED)
endif()

if(NOT fastdds_FOUND)
    find_package(fastdds 3 REQUIRED)
endif()

# Set C++11
include(CheckCXXCompilerFlag)
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_COMPILER_IS_CLANG OR
        CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    check_cxx_compiler_flag(-std=c++11 SUPPORTS_CXX11)
    if(SUPPORTS_CXX11)
        add_compile_options(-std=c++11)
    else()
        message(FATAL_ERROR "Compiler doesn't support C++11")
    endif()
endif()

message(STATUS "Configuring HelloWorld publisher/subscriber example...")
file(GLOB DDS_HELLOWORLD_SOURCES_CXX "src/*.cxx")

```

各セクションで、特定の生成されたファイルを含めるためにこのファイルを完成させていく。

### 1.3.6 トピックデータ型の構築

eProsima Fast DDS-Genは、Javaアプリケーションで、インターフェース記述言語(IDL)ファイルで定義されたデータ型を使用してソースコードを生成する。このアプリケーションは2つの異なることを行うことができる:

- カスタムトピックのC++定義を生成
- トピックデータを使用する機能的な例を生成

このチュートリアルでは前者を使用する。
このプロジェクトでは、Fast DDS-Genアプリケーションを使用して、パブリッシャーが送信し、サブスクライバーが受信するメッセージのデータ型を定義する。

ワークスペースディレクトリで、以下のコマンドを実行する。

```
cd src && touch HelloWorld.idl

```

これにより、srcディレクトリに ``HelloWorld.idl``ファイルが生成される。テキストエディタでファイルを開き、以下のコードスニペットをコピーアンドペーストする。

```
struct HelloWorld
{
    unsigned long index;
    string message;
};

```

これにより、HelloWorldデータ型が定義された。これには2つの要素がある。

- unit32_t型のindex
- std:string型のmessage
  後は、このデータ型をc++11で実装するソースコードを生成するだけである。
  srcディレクトリから以下のコマンドを実行する。

```
<path/to/Fast DDS-Gen>/scripts/fastddsgen HelloWorld.idl

```

これにより、以下のファイルが生成されているはずである:

- HelloWorld.hpp:HelloWorld型の定義
- HelloWorldPubSubTypes.cxx:HelloWorldPubSubTypes.cxxのヘッダーファイル
- HelloWorldCdrAux.ipp:HelloWorld型のシリアライゼーションおよびデシリアライゼーションコード
- HelloWorldCdrAux.hpp:HelloWorldCdrAux.ippのヘッダーファイル
- HelloWorldTypesObjectSupport.cxx:TypeObject表現登録コード
- HelloWorldTypeObjectSupport.hpp:HelloWorldTypeObjectSupport.cxxのヘッダーファイル

### 1.3.7 FastDDSパブリッシャーの作成
ワークスペースのsrcディレクトリから、以下のコマンドを実行してHelloWorldPublisher.cppファイルをダウンロードする。
```
wget -O HelloWorldPublisher.cpp \
    https://raw.githubusercontent.com/eProsima/Fast-RTPS-docs/master/code/Examples/C++/DDSHelloWorld/src/HelloWorldPublisher.cpp

```
これはパブリッシャーアプリケーションのc++ソースコードである。HelloWorldTopicトピックの下で10回の公開を行う。

#### 1.3.7.1 ソースコードの検証
ファイルの暴徒には、@fileフィールドを持つDoxygen形式のコメントブロックがあり、ファイル名が記述されている。
```
/**
 * @file HelloWorldPublisher.cpp
 *
 */

```
その下にはc++ヘッダーのインクルードがある。最初のものは、前のセクションで定義したデータ型のシリアライゼーションおよびデシリアライゼーション関数を含むHelloWorldPubSubTypes.hファイルをインクルードしている。
```
#include "HelloWorldPubSubTypes.hpp"
```

次のブロックは、Fast DDS APIの使用を可能にするC++ヘッダーファイルをインクルードしている。

- DomainParticipantFactory: DomainParticipantオブジェクトの作成と破棄を可能にする。
- DomainParticipant: 他のすべてのEntityオブジェクトのコンテナとして機能し、Publisher、Subscriber、およびTopicオブジェクトのファクトリとして機能する。
- TypeSupport: 参加者に特定のデータ型のシリアライゼーション、デシリアライゼーション、およびキーを取得する関数を提供する。
- Publisher: DataWriterの作成を担当するオブジェクト。
- DataWriter: アプリケーションが特定のトピックの下で公開されるデータの値を設定することを可能にする。
- DataWriterListener: DataWriterListenerの関数の再定義を可能にする。

```
#include <chrono>
#include <thread>

#include <fastdds/dds/domain/DomainParticipant.hpp>
#include <fastdds/dds/domain/DomainParticipantFactory.hpp>
#include <fastdds/dds/publisher/DataWriter.hpp>
#include <fastdds/dds/publisher/DataWriterListener.hpp>
#include <fastdds/dds/publisher/Publisher.hpp>
#include <fastdds/dds/topic/TypeSupport.hpp>

```
次に、アプリケーションで使用するeProsima FastDDSのクラスと関数を含む名前空間を定義する。
```
using namespace eprosima::fastdds::dds;

```
次の行は、パブリッシャーを実装するHelloWorldPublisherクラスを作成する。
```
class HelloWorldPublisher

```
クラスのプライベートデータメンバーに続いて、hello_データメンバーはIDLファイルで作成したデータ型を定義するHelloWorldクラスのオブジェクトとして定義される。  
次に参加者、パブリッシャー、トピック、DataWriter、およびデータ型をDomainParticipantに登録されるオブジェクトである。
```
private:

    HelloWorld hello_;

    DomainParticipant* participant_;

    Publisher* publisher_;

    Topic* topic_;

    DataWriter* writer_;

    TypeSupport type_;
```

次に、DataWriterListenerクラスを継承してPubListenerクラスが定義される。  
このクラスはデフォルトのDataWriterリスナーコールバックをオーバーライドし、イベントが発生した場合にルーチンを実行することを可能にする。  
オーバーライ