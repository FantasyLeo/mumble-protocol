Protocol stack (TCP) 协议栈（TCP)
=================================

Mumble has a shallow and easy to understand stack. Basically it
uses Google's Protocol Buffers [#f1]_ with simple prefixing to
distinguish the different kinds of packets sent through an TLSv1
encrypted connection. This makes the protocol very easily expandable.

Mumble 的协议栈是浅显易懂的。基本上，它使用了谷歌的数据序列化反序列化
工具（Google'Protocol Buffers, protobuf）[#f1]_，使用简单的前缀区分
不同种类的、通过TLSv1加密连接发送来的数据包。这使得协议非常容易地被扩展。

.. _mumble-packet:

.. figure:: resources/mumble_packet.png
   :alt: Mumble packet
   :align: center

   Mumble packet （Mumble 数据包）

The prefix consists out of the two bytes defining the type of the packet
in the payload and 4 bytes stating the length of the payload in bytes
followed by the payload itself. The following packet types are available
in the current protocol and all but UDPTunnel are simple protobuf messages.
If not mentioned otherwise all fields outside the protobuf encoding are big-endian.

前缀由6字节，两部分组成：2个字节（B)定义了在有效载荷（payload）中的数据包的类型；
另外4个字节标明了紧随其后的有效载荷数据字节长度。下面的包类型是目前协议中可用的，
且除了UDPTunnel类型的包外，其他都是简单的protobuf消息包。除非特殊说明，否则所有的
protobuf编码都是二进制大端法。

.. table:: Packet types 包类型

   +---------+------------------------+----------------+
   | Type    | Payload                |comment 说明    |
   +=========+========================+================+
   | 0       | Version                |版本            |
   +---------+------------------------+----------------+
   | 1       | UDPTunnel              |                |
   +---------+------------------------+----------------+
   | 2       | Authenticate           |鉴权（验证）    |
   +---------+------------------------+----------------+
   | 3       | Ping                   |                |
   +---------+------------------------+----------------+
   | 4       | Reject                 |拒绝包          |
   +---------+------------------------+----------------+
   | 5       | ServerSync             |服务同步包      |
   +---------+------------------------+----------------+
   | 6       | ChannelRemove          |删除频道包      |
   +---------+------------------------+----------------+
   | 7       | ChannelState           |频道状态包      |
   +---------+------------------------+----------------+
   | 8       | UserRemove             |删除用户包      |
   +---------+------------------------+----------------+
   | 9       | UserState              |用户状态包      |
   +---------+------------------------+----------------+
   | 10      | BanList                |封禁列表包      |
   +---------+------------------------+----------------+
   | 11      | TextMessage            |文本消息包      |
   +---------+------------------------+----------------+
   | 12      | PermissionDenied       |权限拒绝包      |
   +---------+------------------------+----------------+
   | 13      | ACL                    |                |
   +---------+------------------------+----------------+
   | 14      | QueryUsers             |询问用户包      |
   +---------+------------------------+----------------+
   | 15      | CryptSetup             |加密设置包      |
   +---------+------------------------+----------------+
   | 16      | ContextActionModify    |                |
   +---------+------------------------+----------------+
   | 17      | ContextAction          |                |
   +---------+------------------------+----------------+
   | 18      | UserList               |用户列表        |
   +---------+------------------------+----------------+
   | 19      | VoiceTarget            |音频目标        |
   +---------+------------------------+----------------+
   | 20      | PermissionQuery        |权限询问        |
   +---------+------------------------+----------------+
   | 21      | CodecVersion           |                |
   +---------+------------------------+----------------+
   | 22      | UserStats              |用户状态集合    |
   +---------+------------------------+----------------+
   | 23      | RequestBlob            |                |
   +---------+------------------------+----------------+
   | 24      | ServerConfig           |服务配置        |
   +---------+------------------------+----------------+
   | 25      | SuggestConfig          |建议配置        |
   +---------+------------------------+----------------+

For raw representation of each packet type see the attached Mumble.proto [#f2]_ file.

每一行代码了一种包的类型，详情见Mumble.proto[#f2]_文件。


..      rubric:: Footnotes

.. [#f1] https://github.com/google/protobuf
.. [#f2] https://raw.github.com/mumble-voip/mumble/master/src/Mumble.proto
