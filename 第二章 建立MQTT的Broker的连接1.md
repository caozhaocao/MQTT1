Client在可以接受订阅信息之前，需要主动连接Broker，下面是连接的流程：
本节的内容包括：1.Client连接到Broker的流程；2.连接状态CONNECT 3.连接反馈状态CONNACK.

2.1 Client 连接到Broker的流程
![Client连接Broker](https://github.com/caozhaocao/MQTT1/blob/master/img/02-1.jpg)

2.2 CONNECT

建立连接由Client端发起，Client端首先先向Broker发送一个CONNECT数据包，数据包包括以下内容（这里省略Fixed header）.

2.2.1 可变头 Variable header

在CONNECT数据包可变头中，含有一下信息:

协议名称 Protocol Name：值固定为字符“MQTT”.

协议版本 Protocol Level:对MQTT 3.1.1来说，值为4.

用户名标识 User Name Flag：信息体重是否有用户名字段，1bit，0或者1.

密码标识 Password Flag:信息体中是否有密码字段，1bit，0或者1.

遗愿信息 Retail标识 will Retail:标识遗愿信息是否是Retail信息，1bit,0或者1，第八章会有详细的解释.

遗愿消息 QOS 标识（Will QOS）：标识遗愿消息的 QOS，2bit，0、1 或者 2，我们会在《第08课：Retained 消息和 LWT》详细讲解.

遗愿标识（Will Flag）：标识是否使用遗愿消息，1bit，0 或者 1，我们会在《第08课：Retained 消息和 LWT》详细讲解.

会话清除标识（Clean Session）：标识 Client 是否建立一个持久化的会话，1bit，0 或者 1，当 Clean Session 的标识设为 0 时，代表 Client 希望建立一个持久会话的连接，Broker 将存储该 Client 订阅的主题和未接受的消息，否则 Broker 不会存储这些数据，同时在建立连接时清除这个 Client 之前存在的持久化会话所保存的数据。我们会在《第07课：QoS2 和 OoS 的最佳实践》里面再详细讨论.

连接保活（Keep Alive）: 设置一个单位为秒的时间间隔，Client 和 Broker 之间在这个时间间隔之内需要至少有一次消息交互，否则 Client 和 Broker 会认为它们之间的连接已经断开，在《第09课：Keep Alive 和连接保活》里面我们再详细讨论.

2.2.2 消息体（Payload）
CONNECT 数据包的消息体中包含以下数据.

客户端标识符（Client Identifier）：Client Identifier 是用来标识 Client 身份的字段，在 MQTT 3.1.1 的版本中，这个字段的长度是 1 到 23 个字节，而且只能包含数字和 26 个字母（包括大小写），Broker 通过这个字段来区分不同的 Client.
所以在连接的时候，Client 应该保证它的 Identifier 是唯一的，通常我们可以使用比如 UUID，唯一的设备硬件标识，或者 Android 设备的 DEVICE_ID 等作为 Client Identifier 的取值来源.

MQTT 协议中要求 Client 连接时必须带上 Client Identifier，但是也允许 Broker 在实现时 Client Identifier 为空，这时 Broker 会为 Client 分配一个内部唯一的 Identifier.
如果你需要使用持久化会话，那就必须自己为 Client 设定一个唯一的 Identifier.

用户名（Username）：如果可变头中的用户名标识设为 1，那么消息体中将包含用户名字段，Broker 可以使用用户名和密码来对接入的 Client 进行验证，只允许已授权的 Client 接入.
注意不同的 Client 需要使用不同的 Client Identifier，但它们可以使用同样的用户名和密码进行连接.

密码（Password）：如果可变头中的密码标识设为 1，那么消息体中将包含密码字段.

遗愿主题（Will Topic）：如果可变头中的遗愿标识设为 1，那么消息体中将包含遗愿主题，当 Client 非正常地中断连接的时候，Broker 将向指定的遗愿主题中发布遗愿消息。我们会在后续的课程里面详细讨论.

遗愿消息（Will Message）：如果可变头中的遗愿标识设为 1，那么消息体中将包含遗愿消息，当 Client 非正常地中断连接的时候，Broker 将向指定的遗愿主题中发布由该字段指定的内容。我们会在后续的课程里面详细讨论.

2.3 CONNACK
当 Broker 收到 Client 的 CONNECT 数据包之后，将检查并校验 CONNECT 数据包的内容，之后回复 Client 一个 CONNACK 数据包.

CONNACK 数据包包含以下内容（这里我们略过 Fixed header）.

2.3.1 可变头（Variable header）
CONNACK 数据包的可变头中，含有以下信息.

会话存在标识（Session Present Flag）：用于标识在 Broker 上，是否已存在该 Client（用 Client Identifier 区分）的持久性会话，1bit，0 或者 1。当 Client 在连接时设置 Clean Session=1，则 CONNACK 中的 Session Present Flag 始终为 0；当 Client 在连接时设置 Clean Session=0，那么就有两种情况——如果 Broker 上面保存了这个 Client 之前留下的持久性会话，那么 CONNACK 中的 Session Present Flag 值为 1；如果 Broker 没有保存该 Client 的任何会话数据，那么 CONNACK 中的 Session Present Flag 值为 0.

Session Present Flag 这个特性是在 MQTT 3.1.1 版本中新加入的，之前的版本中并没有这个标识.

连接返回码（Connect Return code）：用于标识 Client 是 Broker 的连接是否建立成功，连接返回码有以下一些值：
Return Code	连接状态
0	连接已建立
1	连接被拒绝，不允许的协议版本
2	连接被拒绝，Client Identifier 被拒绝
3	连接被拒绝，服务器不可用
4	连接被拒绝，错误的用户名或密码
5	连接被拒绝，未授权

这里重点讲一下 Return Code 4 和 5.Return Code 4 在 MQTT 协议中的含义是 Username 和 Password 的格式不正确，但是在大部分的 Broker 实现中，在使用错误的用户名密码时，得到的返回码也是 4.所以这里我们认为 4 就是代表错误的用户名或密码.Return Code 5 一般在 Broker 不使用用户名和密码而使用 IP 地址或者 Client Identifier 进行验证的时候使用，来标识 Client 没有通过验证.
注意： Return Code 2 代表的是 Client Identifier 格式不规范，比如长度超过 23 个字符，包含了不允许的字符等（部分 Broker 的实现在协议标准上做了扩展，比如允许超过 23 个字符的 Client Identifer 等）.

2.3.2 消息体（Payload）
CONNACK 没有消息体.

当 Client 向 Broker 发送 CONNECT 数据包并获得 Return Code 为 0 的 CONNACK 包后，就代表连接建立成功，可以发布和接受消息了.

2.4 小结
本节课我们了解了 Client 连接到 Broker 的流程，接下来我们学习连接的关闭，以及 MQTT 连接建立与关闭的实例代码.

答疑与交流
为了让订阅课程的读者更快更好地掌握课程的重要知识点，我们为每个课程配备了课程学习答疑群服务，邀请作者定期答疑，尽可能保障大家学习效果。同时帮助大家克服学习拖延问题！

购买课程后，一定要添加小助手微信 GitChatty2 ，并将支付截图发给她，小助手会拉你进课程学习群.
