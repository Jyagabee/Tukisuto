---
title: "3章 標準的なQoSポリシー"
draft: false
featured: false
categories: ["FastDDS_Manual"]
tags: [3.1章]
summary: ""
---
### 3.1.2.1. 標準的なQoSポリシー
このセクションでは、DDS標準の各QoSポリシーについて説明する：

- DeadlineQosPolicy
- DestinationOrderQosPolicy
- DurabilityQosPolicy
- DurabilityServiceQosPolicy
- EntityFactoryQosPolicy
- GroupDataQosPolicy
- HistoryQosPolicy
- LatencyBudgetQosPolicy
- LifespanQosPolicy
- LivelinessQosPolicy
- OwnershipQosPolicy
- OwnershipStrengthQosPolicy
- PartitionQosPolicy
- PresentationQosPolicy
- ReaderDataLifecycleQosPolicy
- ReliabilityQosPolicy
- ResourceLimitsQosPolicy
- TimeBasedFilterQosPolicy
- TopicDataQosPolicy
- TransportPriorityQosPolicy
- UserDataQosPolicy
- WriterDataLifecycleQosPolicy

#### 3.1.2.1.1. DeadlineQosPolicy
このQoSポリシーは、新しいサンプルの頻度がある閾値を下回るとアラームを発する。 これは、データが定期的に更新されることが期待される場合に便利である（```DeadlineQosPolicy```を参照）。   
公開側では、期限はアプリケーションが新しいサンプルを供給することが期待される最大期間を定義する。   
キーを持つトピックの場合、このQoSはキーごとに適用される。 ある車両の位置を定期的に公開する必要があるとする。 その場合、データ型のキーとして車両のIDを設定し、期限QoSを希望する公開期間に設定することが可能である。 

QoS Policyデータメンバー一覧：
|データメンバ名|型|デフォルト値|
|---|---|--- |
|```period```|```Duration_t```|```c_TimeInfinite```|

##### 互換性ルール
DataReaderとDataWriterのDeadlineQosPolicy間の互換性を維持するために、（DataWriterに設定された）オファーされた期限期間は、（DataReaderに設定された）要求された期限期間以下でなければならない。

DeadlineQosPolicyは TimeBasedFilterQosPolicy と一貫して設定する必要がある。つまり、期限は 期間は最小間隔以上でなければならない。 

##### 例
<detail><summary>C++</summary>
```// This example uses a DataWriter, but it can also be applied to DataReader and Topic entities
DataWriterQos writer_qos;
// The DeadlineQosPolicy is constructed with an infinite period by default
// Change the period to 1 second
writer_qos.deadline().period.seconds = 1;
writer_qos.deadline().period.nanosec = 0;
// Use modified QoS in the creation of the corresponding entity
writer_ = publisher_->create_datawriter(topic_, writer_qos);```
</detail>

<detail><summary>XML</summary>
```<data_writer profile_name="writer_xml_conf_deadline_profile">
    <qos>
        <deadline>
            <period>
                <sec>1</sec>
            </period>
        </deadline>
    </qos>
</data_writer>

<data_reader profile_name="reader_xml_conf_deadline_profile">
    <qos>
        <deadline>
            <period>
                <sec>1</sec>
            </period>
        </deadline>
    </qos>
</data_reader>```
</detail>

#### 3.1.2.1.2 DeadlineQosPolicy
複数のDataWriterが同じTopicで同じキーを使用してメッセージを送信することができ、DataReader側では、それらのメッセージはすべてデータの同じインスタンス内に格納される（DestinationOrderQosPolicyを参照）。このQoSポリシーは、それらのメッセージの論理的な順序を決定するために使用される基準を制御する。システムの動作は、DestinationOrderQosPolicyKind の値に依存する。

QoS Policyデータメンバ一覧：
|データメンバ名|型|デフォルト値|
|---|---|--- |
|```kind```|```DestinationOrderQosPolicyKind```|```BY_RECEPTION_TIMESTAMP_DESTINATIONORDER_QOS```|

##### DestinationOrderQosPolicyKind
可能な値は2つある(```DestinationOrderQosPolicyKind```参照)：

