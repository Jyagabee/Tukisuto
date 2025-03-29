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
```
<data_writer profile_name="writer_xml_conf_deadline_profile">
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
</data_reader>
```
</detail>

#### 3.1.2.1.2 DeadlineQosPolicy
複数のDataWriterが同じTopicで同じキーを使用してメッセージを送信することができ、DataReader側では、それらのメッセージはすべてデータの同じインスタンス内に格納される（DestinationOrderQosPolicyを参照）。このQoSポリシーは、それらのメッセージの論理的な順序を決定するために使用される基準を制御する。システムの動作は、DestinationOrderQosPolicyKind の値に依存する。

QoS Policyデータメンバ一覧：
|データメンバ名|型|デフォルト値|
|---|---|--- |
|```kind```|```DestinationOrderQosPolicyKind```|```BY_RECEPTION_TIMESTAMP_DESTINATIONORDER_QOS```|

##### DestinationOrderQosPolicyKind
可能な値は2つある(```DestinationOrderQosPolicyKind```参照)：

- ```BY_RECEPTION_TIMESTAMP_DESTINATIONORDER_QOS```: これは、各DataReaderでの受信時刻に基づいてデータを並べ替えることを示す。 
- ```BY_SOURCE_TIMESTAMP_DESTINATIONORDER_QOS```：これは、メッセージが送信された時のDataWriterのタイムスタンプに基づいてデータを並べることを示す。 このオプションは最終値の一貫性を保証する。

どちらのオプションもOwnershipQosPolicyとOwnershipStrengthQosPolicyの値に依存する。つまり、OwnershipがEXCLUSIVEに設定されていて、最後の値が所有権の強度が低いDataWriterから来たものであれば、それは破棄される。

###### 互換性ルール
DataReaderとDataWriterのDestinationOrderQosPolicyが異なるkind値を持つ場合の互換性を維持するために、 DataWriterのkindはDataReaderのkindより高いか等しくなければならない。 また、異なる種類の間の順序は、

 ```BY_RECEPTION_TIMESTAMP_DESTINATIONORDER_QOS``` < ```BY_SOURCE_TIMESTAMP_DESTINATIONORDER_QOS ```

#### 3.1.2.1.3. DurabilityQosPolicy
DataWriterは、ネットワーク上にDataReaderが存在しなくても、Topic全体にメッセージを送信できる。  
DurabilityQoSPolicy は、DataReader が参加する前に Topic 上に存在していたサンプルについて、システムがどのように動作するかを定義する。 システムの動作は、DurabilityQosPolicyKind の値に依存する。  
QoS Policy データ・メンバのリスト：

|データメンバ名|型|初期値|
|---|---|---|
|kind|DurabilityQosPolicyKind|VOLATILE_DURABILITY_QOS for DataReaders <br> TRANSIENT_LOCAL_DURABILITY_QOS for DataWriters|

##### DurabilityQosPolicyKind

可能な値は4つある(DurabilityQosPolicyKind参照)：

- ```TRANSIENT_LOCAL_DURABILITY_QOS```： 新しいDataReaderが参加すると、そのHistoryは過去のサンプルで満たされる。
- ```TRANSIENT_DURABILITY_QOS```：新しいDataReaderが参加すると、そのHistoryは過去のサンプルで満たされる。
- ```PERSISTENT_DURABILITY_QOS```：新しいDataReaderが参加すると、そのHistoryは過去のサンプルで満たされる。

##### 互換性ルール
DataReaderとDataWriterのDurabilityQosPolicyが異なる種類の値を持つ場合の互換性を維持するために、DataWriterの種類はDataReaderの種類より大きいか等しくなければならない。 また、異なる種類の間の順序は次のとおり： 
```VOLATILE_DURABILITY_QOS``` < ```TRANSIENT_LOCAL_DURABILITY_QOS``` < ```TRANSIENT_DURABILITY_QOS``` < ```PERSISTENT_DURABILITY_QOS``` 

可能な組み合わせを表に示す：

|DataWriter kind|DataReader kind|Compatibility|
|---|---|---|
|VOLATILE_DURABILITY_QOS|VOLATILE_DURABILITY_QOS|Yes|
|VOLATILE_DURABILITY_QOSW||TRANSIENT_LOCAL_DURABILITY_QOS|No|
|VOLATILE_DURABILITY_QOS|TRANSIENT_DURABILITY_QOS|No|
|TRANSIENT_LOCAL_DURABILITY_QOS|VOLATILE_DURABILITY_QOS|Yes|
|TRANSIENT_LOCAL_DURABILITY_QOS|TRANSIENT_LOCAL_DURABILITY_QOS|Yes|
|TRANSIENT_LOCAL_DURABILITY_QOS|TRANSIENT_DURABILITY_QOS|No|
|TRANSIENT_DURABILITY_QOS|VOLATILE_DURABILITY_QOS|Yes|
|TRANSIENT_DURABILITY_QOS|TRANSIENT_LOCAL_DURABILITY_QOS|Yes|
|TRANSIENT_DURABILITY_QOS|TRANSIENT_DURABILITY_QOS|Yes|

##### 例

<details>
<summary>C++</summary>
```
// This example uses a DataWriter, but it can also be applied to DataReader and Topic entities
DataWriterQos writer_qos;
// The DurabilityQosPolicy is constructed with kind = VOLATILE_DURABILITY_QOS by default
// Change the kind to TRANSIENT_LOCAL_DURABILITY_QOS
writer_qos.durability().kind = TRANSIENT_LOCAL_DURABILITY_QOS;
// Use modified QoS in the creation of the corresponding entity
writer_ = publisher_->create_datawriter(topic_, writer_qos);
```
</details>

<details>
<summary>XML</summary>
```
<data_writer profile_name="writer_xml_conf_durability_profile">
    <qos>
        <durability>
            <kind>TRANSIENT_LOCAL</kind>
         </durability>
    </qos>
</data_writer>

<data_reader profile_name="reader_xml_conf_durability_profile">
    <qos>
        <durability>
            <kind>VOLATILE</kind>
        </durability>
    </qos>
</data_reader>
```
</details>

