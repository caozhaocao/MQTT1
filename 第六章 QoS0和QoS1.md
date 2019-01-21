QoS全称为Quantity of Service，CONNECT、PUBLISH、SUBSCRIBE中都有QoS的标识，
那么MQTT提供的QoS是什么呢？本节课核心内容： 1.MQTT中的QoS等级；2.QoS0 ；3.QoS1；4.代码实践。


6.1 MQTT中的QoS等级

作为最初用在网络带宽窄、信号不稳定的环境下传输数据的协议，MQTT设计了一套保证信息稳定传输的机制，包括信息应答。存储
和重传。在这套机制下，提供了三种不同层次QoS：

*QoS0，At most once,至多一次；   
*QoS1，At least once,至少一次；   
*QoS2，Exactly once，确保只有一次。   

什么意思呢，QoS是信息的发送方（Sender）和接收方（Receiver）之间达成的一个协议：

*QoS0 代表 Sender发送一条信息，Receive最多能收到一次，也就是说Sender尽力向Receive发送信息，如果发送失败，也就算了；   
*QoS1 代表 Sender发送一条信息，Receive至少能收到一次，也就是说Sender向Receive发送信息，如果发送失败，会继续重试，直到Receive收到信息为止，
但是因为重传的原因，Receive有可能会收到重复的信息；   
*QoS2 代表 Sender发送一条信息，Receive确保能收到而且只收到一次，也就是说Sender尽力向Receive发送信息，如果发送失败，会继续重试，
直到Receive收到信息为止，同时保证Receive不会因为信息重传而收到重复的信息。   

要注意的是，QoS是Sender和Receive之间达成对的协议，不是Publisher和Subscriber之间达成
的协议。也就是Publisher发布一条QoS1的信息，只能保证Broker能至少收到一次这个信息；
至于对应的Subsciber能否至少收到一次这个信息，还要取决于Subscriber在Subscribe的时候
和Broker协商的QoS等级。

接下来我们来看一下QoS0和QoS1的机制，并讨论一下什么是QoS降级。

6.2 QoS0

QoS0是最简单的一个QoS等级了，在这个QoS登记下，Sender和Receive之间一次信息的传递流程如下：

