# paho-mqtt开发笔记

根据网络资料整理，链接如下：

[PahoMQTT-c：异步模式下各回调函数的使用场景 - 简书 (jianshu.com)](https://www.jianshu.com/p/d1a718b11232)

## 回调函数分析

### 1. MQTTAsync_connected—建立连接

```c
typedef void MQTTAsync_connected(void* context, char* cause);
```

说明：

* 每一次SDK与云平台成功连接（收到CONNACK帧且校验通过）后都会调用该回调函数，包括**用户主动建立**和**SDK自动建立**，通过形参cause确定是自动还是手动；
* 手动调用MQTTAsync_connect()函数时，因为配置了onSuccess回调函数，所以**此时本函数是冗余的**；
* 在配置自动连接的情况下，若SDK*在后台自动重连成功则会调用该函数*以提示用户进行后续必要的操作。

形参：

* context：用户数据指针；
* cause：连接成功的原因，“connect onSuccess called”表示手动连接，“automatic reconnect”表示自动连接。

返回值：

无

### 2. MQTTAsync_disconnected—断开连接

```c
typedef void MQTTAsync_disconnected(void* context, MQTTProperties* properties, enum MQTTReasonCodes reasonCode);
```

说明：

* 只有当服务端主动断开时（即收到服务端发来的DISCONN帧）后，才会调用该函数以通知用户，**其他任何断开连接的情况下都不会调用该函数**。

形参：

* context：用户数据指针；
* properties：DISCONN帧的属性字段，只在MQTT V5版本有效，可通过该字段增加用户自定义数据；
* reasonCode：连接断开的原因代码，详见MQTT协议规定。

返回值：

无

### 3. MQTTAsync_connectionLost—连接丢失

```c
typedef void MQTTAsync_connectionLost(void* context, char* cause);
```

说明：

- 客户端必须调用MQTTAsync_setCallbacks()注册该连接丢失函数，以获得SDK返回的连接丢失异步通知并进行对应处理，如重新连接或报告错误等；
- 用户主动调用MQTTAsync_disconnect()释放连接的操作，不会触发该函数调用，*只有系统内部检测连接异常时关闭连接并调用该函数*；
- 调用这个函数的线程是MQTTAsync_sendThread()。

形参：

* context：用户数据指针；
* cause：连接丢失原因，目前代码里为NULL，没有传递任何信息。

返回值：

无

### 4. MQTTAsync_messageArrived—消息到达

```c
typedef int MQTTAsync_messageArrived(void* context, char* topicName, int topicLen, MQTTAsync_message* message);
```

说明：

* 客户端必须调用MQTTAsync_setCallbacks()注册该消息到达异步处理函数，以获取服务端发过来的PUBLISH帧；
* 在客户端实现该消息处理函数时，若处理成功务必在返回1之前执行语句“MQTTClient_freeMessage(&message); MQTTClient_free(topicName);”以释放消息和topic，若失败则无需执行上述语句并返回0，*为避免消息积压建议无条件执行释放语句并返回1，在回调函数外面处理异常情况*；
* SDK并未保存任何一个订阅过的topic字符串，即任何一个PUBLISH帧到达后都会调用该函数以传递消息给用户，且**未订阅的消息也可通过该函数下发**；
* 收到的**所有PUBLISH帧（且只是PUBLISH帧）共用该回调函数**，用户需在回调函数内部区分各topic并做对应处理；
* 执行这个函数的线程是MQTTAsync_receiveThread()。

形参：

* context：用户数据指针；
* topicName：收到的消息topic；
* topicLen：消息Topic的有效长度，若为0表示通过strlen(topicName)即可获得topic有效长度，非0则表示topicName中不止一个NULL字符，需通过topicLen字段获取实际长度；
* message：消息结构体指针，包含收到的消息payload和属性字段。

返回值：

* 处理成功与否的标记，*1表示用户程序已顺利收到消息并释放topic和消息结构；0表示处理失败SDK会存储该消息并下次继续尝试调用该函数*。

### 5. MQTTAsync_deliveryComplete—发送完成

```c
typedef void MQTTAsync_deliveryComplete(void* context, MQTTAsync_token token);
```

说明：

* 客户端必须调用MQTTAsync_setCallbacks()注册该发送完成异步处理函数，以获取发送完成通知；
* SDK调用该函数说明客户端PUBLISH的帧QoS=1 / 2，同时收到了服务端的PUBACK / PUBCOMP，当客户端PUBLISH的帧QoS = 0时，该函数永远你不会被调用；
* SDK调用该函数说明从通信角度已经成功PUBLISH到服务端（因收到了回复），但它不对回复的错误码进行校验，哪怕ACK中携带是错误或无权限信息，该函数依然被SDK调用来通知用户底层发送操作已完成，真正发送成功与否需要关注每个PUBLISH操作的onSuccess/onFailure函数（见下文）；
* 发送完成事件对应全局所有的PUBLISH帧（QoS不为0），由token参数确定具体某一帧的发送完成事件；
* 执行这个函数的线程是MQTTAsync_receiveThread()。

形参：

* context：用户数据指针；
* token：表征发送完成的帧token，该token在调用函数MQTTAsync_send() 或MQTTAsync_sendMessage()返回得到。

返回值：

无

### 6. MQTTAsync_onSuccess/5—操作成功，MQTTAsync_onFailure/5—操作失败

```c
typedef void MQTTAsync_onSuccess(void* context, MQTTAsync_successData* response);
typedef void MQTTAsync_onSuccess5(void* context, MQTTAsync_successData5* response);
typedef void MQTTAsync_onFailure(void* context,  MQTTAsync_failureData* response);
typedef void MQTTAsync_onFailure5(void* context,  MQTTAsync_failureData5* response);
```

说明：

* 分别是MQTT和MQTT V5版本的操作成功函数，通常用到API调用中，主要包括建立连接、断开连接、发布、订阅、取消订阅等；
* connect操作传递的onSuccess函数，只在第一次连接成功时调用该回调函数，SDK内部自动重连时不会调用；
* publish操作传递的onSuccess函数，表示逻辑层的发送成功（无需回复或收到的回复帧中错误码为正常），注意与MQTTAsync_deliveryComplete()区分，而且对应到每一个单独的msg，每个PUBLISH的onSuccess函数都是独立的；
* subscribeMany / unsubscribeMany操作中传递的MQTTAsync_successData5 / MQTTAsync_failureData5中，需注意如果一次订阅 / 释放订阅超过1个topic时，成功或失败都会通过onSuccess5的MQTTAsync_successData5结构中的“reasonCodeCount, reasonCodes”向用户传递多个topic的操作结果，只有1个topic时成功调用onSuccess5失败调用onFailure5。

形参：

* context：用户数据指针；
* response：表征操作结果的结构体指针。

返回值：

无



## 链接字段整理

### 1. cleansession

This is a boolean value. The cleansession setting controls the behaviour of both the client and the server at connection and disconnection time.
这是一个布尔值。 cleansession 设置控制客户端和服务器在连接和断开连接时的行为。

The client and server both maintain session state information. This information is used to ensure "at least once" and "exactly once" delivery, and "exactly once" receipt of messages. Session state also includes subscriptions created by an MQTT client. 
客户端和服务器都维护会话状态信息。 此信息用于确保“至少一次”和“恰好一次”传递以及“恰好一次”接收消息。 会话状态还包括由 MQTT 客户端创建的订阅。

You can choose to maintain or discard state information between sessions.
您可以选择在会话之间保持或丢弃状态信息。

When cleansession is true, the state information is discarded at connect and disconnect. 
当cleansession 为true 时，状态信息在连接和断开连接时被丢弃。

Setting cleansession to false keeps the state information. 
将 cleansession 设置为 false 会保留状态信息。

When you connect an MQTT client application with MQTTAsync_connect(), the client identifies the connection using the client identifier and the address of the server. 
当您使用 MQTTAsync_connect() 连接 MQTT 客户端应用程序时，客户端使用客户端标识符和服务器地址来标识连接。

The server checks whether session information for this client has been saved from a previous connection to the server. 
服务器检查此客户端的会话信息是否已从先前与服务器的连接中保存。

If a previous session still exists, and cleansession=true, then the previous session information at the client and server is cleared. If cleansession=false, the previous session is resumed. If no previous session exists, a new session is started.
如果前一个会话仍然存在，并且cleansession=true，那么客户端和服务器上的前一个会话信息将被清除。 如果cleansession=false，则恢复先前的会话。 如果不存在先前的会话，则开始新的会话。
