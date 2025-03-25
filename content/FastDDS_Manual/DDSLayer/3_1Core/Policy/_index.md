---
title: "3章 ポリシー"
draft: false
featured: false
categories: ["FastDDS_Manual"]
tags: [3.1章]
summary: ""
---
## 3.1.2. ポリシー
サービス品質（QoS）はサービスの動作を指定するために使用され、ユーザーは各エンティティがどのように動作するかを定義することができる。 システムの柔軟性を高めるため、QoSは複数のQoSポリシーに分割され、それぞれ独立に設定できる。 ただし、複数のポリシーが競合する場合もある。 各Qosポリシーは、QosPolicyId_t列挙子で定義された一意のIDを持つ。 このIDは、一部のStatusインスタンスで、Statusが参照する特定のQos Policyを識別するために使用される。 QoS Policyには不変のものがあり、エンティティの作成時またはenable操作を呼び出す前にのみ指定できる。 各DSエンティティは、標準QoS Policy、XTypes拡張、およびeProsima拡張を混在させることができるQoS Policyの特定のセットを持つ。