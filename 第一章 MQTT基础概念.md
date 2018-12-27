在这一课中，让我们来学习 MQTT 协议的基本概念和术语，同时也会介绍一下本课程中代码的开发环境搭建。本节课核心内容包括：

>MQTT 协议的通信模型
>MQTT Client
>MQTT Broker
>MQTT 协议数据包
##  1.1 MQTT 协议的通信模型
就像我们在之前提到的，MQTT 的通信是通过发布/订阅的方式来实现的，消息的发布方和订阅方通过这种方式来进行解耦，它们没有直接地连接，它们需要一个中间方。在 MQTT 里面我们称之为 Broker，用来进行消息的存储和转发。一次典型的 MQTT 消息通信流程如下所示：

public发布方——————>Broker----->Subscribler1 订阅1、Subscribler2 订阅2.....Subscriblern 订阅n

1.发布方将消息发送到 Broker；
2.Broker 接收到消息以后，检查下都有哪些订阅方订阅了此类消息，然后将消息发送到这些订阅方；
3.订阅方从 Broker 获取该消息。

在之后的课程里面，我们将发送方称为 Publisher，将订阅方称为 Subscriber。

##  1.2 MQTT Client
任何终端，嵌入式设备也好，服务器也好，只要运行了 MQTT 的库或者代码，我们都称为 MQTT 的 Client。Publisher 和 Subscriber 都属于 Client，Pushlisher 或者 Subscriber 只取决于该 Client 当前的状态——是在发布还是在订阅消息。当然，一个 Client 可以同时是 Publisher 和 Subscriber。

MQTT Client 库在很多语言中都有实现，包括 Android、Arduino、Ruby、C、C++、C#、Go、iOS、Java、JavaScript，以及 .NET 等。如果你要查看相应语言的库实现，可以在这里找到。

本课程中，我们主要使用 Node.js 的 MQTT Client 库来进行演示，所以需要先安装 Node.js，然后安装 MQTT Client 的 Node.js 包：

npm install mqtt -g
##  1.3 MQTT Broker
如前面所讲的，Broker 负责接收 Publisher 的消息，并发送给相应的 Subscriber，它是整个 MQTT 订阅/发布的核心。在实际应用中，一个 MQTT Broker 还应该提供以下一些功能：

>可以横向扩展，比如集群，来满足大量的 Client 接入；
>可以扩展接入业务系统；
>易于监控，满足高可用性。

我们在导读里面提到的阿里云、腾讯云、青云之类的云服务商提供的 MQTT 服务，其实就可以理解为他们提供了满足上述要求的 MQTT Broker。

在本课程中，我们使用一个公共的 MQTT Broker —— iot.eclipse.org 做演示，同时也会学习如何搭建一个 MQTT Broker。

## 1.4 MQTT 协议数据包
MQTT 协议的数据包格式非常简单，一个 MQTT 协议数据包由下面三个部分组成：

>固定头（Fixed header）：存在于所有的 MQTT 数据包中，用于表示数据包类型及对应标识，表明数据包大小；
>可变头（Variable header）：存在于部分类型的 MQTT 数据包中，具体内容由相应类型的数据包决定；
>消息体（Payload）：存在于部分 MQTT 数据包中，存储消息的具体数据。

接下来看一下固定头的格式，可变头和消息体我们将在讲解各种具体类型的 MQTT 协议数据包的时候 case by case 地讨论。
