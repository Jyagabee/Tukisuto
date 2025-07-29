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

これはパブリッシャーアプリケーションのc++ソースコードである。HelloWorldTopicトピックで10件のパブリケーションを送信する。

```
// Copyright 2016 Proyectos y Sistemas de Mantenimiento SL (eProsima).

//

// Licensed under the Apache License, Version 2.0 (the "License");

// you may not use this file except in compliance with the License.

// You may obtain a copy of the License at

//

//     http://www.apache.org/licenses/LICENSE-2.0

//

// Unless required by applicable law or agreed to in writing, software

// distributed under the License is distributed on an "AS IS" BASIS,

// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

// See the License for the specific language governing permissions and

// limitations under the License.


/**

 * @file HelloWorldPublisher.cpp

 *

 */


#include "HelloWorldPubSubTypes.hpp"


#include <chrono>

#include <thread>


#include <fastdds/dds/domain/DomainParticipant.hpp>

#include <fastdds/dds/domain/DomainParticipantFactory.hpp>

#include <fastdds/dds/publisher/DataWriter.hpp>

#include <fastdds/dds/publisher/DataWriterListener.hpp>

#include <fastdds/dds/publisher/Publisher.hpp>

#include <fastdds/dds/topic/TypeSupport.hpp>


using namespace eprosima::fastdds::dds;


class HelloWorldPublisher

{

private:


    HelloWorld hello_;


    DomainParticipant* participant_;


    Publisher* publisher_;


    Topic* topic_;


    DataWriter* writer_;


    TypeSupport type_;


    class PubListener : public DataWriterListener

    {

    public:


        PubListener()

            : matched_(0)

        {

        }


        ~PubListener() override

        {

        }


        void on_publication_matched(

                DataWriter*,

                const PublicationMatchedStatus& info) override

        {

            if (info.current_count_change == 1)

            {

                matched_ = info.total_count;

                std::cout << "Publisher matched." << std::endl;

            }

            else if (info.current_count_change == -1)

            {

                matched_ = info.total_count;

                std::cout << "Publisher unmatched." << std::endl;

            }

            else

            {

                std::cout << info.current_count_change

                        << " is not a valid value for PublicationMatchedStatus current count change." << std::endl;

            }

        }


        std::atomic_int matched_;


    } listener_;


public:


    HelloWorldPublisher()

        : participant_(nullptr)

        , publisher_(nullptr)

        , topic_(nullptr)

        , writer_(nullptr)

        , type_(new HelloWorldPubSubType())

    {

    }


    virtual ~HelloWorldPublisher()

    {

        if (writer_ != nullptr)

        {

            publisher_->delete_datawriter(writer_);

        }

        if (publisher_ != nullptr)

        {

            participant_->delete_publisher(publisher_);

        }

        if (topic_ != nullptr)

        {

            participant_->delete_topic(topic_);

        }

        DomainParticipantFactory::get_instance()->delete_participant(participant_);

    }


    //!Initialize the publisher

    bool init()

    {

        hello_.index(0);

        hello_.message("HelloWorld");


        DomainParticipantQos participantQos;

        participantQos.name("Participant_publisher");

        participant_ = DomainParticipantFactory::get_instance()->create_participant(0, participantQos);


        if (participant_ == nullptr)

        {

            return false;

        }


        // Register the Type

        type_.register_type(participant_);


        // Create the publications Topic

        topic_ = participant_->create_topic("HelloWorldTopic", "HelloWorld", TOPIC_QOS_DEFAULT);


        if (topic_ == nullptr)

        {

            return false;

        }


        // Create the Publisher

        publisher_ = participant_->create_publisher(PUBLISHER_QOS_DEFAULT, nullptr);


        if (publisher_ == nullptr)

        {

            return false;

        }


        // Create the DataWriter

        writer_ = publisher_->create_datawriter(topic_, DATAWRITER_QOS_DEFAULT, &listener_);


        if (writer_ == nullptr)

        {

            return false;

        }

        return true;

    }


    //!Send a publication

    bool publish()

    {

        if (listener_.matched_ > 0)

        {

            hello_.index(hello_.index() + 1);

            writer_->write(&hello_);

            return true;

        }

        return false;

    }


    //!Run the Publisher

    void run(

            uint32_t samples)

    {

        uint32_t samples_sent = 0;

        while (samples_sent < samples)

        {

            if (publish())

            {

                samples_sent++;

                std::cout << "Message: " << hello_.message() << " with index: " << hello_.index()

                            << " SENT" << std::endl;

            }

            std::this_thread::sleep_for(std::chrono::milliseconds(1000));

        }

    }

};


int main(

        int argc,

        char** argv)

{

    std::cout << "Starting publisher." << std::endl;

    uint32_t samples = 10;


    HelloWorldPublisher* mypub = new HelloWorldPublisher();

    if(mypub->init())

    {

        mypub->run(samples);

    }


    delete mypub;

    return 0;

}
```

