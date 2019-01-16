上一章介绍了MQTT的连接，本章介绍MQTT的关闭连接，本章的核心内容：
1.Client主动关闭连接 2.Broker主动关闭连接 3.代码实践

## 3.1 Client主动关闭连接
Client主动关闭连接的流程非常简单，只需要向Broker发送一个DISCONNECT数据包就可以了。
DISCONNCT数据包没有可变头Variable header和信息体Payload。在Client发送完DISCONNECT之后，就可以关闭底层的TCP连接了，
不需要等待Broker的回复Broker也不会对DISCONNECT数据包回复。

这里读者可能有一个疑问，为什么需要在关闭TCP连接之前，发送一个和Broker没有交互的DISCONNECT数据包，而不是直接关闭底层的TCP连接？

这里涉及到MQTT协议的一个特性，Broker需要判断Client是否正常断开连接。

当Broker收到Client的DISCONNECT数据包的时候，它认为Client是正常地断开连接，那么它会丢失当前连接指定的医院信息（Will Message）。
如果Broker检测到Client连接丢失，但又没有收到DISCONNCT信息包，它会认为Client是非正常断开连接，就会向在连接的时候指定的遗愿主题（Will Topic）
发布遗愿（Will Message）。

## 3.2Broker主动关闭连接

MQTT协议规定Broker在没有收到Client的DISCONNECT数据包之前都应该保持和Client连接，只有Broker在Keep Alive的时间间隔里，没有收到Client的任何MQTT数据包的
时候会主动关闭连接。一些Broker的实现在MQTT协议上做了一些拓展，支持Client的连接管理，可以主动地断开和某个Client的连接。

Broker主动关闭连接之前不会向Client发送任何MQTT数据包，直接关闭底层的TCP连接完事了。

## 3.3 代码实践

这里用代码来展示MQTT连接的建立，和断开各种情况下的示例。

在这里我们使用Node.js的MQTT库，请确保安装Node.js,并通过 npm install mqtt --save安装MQTT库。

这里我们使用一个公共的Broker：iot.eclipse.org.

### 3.3.1 建立持久会话的连接
首先引用MQTT库：
 var mqtt=require('mqtt')
 
 然后建立连接：
   var client=mqtt.connect('mqtt://iot.eclipse.org',{
        clientId:"mqtt_sample_id_1",
        clean:false
        })
   这里通过ClientID选项指定Client Identifier,并通过Clean选项设定Clean Session为false
   代表要建立一个持久化会话的连接。
   
   接下来通过获取connect事件将CONNACK包Return Code和Session Present Flag打印出来，然后断开连接：
   
      client.on('connect',function(connack){
            console.log('return code:${connack.returnCode},sessionPresent:${connack.sessionPresent}')
            client.end()
            
 完整的代码persistent_connection.js如下：
 
      var mqtt=require('mqtt')
      var client=mqtt.connect('mqtt://iot.eclipse.org',{
           clientId:"mqtt_sample_id_1",
           clean:false
           })
 我们在终端上运行：
          node presistent_connection.js
          
 会得到以下输出：
           return code:0,sessionPresent:false
           
连接成功，因为是'mqtt_sample_id_1"的client 第一次建立连接，所以SessionPresent
为false。

再次运行 node persistent_connection.js,输出就会变成：
        return code:0,sessionPresent:true
        
 ### 3.3.2 建立非持久会话的连接
 
 我们只需要将Clean选项设为true，就可以建立一个非持久会话的连接了，完整的代码
 non_persistent_connection.js如下：
 
       var mqtt=require('mqtt')
       var client=mqtt.connect('mqtt://iot.eclipse.org',{
            clientId:"mqtt_sample_id_1",
            clean:true
            })
            client.on('connect',function(connack){
                 console.log('return code:${connack.returnCode},sessionPresent:
                 ${connack.sessionPresent}')
                 client.end()
                 })
                 
   我们在终端上运行：
                  node persistent_connection.js
   会得到以下输出：
                  return code:0,sessionPresent:false
                  
 无论运行多少次，SessionPresent都将为false。
 
 ### 3.3.3 相同的Client Identifier进行连接
 接下来 我们看一看如果两个Client使用同样的Client Identifier会发生什么，我们把代码
 稍微调整下，在连接成功的时候保持连接，然后捕获offline事件，在Client的连接将关闭的时候
 打印出来。
 
 完整的代码identifical.js如下：
    
       var mqtt = require('mqtt')
      var client = mqtt.connect('mqtt://iot.eclipse.org', {
          clientId: "mqtt_identical_1",
      })

      client.on('connect', function (connack) {
          console.log(`return code: ${connack.returnCode}, sessionPresent: ${connack.sessionPresent}`)
      })

      client.on('offline', function () {
          console.log("client went offline")
      })
然后我们打开 两个终端，分别在上面运行 node identifical.js,然后我们会看到在两个终端上
不停的地出现以下打印：
        return code: 0, sessionPresent: false
        client went offline
        return code: 0, sessionPresent: false
        client went offline
        return code: 0, sessionPresent: false
        .......
        
 在MQTT中，在两个Client使用相同的Client Identifier进行连接时，如果第二个Client连接成功，Broker会关闭
 和第一个已经连接上的Client连接。由于我们使用的MQTT库实现了断线重连的功能，所以当连接被Broker关闭时，它
 又会重新连接，结果就是这两个Client交替地把双方顶下线，我们就会看到这样的打印输出。因此在实际应用中，一定
 要保证每一个设备使用的Client Identifier是唯一的。
 
 如果你观察到一个Client不停地上线下线，那么有很大可能是Client Identifier冲突的问题。
 
 ### 3.4 小结
 在本节课中我们学习了MQTT连接关闭的过程，并且学习了连接建立和关闭的代码，下一课我们来学习发布和订阅
 的概念，实现信息在Client之间的传输。