![传递流程图](https://github.com/caozhaocao/MQTT1/blob/master/img/6.1.jpg)

Sender向Receive发送一个包含信息数据的PUBLISH包，然后不管结果如何，丢弃掉已发送的
PUBLISH包，一条信息的发送完成。

6.3 QoS1
QoS要保证信息至少达到Sender一次，所以有一个应答的机制，在QoS1登记下的Sender
和Receive的一次信息的传递流程如下。

![传递反馈流程图](https://github.com/caozhaocao/MQTT1/blob/master/img/6.2.jpg)

1.Sender向Receive发送一个带有信息数据的PUBLISH包，并在本地保存这个PUBLISH包。      
2.Receive收到PUBLISH包以后，向Sender发送一个PUBACK数据包，PUBACK数据包没有信息体（Payload）,在可变头中（Variable header）中
有一个包标识（Packet Identifier），和它收到的PUBLISH包中的PacketIdentifier一致。   
3.Sender收到PUBACK之后，根据PUBACK包中的Packet Identifier找到本地保存的PUBLISH包，然后丢弃掉，一次信息的发送完成。   
4.如果Sender在一段时间内没有收到PUBLISH包对应的PUBACK，它将该PUBLISH包的DUP标识设为1（代表是重新发送的PUBLISH包），然后重新发送PUBLISH包。
重复这个流程，直到收到PUBACK，然后执行第3步。   

6.4 代码实践
这里我们实现一个发布端和一个订阅端，可以通过命令行参数来指定发布和订阅的QoS，同时
通过捕获“packetsend”和“packetreceive”事件，将发送和接受到的MQTT数据包的类型打印出来。

完整的代码 publish_with_qos.js

    var args = require('yargs').argv;
    var mqtt = require('mqtt')
    var client = mqtt.connect('mqtt://iot.eclipse.org', {
        clientId: "mqtt_sample_publisher_2",
        clean: false
    })
    
    client.on('connect', function (connack) {
        if (connack.returnCode == 0) {
            client.on('packetsend', function (packet) {
                console.log(`send: ${packet.cmd}`)
            })
            client.on('packetreceive', function (packet) {
                console.log(`receive: ${packet.cmd}`)
            })
            client.publish("home/sample_topic", JSON.stringify({data: 'test'}), {qos: args.qos})
        } else {
            console.log(`Connection failed: ${connack.returnCode}`)
        }
    })
    
完整的Subscribe_with_qos.js

    var args = require('yargs').argv;
    var mqtt = require('mqtt')
    var client = mqtt.connect('mqtt://iot.eclipse.org', {
        clientId: "mqtt_sample_subscriber_id_2",
        clean: false
    })


    client.on('connect', function (connack) {
        if (connack.returnCode == 0) {
            client.subscribe("home/sample_topic", {qos: args.qos}, function () {
                client.on('packetsend', function (packet) {
                    console.log(`send: ${packet.cmd}`)
                })
                client.on('packetreceive', function (packet) {
                    console.log(`receive: ${packet.cmd}`)
                })
            })
        } else {
            console.log(`Connection failed: ${connack.returnCode}`)
        }
    })

在 subscribe_with_qos.js 中， Client 每次连接到 Broker 之后都会按照参数指定的 QoS 重新订阅主题，订阅成功以后才开始捕获接收和发送的数据包，
所以 Client 在连接之后，重新订阅之前收到的离线消息不会被打印出来。

我们可以通过 node publish_with_qos.js --qos=xxx 和 node subscribe_with_qos.js --qos=xxx 来运行这两个 JS 程序。

接下来我们用 4 种参数组合来运行这两 JS 程序，看看输出分别是什么。

注意：需要先运行 subscribe_with_qos.js 再运行 publish_with_qos.js，确保接收到消息可以打印出来。

6.4.1 发布使用QoS0，订阅使用QoS0

node publish_with_qos.js --qos=0 输出为：

     send:publish
     
node susbcribe_with_qos --qos=0 输出为：

     receive:publish
     
 结果显而易见，Publisher到Broker，Broker到Subscriber都是用的QoS0.
 
 6.4.2 发布使用QoS1，订阅使用QoS1
 
 node publish_with_qos.js --qos=1 输出为：
 
    send:publish
    receive:puback
    
 node subscribe_with_qos.js --qos=1 输出为：
 
    receive:publish
    send:puback
    
同样地，结果显而易见，Publisher到Broker，Broker到Subscriber都是用的QoS1.

6.4.3 发布使用QoS0，订阅使用QoS1

 node publish_with_qos.js --qos=0 输出为：
     send：publish
     
 node subscribe_with_qos.js --qos=1 输出为：
     receive：publish
     
这里就有点奇怪了，很明显Broker到Subscribe这段使用的是QoS0，和Subscribe订阅时指定的QoS不一样。
原因后面来讲。

6.4.4发布使用QoS1，订阅使用QoS0

 node publish_with_qos.js --qos=1 输出为：
 
      send:publish
      receive:puback
 
 node subscribe_with_qos.js --qos=0 输出为：

      receive:puback
      
和设定的一样，Publisher到Broker使用QoS1，Broker到Subscriber使用的QoS0.Publisher使用
QoS1发布信息，但是信息到Subsciber却是QoS.也就是说有可能无法收到信息，这种现象叫做QoS的降级（QoS Degrade）.

这里有一个很重要的计算方法，在MQTT协议中，从Broker到Subscribe这段信息传递的实际QoS等级，这两个QoS等级中的
最小那一个。

        Actual Subscribe QoS=MIN(Publish QoS,Subscribe QoS)
        
 这也就解释了“publish qos=0, subscribe qos=1”的情况下 Subscriber 的实际 QoS 为 0，以及“publish qos=1, subscribe qos=0”时出现 QoS 降级的原因。

理解了实际 Subscriber QoS 的计算方法，你才能很好地设计你系统里面 Publisher 和 Subscriber 使用的 QoS。
例如，如果你希望 Subscriber 至少收到一次 Publisher 的消息，那么你要确保 Publisher 和 Subscriber 都使用不小于 1 的 QoS 等级。
      
6.5 小结

在这一课里面，我们学习了相对比较简单的两种QoS等级，同时学习了实际QoS的计算方法，接下来
我们学习相对复杂的QoS2以及QoS的最佳实践。
      
