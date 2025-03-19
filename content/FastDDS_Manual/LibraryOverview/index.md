---
title: "2章 ライブラリーの概要"
draft: false
featured: false
categories: ["FastDDS_Manual"]
tags: [2章]
summary: ""
---
# 2 ライブラリーの概要

FastDDS(旧FastRTPS)は、分散アプリケーションソフトウェアのためのデータ中心通信ミドルウェア(DCPS)であるDDS仕様の効率的で高性能な実装である。  
このセクションでは、FastDDSのアーキテクチャ、動作、および主な機能について説明する。

## 2.1 アーキテクチャ

高速DDSのアーキテクチャは下図のようになっており、以下のような異なる環境を持つレイヤーモデルを見ることができる。

- **アプリケーションレイヤー**:分散システムにおける通信の実装にFastDDSAPIを使用するユーザーアプリケーション。
- **FastDDSレイヤー**:DDS通信ミドルウェアの堅牢な実装。同一ドメイン内のDo0mainParticipantがドメイントピックの下でパブリッシュ/サブスクライブすることでメッセージを交換する、１つまたは複数のDDSドメインの展開を可能にする。
- **RTPSレイヤー**:DDSアプリケーションとの相互運用性のためのRTPS(Real-Time Publish-Subscribe)プロトコルの実装。このレイヤーは、トランスポートレイヤーの抽象化レイヤーとして機能する。
- **トランスポートレイヤー**:FastDDSは、信頼性の低いトランスポートプロトコル(UDP)、信頼性の高いトランスポートプロトコル(TCP)、共有メモリトランスポートプロトコル(SHM)など、様々なトランスポートプロトコル上で使用できる。

![library_overview](library_overview.svg)

## 2.1 DDSレイヤー

FastDDSのDDSレイヤーでは、通信のためのいくつかの重要な要素が定義されている。ユーザーはアプリケーションでこれらの要素を作成しDDSアプリケーション要素を組み込んでデータ中心の通信システムを作成する。  
FastDDSは、DDS使用に従い、通信に関わるこれらの要素をエンティティとして定義する。DDSエンティティとは、QoS(Quality of Service)設定をサポートし、リスナーを実装するオブジェクトのことである。

- **QoS**: 各エンティティの動作が定義されるメカニズム。
- **リスナー**: アプリケーションの実行中に発生する可能性があるイベントをエンティティに通知するメカニズム。

以下に、DDSエンティティをその説明と機能とともに列挙する。 各エンティティの詳細、QoS、リスナーについてはDDSレイヤーの項を参照。

- **ドメイン** :DDSドメインを識別する正の整数。各 DomainParticipant には割り当てられた DDS ドメインがあり、同じドメイン内の DomainParticipant が通信したり、DDS ドメイン間の通信を分離したりできる。この値は、アプリケーション開発者がDomainParticipantsを作成するときに指定する必要がある。

- **DomainParticipant** :パブリッシャー、サブスクライバー、トピック、マルチトピックなどの他のDDSエンティティを含むオブジェクト。これは、それが含む以前のエンティティの作成と、それらの動作の設定を可能にするエンティティである。

- **パブリッシャー** :パブリッシャーは、データをトランスポートに書き込むDataWriterを使用して、トピックの下にデータをパブリッシュする。これは、それが含むDataWriterエンティティの作成と設定を行うエンティティであり、1つまたは複数のエンティティを含むことができる。

- **DataWriter** :メッセージの発行を担当するエンティティ。ユーザは、このエンティティを作成するときに、データが発行されるトピックを指定する必要がある。発行は、データオブジェクトを DataWriterHistory の変更として書き込むことで行われる。

- **DataWriterHistory** :これは、データオブジェクトに対する変更のリストである。DataWriter が特定の トピックの下でデータの発行に進むと、実際にこのデータに変更が作成される。履歴に登録されるのはこの変更である。これらの変更は、特定のTopicを購読しているDataReaderに送信される。

- **サブスクライバ** :サブスクライバは、トランスポートからデータを読み取る DataReader を使用して、トピックをサブスクライブする。これは、それが含む DataReader エンティティを作成および構成するエンティティであり、1 つまたは複数の DataReader エンティティを含むことができる。

- **DataReader** :パブリケーション受信のためにトピックを購読するエンティティである。ユーザは、このエンティティの作成時にサブスクリプションTopicを提供しなければならない。DataReaderは、そのDataReaderHistoryの変更としてメッセージを受信する。

- **DataReaderHistory** :これは、DataReaderが特定のTopicをサブスクライブした結果として受信したデータオブジェクトの変更を含んでいる。

- **トピック** :パブリッシャの DataWriter とサブスクライバの DataReader を結合するエンティティ。

### 2.1.2 RTPSレイヤー

前述のように、Fast DDSのRTPSプロトコルは、トランスポートレイヤーからDDSアプリケーションエンティティを抽象化することができる。 上のグラフによると、RTPSレイヤには4つの主要エンティティがある。