#### 1.3.7.1 ソースコードの検証

ファイルの先頭には、Doxygen形式のコメントブロックがあり、@fileフィールドがファイル名を記述している。

```
/**
 * @file HelloWorldPublisher.cpp
 *
 */

```

以下にc++ヘッダーのインクルードを示す。最初のものは、前のセクションで定義したデータ型のシリアライゼーションおよびデシリアライゼーション関数を含むHelloWorldPubSubTypes.hファイルをインクルードしている。

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
TypeSupportクラスのtype_オブジェクトは、DomainParticipantにトピックデータ型を登録するために使用されるオブジェクトである。

```
private:

    HelloWorld hello_;

    DomainParticipant* participant_;

    Publisher* publisher_;

    Topic* topic_;

    DataWriter* writer_;

    TypeSupport type_;
```

次に、DataWriterListenerクラスを継承してPubListenerクラスが定義する。
このクラスはデフォルトのDataWriterリスナーコールバックをオーバーライドし、イベントが発生した場合にルーチンを実行することを可能にする。
オーバーライドされたコールバック ``on_publication?mached()``により、DataWriterが発行しているトピックをリッスン(?)している新しいDataReaderが検出されたときの一連のアクションを定義することができる。
``info.current_count_change``は、DataWriterにマッチしたDataReaderの変更を検出する。これは、MachedStatus構造体のメンバで、サブスクリプションのステータス変更を追跡できる。
最後に、このクラスのlistener_オブジェクトはPubListenerのインスタンスとして定義される。

```
class PubListener : public DataWriterListener
{
public:

    PubListener()
        : matched_(0)
    {
    }

    ~PubListener() override
    {
    }

    void on_publication_matched(
            DataWriter*,
            const PublicationMatchedStatus& info) override
    {
        if (info.current_count_change == 1)
        {
            matched_ = info.total_count;
            std::cout << "Publisher matched." << std::endl;
        }
        else if (info.current_count_change == -1)
        {
            matched_ = info.total_count;
            std::cout << "Publisher unmatched." << std::endl;
        }
        else
        {
            std::cout << info.current_count_change
                    << " is not a valid value for PublicationMatchedStatus current count change." << std::endl;
        }
    }

    std::atomic_int matched_;

} listener_;
```

HelloWorldPublisherクラスのpublucコンストラクタとデストラクタを以下に定義する。コンストラクタは、クラスのプライベート・データ・メンバをnullptrに初期化する。
ただし、TypeSupportオブジェクトはHelloWorldPubSubTypeクラスのインスタンスとして初期化される。クラスのデストラクタはこれらのデータ・メンバを削除し、システムメモリを削除する。

```
HelloWorldPublisher()
    : participant_(nullptr)
    , publisher_(nullptr)
    , topic_(nullptr)
    , writer_(nullptr)
    , type_(new HelloWorldPubSubType())
{
}

virtual ~HelloWorldPublisher()
{
    if (writer_ != nullptr)
    {
        publisher_->delete_datawriter(writer_);
    }
    if (publisher_ != nullptr)
    {
        participant_->delete_publisher(publisher_);
    }
    if (topic_ != nullptr)
    {
        participant_->delete_topic(topic_);
    }
    DomainParticipantFactory::get_instance()->delete_participant(participant_);
}
```

HelloWorldPublisherクラスのパブリックメンバ関数を続行すると、次のコードスニペットはパブリックパブリッシャーの初期化メンバー関数を定義する。この関数は、いくつかのアクションを実行する。

