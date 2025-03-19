---
title: "3章 DDSレイヤー"
draft: false
featured: false
categories: ["FastDDS_Manual"]
tags: [3章]
summary: ""
---
eProsima Fast DDSは、2つの異なるAPIを公開し、異なるレベルで通信サービスと相互作用する。 メインのAPIは、Data Distribution Service (DDS) Data-Centric Publish-Subscribe (DCPS) Platform Independent Model (PIM) API、略してDDS DCPS PIMで、Fast DDSが準拠しているData Distribution Service (DDS) version 1.4仕様で定義されている。 このセクションでは、Fast DDS の下でこの API の主な特徴と使用モードについて説明し、この API が分割された 5 つのモジュールについて詳しく説明する：

- コア: 他のモジュールによって洗練される抽象クラスとインターフェースを定義する。また、QoS(Quality of Service)の定義や、ミドルウェアとの通知ベースのインタラクションスタイルのサポートも提供する。
- ドメイン: ```DomainParticipant```クラスが含まれ、サービスのエントリーポイントとして、また多くのクラスのファクトリーとして機能する。DomainParticipantは、サービスを構成する他のオブジェクトのコンテナとしても機能する。
- パブリッシャー: ```Publisher```クラスと```DataWriter```クラス、```PublisherListener```インターフェースと```DataWriterListener```インターフェースなど、発行側で使用されるクラスについて説明する。
- サブスクライバー: ```Subscriber```クラスと```DataReader```クラス、および```SubscriberListener```インターフェースと```DataReaderLsetener```インターフェースなど、サブスクリプション側で使用されるクラスについて説明する。
- トピック: ```Topic```と```TopicDescription```クラスなど、通信トピックとデータ型を定義に使用されるクラスについて説明する。