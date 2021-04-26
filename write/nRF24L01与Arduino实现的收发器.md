# nRF24L01与Arduino实现的收发器 

我的实验是基于一块arduino duemilanove（下文称为Du板） 和一块arduino nano v3（下文成为Na板）来实现的主从模式的消息收发。通讯模块为nRF24L01，关于arduino以及nRF24L01的参数特性就不在这里叙述了。

  程序库使用的是Mirf这个库，使用非常简便。按照下图来中Mirf对应的方式来连接电路。

![img](http://static.oschina.net/uploads/space/2015/0719/145708_IXIH_1438460.jpg)



  我的目的是让Du板可以接收pc通过串口发来的消息，转化后通过nrf发送到另一端的Na板，再由Na板转换之后，通过串口将数据发送到连接到Na板上的pc。反过来也可以进行同样的通讯过程。

![img](http://static.oschina.net/uploads/space/2015/0719/145743_gXv6_1438460.png)



  我写的arduino的程序迭代了3个重大版本，第一个版本使用了顺序的处理方式，主要是进行电路连接的正确性测试以及模块可用性的测试；第二个版本开始尝试arduino中的多任务处理，使用了protothread库（http://dunkels.com/adam/pt/），它可以设置时间片大小，进行多任务切换。这个版本中双向发送信息时出现大量丢包，Du板给Na板发送丢包，Na板给Du板发送也会掉包，并且protothread虽然小巧，但是任务调度上给我的感觉不太好，于是产生了第三个版本的程序；第三个版本多任务处理库替换成了Scoop（https://github.com/fabriceo/SCoop），这是一个功能比较完善的库，代码写起来也比较舒服，但还是会出现丢包的情况，Du板给Na板发送消息时，Na板接受的信息基本没有失真的情况，但是反过来由Na板给Du板发送消息的时候，收不到的情况非常严重。

 

  最后得出结论——与软件没有关系，是硬件的问题导致了消息失真，有可能是Na板的接口电压所致。官网上有解决办法——http://forum.arduino.cc/index.php?topic=202655.0 国内有人做过实验，用10uF——400uF的电容接在vcc到nRF之间就可以解决丢包失真的问题。

 

  我实验的所有源码都在这个github仓库中：https://github.com/gc87/windup