1. HelloWorld型のhello_構造体メンバの内容を初期化する。
2. DomainParticipantのQoSを通じて参加者に名前を割り当てる。
3. DomainParticipantFactoryを使用して参加者を作成する。
4. IDLで定義されたデータ型を登録する。
5. パブリケーションのトピックを作成する。
6. パブリッシャーを作成する。
7. 先に作成したリスナーでDataWriterを作成する。

以上のように、参加者を除くすべてのエンティティのQoS構成は、デフォルト構成(PARTICIPANT_QOS_DEFAULT、PUBLISHER_QOS_DEFAULT、TOPIC_QOS_DEFAULT、DATAWRITER_QOS_DEFAULT)である。
各DDSエンティティのデフォルト値は、DDSスタンダードで確認できる。

```
//!Initialize the publisher
bool init()
{
    hello_.index(0);
    hello_.message("HelloWorld");

    DomainParticipantQos participantQos;
    participantQos.name("Participant_publisher");
    participant_ = DomainParticipantFactory::get_instance()->create_participant(0, participantQos);

    if (participant_ == nullptr)
    {
        return false;
    }

    // Register the Type
    type_.register_type(participant_);

    // Create the publications Topic
    topic_ = participant_->create_topic("HelloWorldTopic", "HelloWorld", TOPIC_QOS_DEFAULT);

    if (topic_ == nullptr)
    {
        return false;
    }

    // Create the Publisher
    publisher_ = participant_->create_publisher(PUBLISHER_QOS_DEFAULT, nullptr);

    if (publisher_ == nullptr)
    {
        return false;
    }

    // Create the DataWriter
    writer_ = publisher_->create_datawriter(topic_, DATAWRITER_QOS_DEFAULT, &listener_);

    if (writer_ == nullptr)
    {
        return false;
    }
    return true;
}
```

パブリッシュを行うために、パブリック・メンバ関数 ``publish()``が実装されている。
DataWriterのリスナー・コールバックでは、DataWriterが発行トピックを受け取っているDataReaderと合致したことを示すために、データ・メンバmached_が更新される。
このデータ・メンバには、検出されたDataReaderの数が格納される。したがって、最初のDataReaderが検出されると、アプリケーションはパブリッシュを開始する。これは単にDataReaderオブジェクトによる変更の書き込みである。

```
//!Send a publication
bool publish()
{
    if (listener_.matched_ > 0)
    {
        hello_.index(hello_.index() + 1);
        writer_->write(&hello_);
        return true;
    }
    return false;
}
```

public run関数は、指定された回数のパブリッシュを実行し、パブリッシュの間に1秒間待機する。

```
//!Run the Publisher
void run(
        uint32_t samples)
{
    uint32_t samples_sent = 0;
    while (samples_sent < samples)
    {
        if (publish())
        {
            samples_sent++;
            std::cout << "Message: " << hello_.message() << " with index: " << hello_.index()
                        << " SENT" << std::endl;
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    }
}
```

最後に、HelloWorldPublisherが初期化され、mainで実行される。

```
int main(
        int argc,
        char** argv)
{
    std::cout << "Starting publisher." << std::endl;
    uint32_t samples = 10;

    HelloWorldPublisher* mypub = new HelloWorldPublisher();
    if(mypub->init())
    {
        mypub->run(samples);
    }

    delete mypub;
    return 0;
}
```

#### CMakeLists.txt

先ほど作成したCMakeLists.txtファイルの最後に、以下のコードスニペットをインクルードする。これにより、実行ファイルのビルドに必要なすべてのソースファイルが追加され、実行ファイルとライブラリがリンクされる。

```
add_executable(DDSHelloWorldPublisher src/HelloWorldPublisher.cpp ${DDS_HELLOWORLD_SOURCES_CXX})
target_link_libraries(DDSHelloWorldPublisher fastdds fastcdr)
```

この時点で、プロジェクトはパブリッシャーアプリケーションのビルド、コンパイル、実行の準備が整った。
ワークスペースのビルドディレクトリから以下のコマンドを実行する。

```
cmake ..
cmake --build .
./DDSHelloWorldPublisher
```

### 1.3.8 FastDDSサブスクライバーの作成

ワークスペースのsrcディレクトリから次のコマンドを実行して、HelloWorldSubsriber.cppファイルをダウンロードする。

