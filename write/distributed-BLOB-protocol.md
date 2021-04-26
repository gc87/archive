# distributed-BLOB-protocol

[TOC]

## 概述

DBP（Distributed BLOB Protocol）是BigBangCore Wallet用于与外部服务进行二进制消息通信的协议。DBP协议参考[MeteorJS](https://github.com/meteor/meteor)项目的[DDP协议](https://github.com/meteor/meteor/blob/master/packages/ddp/DDP.md)进行设计，并采用Google推出的ProtoBuf作为数据交换格式。本文主要通过阐述BigBangCore Wallet与Light Wallet Service的交互来解释该协议的细节，下文中将简称BigBangCore Wallet为Core，简称Light Wallet Service为LWS。协议将支持以下两种操作：

* LWS订阅Core的一个或多个主题（topic），并且每个主题对应不同的链上数据传输，随着时间的推移Core会将链上数据的状态变化通知到LWS;
* LWS到Core的远程过程调用;

## 一般消息结构

Core与LWS通讯中将使用原始的socket来承载低级别的消息传输。其间传输的消息均为序列化为二进制的对象。每一个消息对象都包含名为`msg`的特殊字段用来指定消息类型，以及包含其他跟具体消息类型有关的字段。每条消息对应的二进制对象在传输过程中都会在头部附一个4字节（32位）的数据长度描述，用于接收端正确接收完整数据：`[len(msg)|msg]`。

~~Core和LWS必须默认忽略消息中的任何未知字段，同时~~LWS不得发送除协议中记录的字段之外的其他字段。Core端所有字段必须是可选的/可忽略的，以便与旧版本协议兼容。

### 消息类型

| 消息类型  | 发送方   | 接收方   | 说明                               | 实现优先级 |
| --------- | -------- | -------- | ---------------------------------- | ---------- |
| CONNECT   | LWS      | Core     | 建立协议连接                       | ***        |
| CONNECTED | Core     | LWS      | 协议连接成功                       | ***        |
| FAILED    | Core     | LWS      | 协议连接失败                       | ***        |
| PING      | LWS/Core | Core/LWS | ping                               | *          |
| PONG      | LWS/Core | Core/LWS | pong                               | *          |
| SUB       | LWS      | Core     | 订阅消息通道                       | ***        |
| UNSUB     | LWS      | Core     | 取消订阅消息通道                   | **         |
| NOSUB     | Core     | LWS      | 订阅失败                           | ***        |
| READY     | Core     | LWS      | 订阅成功                           | ***        |
| ADDED     | Core     | LWS      | 与主题业务相关的具体对象的新增操作 | ***        |
| CHANGED   | Core     | LWS      | 修改对象（作为预留的消息类型）     | *          |
| REMOVED   | Core     | LWS      | 删除对象（作为预留的消息类型）     | *          |
| METHOD    | LWS      | Core     | 远程过程调用请求                   | ***        |
| RESULT    | Core     | LWS      | 远程过程调用结果                   | ***        |
| ERROR     | Core     | LWS      | 顶级错误消息                       | *          |

说明：`***`-优先，`**`-其次，`*`-可选

#### Msg

`dbp.msg.proto`

```protobuf
syntax = "proto3";

package dbp;
enum Msg
{
	CONNECT = 0;
	CONNECTED = 1;
	FAILED = 2;

	PING = 3;
	PONG = 4;

	SUB = 5;
	UNSUB = 6;
	NOSUB = 7;
	READY = 8;

	ADDED = 9;
	CHANGED = 10;
	REMOVED = 11;

	METHOD = 12;
	RESULT = 13;
	ERROR = 14;
}
```

#### Base

`dbp.base.proto`

```protobuf
syntax = "proto3";

package dbp;
import "google/protobuf/any.proto";
import "dbp.msg.proto";
message Base
{
	Msg msg = 1;    
	google.protobuf.Any object = 2;
}
```

下文中出现的所有**消息类型**，实例化后均可作为Base的object动态实例。在Core与LWS的常规交互中，每一个消息对象均使用Base类型进行序列化与反序列化，接收端反序列化之后可获知确定的消息类型，并使用特定消息类型对object字段进行装箱（Pack）操作，随即提取消息意义。

## 建立协议连接

### 消息：

#### CONNECT

`dbp.connect.proto`

```protobuf
syntax = "proto3";

package dbp;
message Connect
{
    //Msg::CONNECT
    string session = 1; //用于尝试重新连接到已有的会话中
    int32 version = 2; //协议版本
    string client = 3; //客户端类别
    map<string, google.protobuf.Any> udata = 4; //用户自定义数据
}
```

#### CONNECTED

`dbp.connected.proto`

```protobuf
syntax = "proto3";

package dbp;
message Connected
{
    //Msg::CONNECTED
    string session = 1; //会话标识字符串
}
```

#### FAILED

`dbp.failed.proto`

```protobuf
syntax = "proto3";

package dbp;
message Failed
{
    //Msg::FAILED
    string reason = 1; //失败原因
    string session = 2; //会话标识字符串
    repeated int32 version = 3; //Core支持的协议版本，,可携带多个支持版本
}
```

### 过程解释：

这里约定socket底层TCP连接已经建立。

在`connect`消息体中，可通过udata数据项来扩展用户数据，数据统一以映射表的形式传入，key代表数据项的名称，value可根据需要自行设置其类型。DBP面向不同的业务，相应的用户扩展数据也不尽相同，可在**附录**部分查看扩展数据列表。

1. LWS发送session值为空的`connect`消息表示正在**新建**协议连接，在此情况下，需要尽可能的设置消息体的其他字段。当LWS发送session值不为空的`connect`消息时，表示正在**复用**该session对应的协议连接，此种情况可以不设置消息体的其他字段;

2. Core接收到`connect`消息，通过session判断属于哪种协议连接模式，如果是**新建**则通过其他消息字段设置会话信息;如果是**复用**则通过对应session值查找已有会话信息。

3. 在**新建**模式下，Core将对若干条件进行检查，检查结果通过则返回`connected`消息到LWS，检查结果未通过则返回`falied`消息到LWS。

4. 在**复用**模式下，Core如果未能找到对应的已有会话信息，则返回`falied`消息到LWS;若找到则根据会话信息还原业务过程。

5. LWS接收到`connected`消息，可提取出session，用于在出现异常时恢复会话;

6. LWS接收到`failed`消息，可提取出reason并由此判断协议连接失败的原因，修正后再次**新建**协议连接。

#### FAILED错误码

| reason | 解释                     | 解决                                     |
| ------ | ------------------------ | ---------------------------------------- |
| 001    | 不支持的协议版本         | 提取Core支持的版本，并**新建**协议连接   |
| 002    | session失效              | LWS标记session为失效，并**新建**协议连接 |
| 003    | 超过Core的最大连接数限制 | 稍后**新建**协议连接                     |
| ...... | ......                   | ......                                   |


## Heartbeats

### 消息：

#### PING

`dbp.ping.proto`

```protobuf
syntax = "proto3";

package dbp;
message Ping
{
    //Msg::PING
    string id = 1; //与pong相关的标识符
}
```

#### PONG

`dbp.pong.proto`

```protobuf
syntax = "proto3";

package dbp;
message Pong
{
    //Msg::PONG
    string id = 1; //与ping携带的标识符相同
}
```

### 过程解释：

在建立协议连接之后，Core和LWS任何一方都可以发送`ping`，发送方可以选择在消息中包含id字段。当另一方收到ping时，它必须立即响应pong消息。如果收到的ping消息包含id字段，则pong消息必须包含相同的id字段。

Core通过发送`ping`并接收对应`pong`来维护内部会话，当收到`pong`消息时Core随即更新会话的时间戳，当前系统时间戳与会话时间戳的差值可作为判断会话是否失效的标准。

## 订阅管理

### 消息：

#### SUB

`dbp.sub.proto`

```protobuf
syntax = "proto3";

package dbp;
message Sub
{
    //Msg::SUB
    string id = 1; //此订阅的LWS标识符
    string name = 2; //订阅的名称（topic）
}
```

#### UNSUB

`dbp.unsub.proto`

```protobuf
syntax = "proto3";

package dbp;
message Unsub
{
    //Msg::UNSUB
    string id = 1; //sub时传递的LWS标识符
}
```

#### NOSUB

`dbp.nosub.proto`

```protobuf
syntax = "proto3";

package dbp;
message Nosub
{
    //Msg::NOSUB
    string id = 1; //sub时传递的LWS标识符
    string error = 2; //可选，用于标识错误类型
}
```

#### READY

`dbp.ready.proto`

```protobuf
syntax = "proto3";

package dbp;
message Ready
{
    //Msg::READY
    string id = 1; //sub时传递的LWS标识符
}
```

### 过程解释：
LWS可以通过向Core发送订阅管理相关的消息，实时获取自己感兴趣的消息流。消息流中的消息包括添加，更改（预留）和删除（预留）相关的消息。通过这些消息承载了LWS应跟踪的数据集。 

1. LWS向Core发送`sub`类型的消息，消息中包含代表本次订阅的标识符以及将要订阅的主题信息（topic name），有相同标识符和相同主题名的多次订阅会被Core合并为一次订阅。

2. 当Core收到`sub`消息并检查到自己拥有对应的发布主题（topic）时则返回`ready`消息到LWS，并作记录以准备给LWS发送与该topic相关的消息。

3. 当Core检查不到自己拥有对应的发布主题时则返回`nosub`消息，不作记录，不发送与该topic相关的消息。

4. LWS向Core发送`unsub`类型的消息，用于解除之前已经订阅成功的topic，解除之后，删除相关的记录，不再发送与该topic相关的消息。

#### NOSUB错误码

| error | 解释                     | 解决                  |
| ----- | ------------------------ | --------------------- |
| 001   | 指定的topic不存在        | 检查topic后重新订阅   |
| 005   | 订阅操作错误（内部错误） | 提交bug到core开发团队 |

## 数据消息

### 消息：

#### ADDED

`dbp.added.proto`

```protobuf
syntax = "proto3";

package dbp;
import "google/protobuf/any.proto";
message Added
{
    //Msg::ADDED
    string name = 1; //主题名称
    string id = 2; //sub消息中携带的标识符
    google.protobuf.Any object = 3; //主题业务相关的具体对象
}
```

#### CHANGED

`dbp.changed.proto`

```protobuf
syntax = "proto3";

package dbp;
import "google/protobuf/any.proto";
message Changed
{
    //Msg::CHANGED
    string name = 1; //主题名称
    string id = 2; //sub消息中携带的标识符
    google.protobuf.Any object = 3; //主题业务相关的具体对象
}
```

#### REMOVED

`dbp.removed.proto`

```protobuf
syntax = "proto3";

package dbp;
import "google/protobuf/any.proto";
message Removed 
{
    //Msg::REMOVED
    string name = 1; //主题名称
    string id = 2; //sub消息中携带的标识符
    google.protobuf.Any object = 3; //主题业务相关的具体对象
}
```

### 过程解释：

当订阅操作完成，并且链上的相关数据改变时，Core便会将这些改变以数据消息的形式发送到LWS。LWS接收到数据消息的时候先确认消息的操作类型，再确认消息携带的topic类别并（通过本地与订阅操作绑定的回调函数）进行特定的处理。

站在LWS角度，每次接收到数据消息都表示链上数据已发生变化。由于是链上数据，数据本身只可能发生add不太可能发生change以及remove，所以我们主要围绕added消息类型进行设计，changed和removed作为预留的数据消息类型保留。

## 远程过程调用

### 消息：

`dbp.method.proto`

#### METHOD

```protobuf
syntax = "proto3";

package dbp;
import "google/protobuf/any.proto";
message Method 
{
    //Msg::METHOD
    string method = 1; //调用的方法名称
    string id = 2; //本次调用由LWS生成的标识符
    google.protobuf.Any params = 3; //方法的参数
}
```

#### RESULT

`dbp.result.proto`

```protobuf
syntax = "proto3";

package dbp;
import "google/protobuf/any.proto";
message Result
{
    //Msg::Ressult
    string id = 1; //method消息中携带的标识符
    string error = 2; //可选，用于标识错误信息
    repeated google.protobuf.Any result = 3; //调用返回值，类型为与调用相关的业务对象数组，长度大于等于0
}
```

### 过程解释：

远程过程调用作为协议的另一个重要组成部分，主要用于实现REQ-REP模型，同时也是LWS主动发起的与业务相关的请求的唯一方式（“订阅管理”非业务相关）。

LWS的交易数据上链，以及主动实现与链上数据的同步均可使用远程过程调用的方式标准化实现。method消息中的字段可以与http协议中的url相类比，如：

```url
/sendtransaction?data=0x00000001b&nonce=0x11&id=11010
```

，其中method对应于path。

调用执行前Core先检查是否存在对应的方法，如果存在则执行该方法并通过`result`消息返回结果到LWS。

如果不存在对应的方法或者方法内部出现异常则通过`result`消息返回错误信息到LWS。

错误信息以错误码开头，可使用横线连接错误码与错误信息说明，如：003-block read error。

#### RESULT错误码

| error | 解释                                 | 解决                           |
| ----- | ------------------------------------ | ------------------------------ |
| 001   | 调用方法不存在                       | 检查方法远程方法名称，再次调用 |
| 005   | 调用错误（内部错误）                 | 提交bug到core开发团队          |
| 002   | 参数错误                             | 检查调用参数，再次调用         |
| 003   | 业务相关错误如：003-block read error | 提交bug到core开发团队          |

## 顶级错误

### 消息：

#### ERROR

`dbp.error.proto`

```protobuf
syntax = "proto3";

package dbp;
message Error
{
    //Msg::Error
    string reason = 1; //错误原因
    string explain = 2; //详细说明
}
```

从LWS发送到Core的消息可能导致LWS收到顶级错误消息，触发的原因如下：

| error | 解释                                    | 解决       |
| ----- | --------------------------------------- | ---------- |
| 001   | 发送除CONNECT之外的任何消息作为首条消息 | 断开重连接 |
| 002   | 发送不是有效ProtoBuf对象的消息          | 断开重连接 |
| 003   | 发送未知的Msg类型                       | 断开重连接 |
| 004   | 发送的消息未包含必填字段                | 断开重连接 |



## 附录一 Light Wallet Service

### CONNECT扩展数据（udata）：

| 名称（key） | 类型（value） | 说明                              |
| ----------- | ------------- | --------------------------------- |
| forkid      | ForkID        | 以字符串列表的形式存储0-n个分支id |

#### ForkID

`lws.udata.forkid.proto`

```protobuf
syntax = "proto3";

package lws;
message ForkID 
{
    repeated string ids = 1;
}
```

### LWS主题与业务对象说明：

业务对象与**数据消息**相关，可能会被具体编码到ADDED、CHANGED、REMOVED消息体的object字段当中。

订阅主题（topic）与业务对象存在一一对应的关系，当LWS订阅了某一主题之后，就有可能会接收到**数据消息**中的多种消息体（增、删、改）。

| 订阅主题  | 数据消息 | 业务对象    |
| --------- | -------- | ----------- |
| all-block | ADDED    | Block       |
| all-tx    | ADDED    | Transaction |

#### Transaction

`lws.data.transaction.proto`

```protobuf
syntax = "proto3";

package lws;
message Transaction 
{
    uint32 nVersion = 1; //版本号,目前交易版本为 0x0001
    uint32 nType = 2; //类型, 区分公钥地址交易、模板地址交易、即时业务交易和跨分支交易
    uint32 nTimeStamp = 3; //交易产生时间
    uint32 nLockUntil = 4; //交易冻结至高度为 nLockUntil 区块
    bytes hashAnchor = 5; //交易有效起始区块 HASH
    message CTxIn
    {
        bytes hash = 1;
        uint32 n = 2;
    }
    repeated CTxIn vInput = 6; //前序交易输出列表,包含前序交易 ID 和输出点序号
    message CDestination
    {
        enum PREFIX
        {
            PREFIX_NULL = 0;
            PREFIX_PUBKEY = 1;
            PREFIX_TEMPLATE = 2;
            //PREFIX_MAX = 3;
        }
        uint32 prefix = 1;
        bytes data = 2;
        uint32 size = 3; //设置为33
    }
    CDestination cDestination = 7; //输出地址
    int64 nAmount = 8; //输出金额
    int64 nTxFee = 9; //网络交易费
    bytes vchData = 10; //输出参数(模板地址参数、跨分支交易共轭交易)
    bytes vchSig = 11; //交易签名
    bytes hash = 12; //当前tx hash
    int64 nChange = 13; //余额 
}
```

#### Block

`lws.data.block.proto`

```protobuf
syntax = "proto3";

package lws;
import "lws.data.transaction.proto";
message Block
{
    uint32 nVersion = 1; //版本号，目前区块版本为 0x0001
    uint32 nType = 2; // 类型,区分创世纪块、主链区块、业务区块和业务子区块
    uint32  nTimeStamp = 3; //时间戳，采用UTC秒单位
    bytes hashPrev = 4; //前一区块的hash
    bytes hashMerkle = 5; //Merkle tree的根
    bytes vchProof = 6;  //用于校验共识合法性数据
    Transaction txMint = 7; // 出块奖励交易
    repeated Transaction vtx = 8; //区块打包的所有交易
    bytes vchSig = 9; //区块签名
    uint32 nHeight = 10; //区块高度
    bytes hash = 11; //当前区块hash
}
```

### LWS远程过程调用说明

| 远程函数        | 参数对象     | 返回值对象  |
| --------------- | ------------ | ----------- |
| getblocks       | GetBlocksArg | Block       |
| gettransaction  | GetTxArg     | Transaction |
| sendtransaction | SendTxArg    | SendTxRet   |

#### GetBlocksArg

`lws.arg.getblocks.proto`

```protobuf
syntax = "proto3";

package lws;
message GetBlocksArg 
{
    bytes hash = 1; //起始块hash
    int32 number = 2; //将要获取的最大块数
}
```

#### GetTxArg

`lws.arg.gettransaction.proto`

```protobuf
syntax = "proto3";

package lws;
message GetTxArg
{
    bytes hash = 1; //txid
}
```

#### SendTxArg

`lws.arg.sendtransaction.prot`

```protobuf
syntax = "proto3";

package lws;
message SendTxArg
{
    bytes data = 1; //上链的tx数据
}
```

#### 上链数据解析

上链数据项目与**Transaction**对象字段相一致。

| 序号        | 数据项        | 类型            | 传输大小         | 说明                                        |
| ----------- | ------------- | --------------- | ---------------- | ------------------------------------------- |
| 1           | nVersion      | uint16          | 2 byte           | 版本号                                      |
| 2           | nType         | uint16          | 2 byte           | 类型                                        |
| 3           | nTimeStamp    | uint32          | 4 byte           | 交易创建时间戳                              |
| 4           | nLockUntil    | uint32          | 4 byte           | 冻结高度                                    |
| 5           | hashAnchor    | uint256         | 32 byte          | 有效交易起始hash                            |
| 6`**`       | size0         | uint8           | 1 byte           | 序列长度，第6项数据为vInput序列             |
| ...         |               | uint8           | 0~8 byte         |                                             |
| 7（vInput） | hash + n      | uint256 & uint8 | size0 * (32 + 1) | 序列中单项的分布，包括前序输入的hash和索引n |
| 8（sendTo） | prefix + data | uint8 & uint256 | 1 + 32 byte      | 目标地址的前缀（标识地址类型） + 地址       |
| 9           | nAmount       | int64           | 8 byte           | 交易金额                                    |
| 10          | nTxFee        | int64           | 8 byte           | 交易费                                      |
| 11`**`      | size1         | uint8           | 1 byte           | 序列长度，第11项数据为字节序列              |
| ...         |               | uint8           | 0~8 byte         |                                             |
| 12          | vchData       | bytes           | size1 * 1 byte   | 交易携带的数据                              |
| 13`**`      | size2         | uint8           | 1 byte           | 序列长度，第13项数据为字节序列              |
| ...         |               | uint8           | 0~8 byte         |                                             |
| 14          | vchSig        | bytes           | size2 * 1 byte   | 签名                                        |

`**`：size（n）标识相邻后续字段的数据项的数目（size0标识vInput数目，size1、size2分别标识vchData和vchSig的存储字节数），先读取1 byte，若小于253时 ，则该序列长度即为该值; 若等于253则继续读取后2 byte，将后2 byte的数值与253相加则为该序列的长度;若等于254则继续读取后4 byte，将后4 byte的数值与254相加则为该序列的长度;若等于255则继续读取后8 byte，将后8 byte的数值与255相加则为该序列的长度。

字节序列示意：`2 byte "nVersion"|2 byte "nType"|4 byte "nTimeStamp"|4 byte "nLockUntil"|32 byte "hashAnchor"|1～9 byte "size0"|32 byte "hash"|1 byte "n"|......|1 byte "prefix"|32 byte "data"|8 byte "nAmount"|8 byte "nTxFee"|1～9 byte "size1"|1 byte|......|1～9 byte "size2"|1 byte|......`

实例，从模板地址`20g0cmxw8jx1yjerwxdcr96awkk5g4rb82secma96dh565xrfrbh35yg5` 发送1个token到钱包地址`1v4zhq1f93kpmgtxsdg2dzk4c7zs33w2j50wfyw5vtpp8mvwbdg4gcq9c`的已签名hex：`01000000000000004cee0137f785fb006e58ff1b9295dc00df21c10686dc25f39076641dacb1158a01fa03c95ed1ff4b3361e1363f86086c4705a29c14b366a6d2b434792bb15020f50101d93f1b85e91ced486bb96c04dfcc8c3ff231f0522838ff70bbd5ac8a6f8b6c0940420f0000000000640000000000000000810eedd2ff9edde5c8317a66817e4c52191cad131f9bd611dd998ea95779d061ac01d93f1b85e91ced486bb96c04dfcc8c3ff231f0522838ff70bbd5ac8a6f8b6c090cacf1b3dc268eb041da1a73d5de5f5c97902cd79723cf8365fd2e7cfdaf6b370fd8ef8ccccf81c2a6124379ca224fbdc56acddd211fcb90b507ed6a6a731c09`

#### SendTxRet

`lws.ret.sendtransaction.proto`

```protobuf
syntax = "proto3";

package lws;
message SendTxRet
{
    bytes hash = 1; //上链数据的hash
    string result = 2; //处理结果 succeed/failed/processing
    string reason = 3; //错误原因
}
```



## 附录二 超级节点

**本节内容请结合「FnFn技术白皮书」分布式超级节点部分的内容理解**。

### 结构说明

**超级节点的定义**：由DPOS Node和Fork Nodes所构成的集群。DPOS Node与FnFn Nodes之间不采用DBP协议通讯，DBP协议通讯仅发生在DPOS Node与Fork Nodes之间以及上下级Fork Nodes之间。Fork Nodes同时分担了DPOS节点的计算以及存储压力。

超级节点的网络拓扑采用拓展性较强的树状结构，同时为了避免超级节点集群内的竞争出块以及Tx竞争验证问题，数据流采用分量路由方式（另一种是全量数据总线方式），即：每个上级节点携带下级的路由表，数据经过每个Fork Node时由该节点处理与之相关的分支业务并将无关分支数据往下级节点路由，从DPOS节点下发的全量数据在经过每个Fork Node之后减量。

采用树状分量路由方式的优点是：简单；缺点是：存在单点故障的可能性。

| 序号 | 发送方           | 接收方           | 数据类别       | 传输方式                |
| ---- | ---------------- | ---------------- | -------------- | ----------------------- |
| 1    | FnFn Nodes       | DPOS Node        | Inv，Block，Tx | PeerNet                 |
| 2    | DPOS Node        | FnFn Nodes       | Inv，Block，Tx | PeerNet                 |
| 3    | DPOS Node        | Fork Nodes       | Inv，Tx，Block | Socket（DBP Added消息） |
| 4    | Fork Nodes       | DPOS Node        | Inv，Tx，Block | Socket（DBP过程调用）   |
| 5    | Fork Node-Master | Fork Node-Slave  | Inv，Tx，Block | Socket（DBP Added消息） |
| 6    | Fork Node-Slave  | Fork Node-Master | Inv，Tx，Block | Socket（DBP过程调用）   |

`本节内容将只讨论涉及Socket API（DBP）的部分（上表中的3至6行）`

#### 功能点描述

1. DPOS节点通过PeerNet接收P2P网络数据，并经过虚拟PeerNet桥接DBP协议，将网络数据通路转换到Socket API；
2. DPOS本身拥有其所有后继节点的路由信息，当P2P网络传来未知分支的数据时，可以动态调用**直接次级Fork Node**处理该未知分支信息；
3. 当超级节点加入对未知分支进行专门处理的Fork Node时，需要将该Fork Node作为第2点所述的Fork Node后继节点，一旦接入则可从前序节点获得已有的子链相关数据，并且前序节点会将新收到的数据路由到新加入节点。
4. 上述从P2P网络获取的信息包括：安全主链Block、业务分支Block、业务分支子Block、已确认Tx、未确认Tx；
5. 每一个Fork Node拥有直接连接DPOS节点的能力，每一个Fork Node拥有直接连接上级Fork Node的能力。
6. 每级Fork Node有自己的分支白名单列表。
7. DPOS的所有下级节点均会获得DPOS的签名密钥，用于对自己所在分支的出块进行签名。
8. 每一个Fork Node均能独立对相应分支的Tx完成校验。
9. 每一个Fork Node均能将自身出块通知以及自身已验证的Tx通知，沿网络拓扑向上传递，并能作为中继将下级节点传递而来的Block通知和Tx通知传递到DPOS节点。
10. DPOS通过虚拟PeerNet桥接DBP协议，获取到下级节点传来的Block和Tx通知，稍作处理里之后通过PeerNet广播到P2P网络中。
11. 新出的块以及完成校验的Tx均会存储在处理节点，随时供上级节点取用。

### CONNECT扩展数据（udata）：

| 名称（key） | 类型（value） | 说明                                                        |
| ----------- | ------------- | ----------------------------------------------------------- |
| forkid      | ForkID        | 以字符串列表的形式存储0-n个分支id，包括所有下级节点的分支id |

在超级节点构架中，Fork Nodes发送`connect`消息与上级节点进行协议连接时，必须携带自己所处理的分支forkid以及所有下级节点所处理的分支forkid。这样可以保证最终Fork Nodes接入DPOS Node之后后者可以获得完整的分支列表。

#### ForkID

`sn.udata.forkid.proto`

```protobuf
syntax = "proto3";

package sn;
message ForkID 
{
    repeated string ids = 1;
}
```

### 超级节点主题与业务对象说明：

| 订阅主题  | 数据消息 | 业务对象    | 发布者           | 订阅者          |
| --------- | -------- | ----------- | ---------------- | --------------- |
| all-tx    | ADDED    | Transaction | DPOS Node        | Fork Node       |
| all-block | ADDED    | Block       | DPOS Node        | Fork Node       |
| sys-cmd   | ADDED    | SySCmd      | DPOS Node        | Fork Node       |
| tx-cmd    | ADDED    | TxCmd       | DPOS Node        | Fork Node       |
| block-cmd | ADDED    | BlockCmd    | DPOS Node        | Fork Node       |
| all-tx    | ADDED    | Transaction | Fork Node-Master | Fork Node-Slave |
| all-block | ADDED    | Block       | Fork Node-Master | Fork Node-Slave |
| sys-cmd   | ADDED    | SySCmd      | Fork Node-Master | Fork Node-Slave |
| tx-cmd    | ADDED    | TxCmd       | Fork Node-Master | Fork Node-Slave |
| block-cmd | ADDED    | BlockCmd    | Fork Node-Master | Fork Node-Slave |

`sn.data.cmd.proto`

```protobuf
syntax = "proto3";

package sn;
message SysCmd 
{
	string id = 1; //随机字符串
	bytes forkid = 2;
	int32 cmd = 3;
	repeated string arg = 4;
}
```

`sn.data.cmd.proto`

```protobuf
syntax = "proto3";

package sn;
message TxCmd
{
	string id = 1; //随机字符串
	bytes forkid = 2; //分支id
	bytes hash = 3; //tx hashid
}
```

`sn.data.cmd.proto`

```protobuf
syntax = "proto3";

package sn;
message BlockCmd
{
	string id = 1;  //随机字符串
	bytes forkid = 2; //分支id
	bytes hash = 3; //区块hash id
}
```

### PeerNet的事件对象

`sn.data.event.proto`

```protobuf
syntax = "proto3";

package sn;
message VPeerNetEvent
{
    int32 type = 1; // 事件类型
    bytes data = 2; // 虚拟PeerNet事件的二进制数据，会携带Block Tx Inv的payload二进制序列
}
```


### 超级节点远程过程调用说明：

| 远程函数  | 参数对象      | 返回值对象 | 实现者           | 调用者          |
| --------- | ------------- | ---------- | ---------------- | --------------- |
| sendevent | VPeerNetEvent | null       | DPOS Node        | Fork Node       |
| sendevent | VPeerNetEvent | null       | Fork Node-Master | Fork Node-Slave |

1. **sendevent**：当节点的虚拟PeerNet产生各种事件，需要通过该函数向上传递通知信息到DPOS节点发送到P2P网络或者如果满足自身节点的分支白名单就给自身的NetChannel进行处理。


### 超级节点的Event定义以及处理方式

#### 基本条件

- 本地分支白名单 </br>
也就是在multiverse.conf中addfork加入的分支ID

- 本地GetData列表 </br>
GetData请求向上发送的记录，当收到块时，需要与GetData中的Inv集合进行对比，在集合之外的，Block Tx向下传递。该GetData表是map((forkid, nonce), set)这样的结构，以ForkID和nonce作为Key，以Inv的set作为value

#### Active & Deactive 事件

- 由根节点的PeerNet产生, 也就是根节点连接或断开P2P网络会产生这两个事件
- 事件全量下发，不经过分支白名单判断，相当于超级节点的每个Fork Node必须通知到

#### Reward & Close 事件

- Fork Node的NetChannel产生
- 全量向上传递到Root Node，由Root Node的Virtual PeerNet传递到P2P网络

Reward是收到Block保存以后发出的奖励。如果NetChannel收到重复Block或者Tx，就会发送Close事件，Close事件有多种原因，有DDOS，超时，网络错误等几种原因

#### 下行 Subscribe & Unsubscribe 事件

- 由Root Node的PeerNet接收产生
- 根据每级的Fork Node的分支白名单判断Fork Node是否支持订阅和解订阅的分支，一旦由对应的分支，通过本地的Virtual PeerNet送入本地NetChannel进行处理

这两个事件的意义是P2P网络对端节点向超级节点订阅或解订阅对端所关注的分支

#### 上行 Subscribe & Unsubscribe 事件

- 由Fork Node的NetChannel根据配置文件addfork的分支ID产生
- 全量发送至Root Node，由Root Node向P2P网络订阅或解订阅
- 由于是广播消息,要考虑不同 Peer 重复计数，每级节点以计数器方式记录本地和下级节点分支订阅, Subscribe 和 Unsubscribe 分别对应计数器增减。当分支订阅计数器由非 0 减为 0 时,向上级节点构造发送 Unsubscribe ;当分支订阅计数器由 0 增加为 1 时 , 向上级节点构造发送Subscribe

这两个事件的意义是超级节点的Fork Node向P2P网络订阅或解订阅自己所关注的分支，与下行的事件是相反的。

#### 下行 GetBlocks 事件

- 由Root Node的PeerNet 接收,检查是否在顶层节点分支白名单或Fork Node的分支白名单

- Fork Node如果有与之对应的分支就送入本地NetChannel中，如果本地白名单支持对应分支那么禁止往下级节点发送，一旦下级Fork Node也由对应的分支，那么GetBlocks返回的Inv就有冗余。只有在本地没有与之对应的分支才继续往下推送GetBlocks事件

意义是，P2P网络对端节点主动向超级节点获取块的Inv

#### 上行 GetBlocks 事件

- 由Fork Node根据分支白名单在NetChannel产生
- 收到下级Fork Node 的 GetBlocks ,先检查是否在本地分支白名单内,如在白名单内,以 nNonce 为 0xffffffffffff..ff(max value) 向
下级Fork Node回应 ; 否则向上级节点传递 GetBlocks，如果Root Node也没有对应的分支，应该向P2P网络GetBlocks

意义是，Fork Node向上级节点，或P2P网络对端节点获取块的Inv


#### 下行 Inv 事件

- 由Root Node 的 PeerNet 接收或任何一级Fork Node产生
- Inv自带分支信息，全量下推，每级Fork Node通过分支白名单检查，送入本地的NetChannel

意义是，下级Fork Node发起的GetBlocks收到的Inv，或者P2P网络的Inv广播


#### 上行 Inv 事件

- Fork Node的NetChannel产生
- 全量上传，直到Root Node向P2P网络发出去

#### 下行 GetData 事件

- 由Root Node 的 PeerNet 接收
- Fork Node如果有与之对应的分支就送入本地NetChannel中，如果本地白名单支持对应分支那么禁止往下级节点发送，一旦下级Fork Node也由对应的分支，那么GetData返回的Block Tx就有冗余。只有在本地没有与之对应的分支才继续往下推送GetData事件

意义是，P2P网络对端节点向超级节点发起GetData请求

#### 上行 GetData 事件

- 由Fork Node的NetChannel产生

- 收到下级Fork Node 的 GetData ,先检查是否在本地分支白名单内,如在白名单内,以 nNonce 为 0xffffffffffff..ff(max value) 向
下级Fork Node回应 ; 否则向上级节点传递 GetData，如果Root Node也没有对应的分支，应该向P2P网络GetData

意义是，超级节点的ForkNode向P2P网络对端节点或上级Fork Node发起GetData请求

#### 下行 Block & Tx 事件

- 由Root Node或Fork Node产生
- 检查Fork Node本地否则有对应 GetData 请求发出(包括 nNonce 和
CInv ),如有由本地 NetChannel 处理。再向下推送一份副本

- Fork NOde 的 netChannel schedule 在处理 Block 事件时,应做以下调
整,如 nNonce=0xfffffffff...fff(max value), 不判断 nNonce 匹配,也不发送Reward/Close


意义是，超级节点的ForkNode向上级ForkNode节点或P2P网络对端节点GetData所返回的数据

#### 上行 Block & Tx 事件

- Fork Node的NetChannel产生
- 全量上传，直至Root Node向P2P网络发送

意义是，P2P网络对端节点向超级节点GetData所返回的数据


## 修改记录

| 日期       | 版本 | 内容                                     | 修改人       |
| ---------- | ---- | ---------------------------------------- | ------------ |
| 2018-08-09 | 0.1  | pre1                                     | Gao Chun     |
| 2018-08-13 | 0.2  | pre2                                     | Gao Chun     |
| 2018-08-19 | 0.3  | pre3 更改为proto描述，加入业务对象描述   | Gao Chun     |
| 2018-10-11 | 1    | 完整支持LWS                              | Gao Chun     |
| 2018-10-11 | 1.1  | pre1 添加超级节点socket api              | Gao Chun     |
| 2018-11-01 | 1.2  | pre2 超级节点协议第二种方案              | Gao Chun     |
| 2018-11-28 | 1.3  | pre3 超级节点虚拟Peernet相关的Event定义  | Shaohan Chen |
| 2018-12-25 | 1.4  | pre4 超级节点PeerNet事件处理方式以及定义 | Shaohan Chen |

*说明：实际的协议实现过程中应采用协议的整数版本。*