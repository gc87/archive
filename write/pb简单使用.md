# pb简单使用

在 protobuf 的术语中，结构化数据被称为 Message。

```protobuf
package lm; 
message helloworld 
{ 
   required int32     id = 1;  // ID 
   required string    str = 2;  // str 
   optional int32     opt = 3;  //optional field 
}
```

一个比较好的习惯是认真对待 proto 文件的文件名。比如将命名规则定于如下`packageName.MessageName.proto`

```bash
protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/addressbook.proto
```

传输过程中需要把protobuf序列化后的数据以及其长度进行组合，其中长度作为prefix用4字节无符号整形表示并放在数据流的前端，序列化后的对象数据放在数据流的后端，然后再将数据流send出去。

https://stackoverflow.com/questions/8132647/does-protobuf-need-a-network-packet-header

https://stackoverflow.com/questions/4810026/sending-protobuf-messages-with-boostasio

https://eli.thegreenplace.net/2011/03/20/boost-asio-with-protocol-buffers-code-sample

https://stackoverflow.com/questions/11640864/length-prefix-for-protobuf-messages-in-c

https://colobu.com/2015/01/07/Protobuf-language-guide/ 

https://colobu.com/2017/03/16/Protobuf3-language-guide/