```
// Copyright 2016 Proyectos y Sistemas de Mantenimiento SL (eProsima).

//

// Licensed under the Apache License, Version 2.0 (the "License");

// you may not use this file except in compliance with the License.

// You may obtain a copy of the License at

//

//     http://www.apache.org/licenses/LICENSE-2.0

//

// Unless required by applicable law or agreed to in writing, software

// distributed under the License is distributed on an "AS IS" BASIS,

// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

// See the License for the specific language governing permissions and

// limitations under the License.


/**

 * @file HelloWorldSubscriber.cpp

 *

 */


#include "HelloWorldPubSubTypes.hpp"


#include <chrono>

#include <thread>


#include <fastdds/dds/domain/DomainParticipant.hpp>

#include <fastdds/dds/domain/DomainParticipantFactory.hpp>

#include <fastdds/dds/subscriber/DataReader.hpp>

#include <fastdds/dds/subscriber/DataReaderListener.hpp>

#include <fastdds/dds/subscriber/qos/DataReaderQos.hpp>

#include <fastdds/dds/subscriber/SampleInfo.hpp>

#include <fastdds/dds/subscriber/Subscriber.hpp>

#include <fastdds/dds/topic/TypeSupport.hpp>


using namespace eprosima::fastdds::dds;


class HelloWorldSubscriber

{

private:


    DomainParticipant* participant_;


    Subscriber* subscriber_;


    DataReader* reader_;


    Topic* topic_;


    TypeSupport type_;


    class SubListener : public DataReaderListener

    {

    public:


        SubListener()

            : samples_(0)

        {

        }


        ~SubListener() override

        {

        }


        void on_subscription_matched(

                DataReader*,

                const SubscriptionMatchedStatus& info) override

        {

            if (info.current_count_change == 1)

            {

                std::cout << "Subscriber matched." << std::endl;

            }

            else if (info.current_count_change == -1)

            {

                std::cout << "Subscriber unmatched." << std::endl;

            }

            else

            {

                std::cout << info.current_count_change

                          << " is not a valid value for SubscriptionMatchedStatus current count change" << std::endl;

            }

        }


        void on_data_available(

                DataReader* reader) override

        {

            SampleInfo info;

            if (reader->take_next_sample(&hello_, &info) == eprosima::fastdds::dds::RETCODE_OK)

            {

                if (info.valid_data)

                {

                    samples_++;

                    std::cout << "Message: " << hello_.message() << " with index: " << hello_.index()

                              << " RECEIVED." << std::endl;

                }

            }

        }


        HelloWorld hello_;


        std::atomic_int samples_;


    }

    listener_;


public:


    HelloWorldSubscriber()

        : participant_(nullptr)

        , subscriber_(nullptr)

        , topic_(nullptr)

        , reader_(nullptr)

        , type_(new HelloWorldPubSubType())

    {

    }


    virtual ~HelloWorldSubscriber()

    {

        if (reader_ != nullptr)

        {

            subscriber_->delete_datareader(reader_);

        }

        if (topic_ != nullptr)

        {

            participant_->delete_topic(topic_);

        }

        if (subscriber_ != nullptr)

        {

            participant_->delete_subscriber(subscriber_);

        }

        DomainParticipantFactory::get_instance()->delete_participant(participant_);

    }


    //!Initialize the subscriber

    bool init()

    {

        DomainParticipantQos participantQos;

        participantQos.name("Participant_subscriber");

        participant_ = DomainParticipantFactory::get_instance()->create_participant(0, participantQos);


        if (participant_ == nullptr)

        {

            return false;

        }


        // Register the Type

        type_.register_type(participant_);


        // Create the subscriptions Topic

        topic_ = participant_->create_topic("HelloWorldTopic", "HelloWorld", TOPIC_QOS_DEFAULT);


        if (topic_ == nullptr)

        {

            return false;

        }


        // Create the Subscriber

        subscriber_ = participant_->create_subscriber(SUBSCRIBER_QOS_DEFAULT, nullptr);


        if (subscriber_ == nullptr)

        {

            return false;

        }


        // Create the DataReader

        reader_ = subscriber_->create_datareader(topic_, DATAREADER_QOS_DEFAULT, &listener_);


        if (reader_ == nullptr)

        {

            return false;

        }


        return true;

    }


    //!Run the Subscriber

    void run(

            uint32_t samples)

    {

        while (listener_.samples_ < samples)

        {

            std::this_thread::sleep_for(std::chrono::milliseconds(100));

        }

    }


};


int main(

        int argc,

        char** argv)

{

    std::cout << "Starting subscriber." << std::endl;

    uint32_t samples = 10;


    HelloWorldSubscriber* mysub = new HelloWorldSubscriber();

    if (mysub->init())

    {

        mysub->run(samples);

    }


    delete mysub;

    return 0;

}
```