- RTPSドメイン RTPSプロトコルに対するDDSドメインの拡張。
- RTPSParticipant 他のRTPSエンティティを含むエンティティ。それを含むエンティティのコンフィギュレーションと作成を可能にする。
- RTPSWriter メッセージのソース。DataWriterHistoryに書き込まれた変更を読み取り、以前にマッチした全てのRTPSReaderへ送信する。
- RTPSReader メッセージの受信エンティティ。RTPSWriterによって報告された変更をDataReaderHistoryに書き込む。

### 2.1.3 トランスポートレイヤー
FastDDSは、さまざまなトランスポートプロトコル上でのアプリケーションの実装をサポートしている。 UDPv4、UDPv6、TCPv4、TCPv6、およびシェアードメモリトランスポート（SHM）である。 デフォルトでは、DomainParticipantはUDPv4とSHMトランスポートプロトコルを実装する。 サポートされているすべてのトランスポートプロトコルのコンフィギュレーションは、 トランスポートレイヤーのセクションで詳しく説明されている。

## 2.2 プログラミングと実行モデル

FastDDSは同時実行及びイベントベースである。以下では、FastDDSの動作を支配するマルチスレッドも出ると、起こりえるイベントについて説明する。

### 2.2.1 並行処理とマルチスレッド

FastDDSは並行マルチスレッドを実装している。各DomainParticipantは、ロギング、メッセージ受信、非同期通信などのバックグラウンドタスクを処理するスレッドのセットを生成する。  
つまり、FastDDSAPIはスレッドセーフなので、異なるスレッドから同じDomainParticipantのメソッドを呼びだすことができる。  
ただし、外部関数がライブラリ内で実装されているスレッドによって変更されるリソースにアクセスする場合は、このマルチスレッド実装を考慮する必要がある。  
FastDDSによって生成されるスレッドの完全なセットを以下に示す。トランスポート関連のスレッド (UDP、TCP、SHM タイプとしてマークされる) は、適切なトランスポートが使用される場合にのみ作成される。


| 名前 | 種類 | カーディナリティ | OSスレッド名 | 説明 |
| --- | --- | --- | --- | --- |
| **Event** | 一般 | ドメインパーティシパントごとに1つ | `dds.ev.<participant_id>` | 定期的およびトリガーされた時間イベントを処理する。詳細はDomainParticipantQosを参照。 |
| **Discovery Server Event** | 一般 | ドメインパーティシパントごとに1つ | `dds.ds_ev.<participant_id>` | Discovery Serverデータベースへのアクセスを同期する。詳細はDomainParticipantQosを参照。 |
| **Asynchronous Writer** | 一般 | 有効な非同期フローコントローラごとに1つ。最小1つ。 | `dds.asyn.<participant_id>.<async_flow_controller_index>` | 非同期書き込みを管理する。同期ライターであっても、いくつかの通信形式はバックグラウンドで開始する必要がある。詳細はDomainParticipantQosおよびFlowControllersQosを参照。 |
| **Datasharing Listener** | 一般 | データリーダーごとに1つ | `dds.dsha.<reader_id>` | Datasharingを介して受信したメッセージを処理するリスナースレッド。詳細はDataReaderQosを参照。 |
| **Reception (UDP)** | UDP | ポートごとに1つ | `dds.udp.<port>` | 受信したUDPメッセージを処理するリスナースレッド。詳細はTransportConfigQosおよびUDPTransportDescriptorを参照。 |
| **Reception (TCP)** | TCP | TCP接続ごとに1つ | `dds.tcp.<port>` | 受信したTCPメッセージを処理するリスナースレッド。詳細はTCPTransportDescriptorを参照。 |
| **Accept (TCP)** | TCP | TCPトランスポートごとに1つ | `dds.tcp_accept` | 受信したTCP接続要求を処理するスレッド。詳細はTCPTransportDescriptorを参照。 |
| **Keep Alive (TCP)** | TCP | TCPトランスポートごとに1つ | `dds.tcp_keep` | TCP接続のためのキープアライブスレッド。詳細はTCPTransportDescriptorを参照。 |
| **Reception (SHM)** | SHM | ポートごとに1つ | `dds.shm.<port>` | SHMセグメント経由で受信したメッセージを処理するリスナースレッド。詳細はTransportConfigQosおよびSharedMemTransportDescriptorを参照。 |
| **Logging (SHM)** | SHM | ポートごとに1つ | `dds.shmd.<port>` | 転送されたパケットをファイルに記録・ダンプする。詳細はTransportConfigQosおよびSharedMemTransportDescriptorを参照。 |
| **Watchdog (SHM)** | SHM | 1つ | `dds.shm.wdog` | 開いている共有メモリセグメントの状態を監視する。詳細はTransportConfigQosおよびSharedMemTransportDescriptorを参照。 |
| **General Logging** | ログ | 1つ | `dds.log` | 消費者ログにエントリを蓄積し、適切なログに書き込む。詳細はLogging Threadを参照。 |
| **Security Logging** | ログ | ドメインパーティシパントごとに1つ | `dds.slog.<participant_id>` | セキュリティログエントリを蓄積し、書き込む。詳細はDomainParticipantQosを参照。 |
| **Watchdog (Filewatch)** | ファイル監視 | 1つ | `dds.fwatch` | 監視対象ファイルの変更ステータスを追跡する。詳細はDomainParticipantFactoryQosを参照。 |
| **Callback (Filewatch)** | ファイル監視 | 1つ | `dds.fwatch.cb` | 監視対象ファイルの変更時に登録されたコールバックを実行する。詳細はDomainParticipantFactoryQosを参照。 |
| **Reception (TypeLookup Service)** | 型ルックアップサービス | ドメインパーティシパントごとに2つ | `dds.tls.replies.<participant_id>` `dds.tls.requests.<participant_id>` | 未知のデータ型でリモートエンドポイントのディスカバリ情報を受信したときに動作する。 |