#### 1.3.8.1 コードの検証

パブリッシャーとサブスクライバーのアプリケーションのソースコードは、ほとんど同じであるため、このドキュメントでは、既に説明されている部分を省き、両社の主な違いに焦点を当て解説する。パブリッシャーの説明と同じ構造に従い、最初のステップはC++ヘッダーファイルのインクルードである。この中で、パブリッシャークラスをインクルードしているファイルはサブスクライバークラスに、DataWriterクラスはDataReaderくらすに置き換わっている。

- ``Subscriber``.DataReaderの作成と設定を担当するオブジェクト。
- ``DataReader``.データの実際の受信を担当するオブジェクト。読み取るデータを識別するトピック(TopicDscription)をアプリケーションに登録し、サブスクライバーが受信したデータにアクセスする。
- ``DataReaderListener``.DataReaderに割り当てられたリスナー。
- ``DataReaderQoS``.DataReaderのQoSを定義する構造体。
- ``SampleInfo``.「read」または「taken」された各サンプルに付随する情報。

```
#include <fastdds/dds/domain/DomainParticipant.hpp>
#include <fastdds/dds/domain/DomainParticipantFactory.hpp>
#include <fastdds/dds/subscriber/DataReader.hpp>
#include <fastdds/dds/subscriber/DataReaderListener.hpp>
#include <fastdds/dds/subscriber/qos/DataReaderQos.hpp>
#include <fastdds/dds/subscriber/SampleInfo.hpp>
#include <fastdds/dds/subscriber/Subscriber.hpp>
#include <fastdds/dds/topic/TypeSupport.hpp>
```

次の行では、サブスクライバを実装する HelloWorldSubscriber クラスを定義している。

```
class HelloWorldSubscriber
```

クラスのプライベートデータメンバから初めて、データリーダーリスナーの実装に言及する価値がある。
クラスのプライベートメンバは、参加者、サブスクライバー、トピック、DataReader、およびデータ型になる。DataWriterの場合と同じように、リスナーはイベントが発生した場合に実行されるコールバックを実装する。
SubListenerの最初のオーバーライドされたコールバックは ``on_subscription_mached()``であり、これはDataWriterの ``on_publication_mached()``コールバックのアナログである。

```
void on_subscription_matched(
        DataReader*,
        const SubscriptionMatchedStatus& info) override
{
    if (info.current_count_change == 1)
    {
        std::cout << "Subscriber matched." << std::endl;
    }
    else if (info.current_count_change == -1)
    {
        std::cout << "Subscriber unmatched." << std::endl;
    }
    else
    {
        std::cout << info.current_count_change
                  << " is not a valid value for SubscriptionMatchedStatus current count change" << std::endl;
    }
}
```

2番目のオーバーライドされたコールバックは、``on_data_available()``である。このコールバックでは、DataReaderがアクセスできる次の受信したサンプルが取り込まれ、その内容が表示されるように処理される。
ここｋで、SampleInfoクラスのオブジェクトが定義され、サンプルが既に読み込まれたか、取り込まれたかを判断する。サンプルが読み込まれるたびに、受信したサンプルのカウンタが増加する。

```
void on_data_available(
        DataReader* reader) override
{
    SampleInfo info;
    if (reader->take_next_sample(&hello_, &info) == eprosima::fastdds::dds::RETCODE_OK)
    {
        if (info.valid_data)
        {
            samples_++;
            std::cout << "Message: " << hello_.message() << " with index: " << hello_.index()
                      << " RECEIVED." << std::endl;
        }
    }
}
```

クラスのパブリック・コンストラクタとデストラクタを以下に定義する。

```
HelloWorldSubscriber()
    : participant_(nullptr)
    , subscriber_(nullptr)
    , topic_(nullptr)
    , reader_(nullptr)
    , type_(new HelloWorldPubSubType())
{
}

virtual ~HelloWorldSubscriber()
{
    if (reader_ != nullptr)
    {
        subscriber_->delete_datareader(reader_);
    }
    if (topic_ != nullptr)
    {
        participant_->delete_topic(topic_);
    }
    if (subscriber_ != nullptr)
    {
        participant_->delete_subscriber(subscriber_);
    }
    DomainParticipantFactory::get_instance()->delete_participant(participant_);
```

次に、サブスクライバの初期化パブリックメンバ関数が来る。これは、HelloWorldPublisherで定義された初期化パブリックメンバ関数と同じである。
参加者を除くすべてのエンティティのQoS構成は、デフォルトのQoS(``PARTICIPANT_QOS_DEFAULT``、``SUBSCRIBER_QOS_DEFAULT``、``TOPIC_QOS_DEFAULT``、``DATAREADER_QOS_DEFAULT``)である。
各DDSエンティティのQoSデフォルト値は、DDS標準で確認できる。

```
//!Initialize the subscriber
bool init()
{
    DomainParticipantQos participantQos;
    participantQos.name("Participant_subscriber");
    participant_ = DomainParticipantFactory::get_instance()->create_participant(0, participantQos);

    if (participant_ == nullptr)
    {
        return false;
    }

    // Register the Type
    type_.register_type(participant_);

    // Create the subscriptions Topic
    topic_ = participant_->create_topic("HelloWorldTopic", "HelloWorld", TOPIC_QOS_DEFAULT);

    if (topic_ == nullptr)
    {
        return false;
    }

    // Create the Subscriber
    subscriber_ = participant_->create_subscriber(SUBSCRIBER_QOS_DEFAULT, nullptr);

    if (subscriber_ == nullptr)
    {
        return false;
    }

    // Create the DataReader
    reader_ = subscriber_->create_datareader(topic_, DATAREADER_QOS_DEFAULT, &listener_);

    if (reader_ == nullptr)
    {
        return false;
    }

    return true;
```

パブリックメンバ関数 ``run()``は、全てのサンプルが受信されるまでサブスクライバを確実に実行する。このメンバ関数は、CPUを楽にするために100msのスリープ間隔を持つサブスクライバのアクティブな待機を実装する。

```
//!Run the Subscriber
void run(
        uint32_t samples)
{
    while (listener_.samples_ < samples)
    {
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
```

最後に、サブスクライバを実装する参加者が初期化され、mainで実行される。

```
};

int main(
        int argc,
        char** argv)
{
    std::cout << "Starting subscriber." << std::endl;
    uint32_t samples = 10;

    HelloWorldSubscriber* mysub = new HelloWorldSubscriber();
    if (mysub->init())
    {
        mysub->run(samples);
    }

    delete mysub;
```

#### 1.3.8.2 CMakeLists.txt

先ほど作成したCMakeLists.txtファイルの最後に、以下のコードスニペットをインクルードする。これにより、実行ファイルのビルドに必要な全てのソースファイルが追加され、実行ファイルとライブラリがリンクされる。

```
add_executable(DDSHelloWorldSubscriber src/HelloWorldSubscriber.cpp ${DDS_HELLOWORLD_SOURCES_CXX})
target_link_libraries(DDSHelloWorldSubscriber fastdds fastcdr)
```

この時点で、プロジェクトはサブスクライバーアプリケーションのビルド、コンパイル、および実行の準備が整った。
ワークスペースのビルドディレクトリから、以下のコマンドを実行する。

```
cmake ..
cmake --build .
./DDSHelloWorldSubscriber
```

### 1.3.9 実行

最後に、ビルドディレクトリから、パブリッシャーとサブスクライバーのアプリケーションを2つのターミナルから実行する。

```
./DDSHelloWorldPublisher
./DDSHelloWorldSubscriber
```

{{< video src="simple_pubsub.mp4" controls="true" >}}{{< /video >}}

### 1.3.10 要約

このチュートリアルでは、パブリッシャーとサブスクライバーのDDSアプリケーションを作成した。また、ソースコードをコンパイルするためのCMakeファイルのビルド方法と、FastDDSおよびFastCDRライブラリをプロジェクトにインクルードして使用する方法を学習した。

### 1.3.11 次のステップ

eProsima FastDDSのGithubリポジトリには、多くのユースケースやシナリオに対応したDDS通信を実装した、より複雑なサンプルがある。[こちら](https://github.com/eProsima/Fast-DDS/tree/master/examples/cpp)にある。