これらのスレッドの中には、特定の条件が満たされたときにのみ発生するものもある：
- Datasharingリスナースレッドは、Datasharingが使用されている場合にのみ作成される。
- Discovery Serverイベントスレッドは、DomainParticipantがDiscovery Server SERVERとして構成されている場合にのみ作成される。 
- TCPキープアライブスレッドでは、キープアライブ期間を0より大きい値に構成する必要がある。 
- セキュリティロギングおよび共有メモリパケットロギングスレッドでは、どちらも特定の構成オプションを有効にする必要がある。 
- Filewatchスレッドは、FASTDDS_ENVIRONMENT_FILEが使用されている場合にのみ生成される。

トランスポートスレッドに関して、FastDDSはデフォルトでUDPto共有メモリトランスポートの両方を使用する。ポート構成は展開の特定のニーズに合わせて構成できるが、デフォルトの構成では常にメタトラフィックポートとユニキャストユーザートラフィックポートを使用する。  
TCPはマルチキャストをサポートしていないため、これはUDPと共有メモリの両方に適用される。

FastDDSは、ThreadSettingsによって、作成するスレッドの特定の属性を設定することができる。

### 2.2.2 イベント駆動型アーキテクチャ
FastDDSが特定の状態に反応したり、定期的なオペレーションをスケジュールしたりすることを可能にするタイムイベントシステムがある。これらのほとんどはDDSとRTPSメタデータに関連するため、ユーザーにはほとんど見えない。しかし、ユーザーはTimedEventクラスを継承することで、アプリケーションで周期的なタイムイベントを定義することができる。

## 2.3 機能
Fast DDSには、ユーザーがアプリケーションに実装して設定できる追加機能がいくつかある。 その概要は以下の通り。

### 2.3.1 ディスカバリープロトコル
ディスカバリープロトコルは、あるトピックの下で発行するDataWriterと、同じトピックをサブスクライブするDataReaderをマッチングさせ、データの共有を開始させるメカニズムを定義する。これは通信プロセスのどの時点でも定義される。  
FastDDSは以下のディスカバリーメカニズムを定義する:

- **シンプルディスカバリー。**これはRTPS標準に定義されているデフォルトのディスカバリメカニズムであり、他のDDS実装との互換性を提供する。ここでは、DomainParticipantはそれらが実装するDataWriterとDataReaderに続いて一致するように、早い段階で個別に発見される。
- **ディスカバリーサーバー。**このディスカバリメカニズムは集中型のディスカバリアーキテクチャを使用し、サーバーはメタトラフィックディスカバリのハブとして機能する。
- **スタティックディスカバリ。**これはDomainParticipant同士のディスカバリを実装するが、各 DomainParticipantに含まれるエンティティ(DataReader/DataWriter)のディスカバリをスキップすることが可能である。 
- **手動ディスカバリ。** このメカニズムはRTPSレイヤーのみと互換性がある。 これは、外部メタ情報チャネルを使用して、RTPSParticipants、 RTPSWriters、RTPSReadersのマッチングとアンマッチを手動で行うことを可能にする。

Fast DDSに実装されているすべてのディスカバリープロトコルの詳細な説明と設定は、ディスカバリーセクションを参照。

### 2.3.2 セキュリティ
FastDDSは、3つのレベルでプラグイン可能なセキュリティを実装することにより、安全な通信を提供するように構成することができる。

リモートDomainParticipantsの認証。 DDS:Auth:PKI-DHプラグインは、信頼できる認証局（CA）とECDSAデジタル署名アルゴリズムを使用して認証を行い、相互認証を実行する。 また、鍵合意プロトコルとして楕円曲線ディフィー・ヘルマン（ECDH）またはMODP-2048ディフィー・ヘルマン（DH）を使用して共有秘密を確立します。 エンティティのアクセス制御。 DDS:Access:Permissionsプラグインは、DDSドメインおよびトピックレベルでのDomainParticipantsへのアクセス制御を提供する。 データの暗号化。 DDS:Crypto:AES-GCM-GMACプラグインは、ガロアカウンターモード（AES-GCM）のAdvanced Encryption Standard（AES）を使用した認証暗号化を提供します。