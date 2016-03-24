Establishing a connection 建立连接
==================================

This section describes the communication between the server and the client
during connection establishing, note that only the TCP connection needs
to be established for the client to be connected. After this the client
will be visible to the other clients on the server and able to send other
types of messages.

本章描述了服务端与客户端之间建立连接的通讯过程。注意对于客户端只有TCP连接
需要建立。这之后，客户端可通过服务端连通其他客户端且能够发送其他各种消息
类型。（咋翻译啊）

Connect 内容
------------

As the basis for the synchronization procedure the client has to first
establish the TCP connection to the server and do a common TLSv1 handshake.
To be able to use the complete feature set of the Mumble protocol it is
recommended that the client provides a strong certificate to the server.
This however is not mandatory as you can connect to the server without
providing a certificate. However the server must provide the client with
its certificate and it is recommended that the client checks this.

对于同步工序，客户端必须先与服务器建立TCP连接，且完成TLSv1握手。为了能够
使用完整的设置Mumble协议的特性，推荐客户端为服务器提供一个完整的证书。
当然这并不是强制的，你一样可以不提供给服务器任何证书。然而服务器必须提供给
客户端一个它的证书，且推荐客户端验证该证书。

.. figure:: resources/mumble_connection_setup.png
   :alt: Mumble connection setup
   :align: center

   Mumble connection setup  连接过程

Version exchange 版本交换
-------------------------

Once the TLS handshake is completed both sides should transmit their version
information using the Version message. The message structure is described below.

一旦TLS握手完成，两边都会使用版本消息包发送他们自己的版本信息。消息结构体描述如下：

.. table:: Version message

   +--------------------------------------+
   | Version                              |
   +===========================+==========+
   | version                   | uint32   |
   +---------------------------+----------+
   | release                   | string   |
   +---------------------------+----------+
   | os                        | string   |
   +---------------------------+----------+
   | os_version                | string   |
   +---------------------------+----------+

The version field is a combination of major, minor and patch version numbers (e.g. 1.2.0)
so that major number takes two bytes and minor and patch numbers take one byte each.
The structure is shown in figure \ref{fig:versionEncoding}. The release, os and os\_version
fields are common strings containing additional information.

Version（版本）字段由主版本、副版本和分支版本号组成（例如：1.2.0），所以主版本号占用了两个字节，
副版本号和分支版本号各占用一字节。

.. table:: Version field encoding (uint32)

   +---------------------------+----------+----------+
   | Major                     | Minor    | Patch    |
   +===========================+==========+==========+
   | 2 bytes                   | 1 byte   | 1 byte   |
   +---------------------------+----------+----------+

The version information may be used as part of the *SuggestConfig* checks, which usually
refer to the standard client versions. The major changes between these versions are listed
in table below. The *release*, *os* and *os_version* information is not interpreted in
any way at the moment.

版本信息可能被用作 *SuggestConfig* 检查的一部分，这通常要参考标准的客户端版本。主要被变更
在下表中所列举的。在目前 *release*, *os* and *os_version* 信息没有任何解释。

.. table:: Mumble version differences （mumble版本间的不同）

   +---------------+-------------------------------------------+
   | Version       | Major changes                             |
   +===============+===========================================+
   | 1.2.0         | CELT 0.7.0 codec support                  |
   +---------------+-------------------------------------------+ 
   | 1.2.2         | CELT 0.7.1 codec support                  |
   +---------------+-------------------------------------------+
   | 1.2.3         | CELT 0.11.0 codec                         |
   +---------------+-------------------------------------------+
   | 1.2.4         | Opus codec support, SuggestConfig message |
   +---------------+-------------------------------------------+

Authenticate 认证
-----------------

Once the client has sent the version it should follow this with the Authenticate message.
The message structure is described in the figure below. This message may be sent immediately
after sending the version message. The client does not need to wait for the server version
message.

一旦客户端已经发送了版本信息，接下来它将会发送认证信息。认证信息的结构如下图表所描述。在发送
版本消息之后，认证消息会被立即发送。客户端不需要等待服务器回复的版本信息。

.. table:: Authenticate message （认证信息)

   +-----------------------------------------------+
   | Authenticate                                  |
   +===========================+===================+
   | username                  | string            |
   +---------------------------+-------------------+
   | password                  | string            |
   +---------------------------+-------------------+
   | tokens (令牌、凭证）      | string            |
   +---------------------------+-------------------+

The username and password are UTF-8 encoded strings. While the client is free to accept any
username from the user the server is allowed to impose further restrictions. Furthermore
if the client certificate has been registered with the server the client is primarily
known with the username they had when the certificate was registered. For more
information see the server documentation.

用户名（username）和密码（password)是UTF-8编码字符串。While the client is free
to accept any username from the user the server is allowed to impose further restrictions.
此外，如果客户端的证书已经在该服务器注册了，则该客户端在基本是要知道他们在注册该证书时的
用户名。更多信息参考服务器文档。

The password must only be provided if the server is passworded, the client provided no
certificate but wants to authenticate to an account which has a password set, or to
access the SuperUser account.

密码必须被提供，在服务器被设置密码，客户但没有证书且想认证一个有密码的帐号，或者进入超级用户（SuperUser）
账号 时。

The third field contains a list of zero or more token strings which act as passwords
that may give the client access to certain ACL groups without actually being a
registered member in them, again see the server documentation for more information.

第三个字段包含了一个为零的链表或者多个充当密码作用的凭证字符串，那可能会给客户端一个
进入某一个ACL组，而该客户端并没有注册。详情请参考服务器文档。

Crypto setup 加密设置
---------------------

Once the Version packets are exchanged the server will send a CryptSetup packet to
the client. It contains the necessary cryptographic information for the OCB-AES128
encryption used in the UDP Voice channel. The packet is described in figure
below. The encryption itself is described in a later section.

一旦完成版本数据包的交换，服务器将会发送给客户端一个加密设置（CyrptSetup）数据包。
它包含了必要的加密算法信息，对于UDP语音通道使用的 OCB-ASE128加密。该数据包的描述如下表。
加密的描述在后面的章节中。

.. table:: CryptSetup message （加密设置信息）

   +-----------------------------------------------+
   | CryptSetup                                    |
   +===========================+===================+
   | key                       | bytes             |
   +---------------------------+-------------------+
   | client_nonce              | bytes             |
   +---------------------------+-------------------+
   | server_nonce              | bytes             |
   +---------------------------+-------------------+

Channel states 频道状态
-----------------------

After the client has successfully authenticated the server starts listing the channels
by transmitting partial ChannelState message for every channel on this server. These
messages lack the channel link information as the client does not yet have full
picture of all the channels. Once the initial ChannelState has been transmitted
for all channels the server updates the linked channels by sending new packets for
these. The full structure of these ChanneLState messages is shown below:

在客户端通过服务器的认证后（具体是谁通过是的认证啊，晕了），服务器开始传输部分信息为所有在服务器上的频道（翻译不对）。
这些信息中缺少频道链信息，以至于客户端的还不能完全绘制出所有的频道。一旦初始的频道状态
（channelState）被传输到所有的频道，服务器将为此通过发送新的包来更新链频道。完整的
频道状态信息的结构如下：

.. table:: ChannelState message

   +-----------------------------------------------+
   | ChannelState                                  |
   +===========================+===================+
   | channel_id                | uint32            |
   +---------------------------+-------------------+
   | parent                    | uint32            |
   +---------------------------+-------------------+
   | name                      | string            |
   +---------------------------+-------------------+
   | links                     | repeated uint32   |
   +---------------------------+-------------------+
   | description               | string            |
   +---------------------------+-------------------+
   | links_add                 | repeated uint32   |
   +---------------------------+-------------------+
   | links_remove              | repeated uint32   |
   +---------------------------+-------------------+
   | temporary                 | optional bool     |
   +---------------------------+-------------------+
   | position                  | optional int32    |
   +---------------------------+-------------------+


*The server must send a ChannelState for the root channel identified with ID 0.*

服务器必须发送一个ID为0的Root频道说明。

User states  用户状态
---------------------

After the channels have been synchronized the server continues by listing the
connected users. This is done by sending a UserState message for each user
currently on the server, including the user that is currently connecting.

.. table:: UserState message

   +-----------------------------------------------+
   | UserState                                     |
   +===========================+===================+
   | session                   | uint32            |
   +---------------------------+-------------------+
   | actor                     | uint32            |
   +---------------------------+-------------------+
   | name                      | string            |
   +---------------------------+-------------------+
   | user_id                   | uint32            |
   +---------------------------+-------------------+
   | channel_id                | uint32            |
   +---------------------------+-------------------+
   | mute                      | bool              |
   +---------------------------+-------------------+
   | deaf                      | bool              |
   +---------------------------+-------------------+
   | suppress                  | bool              |
   +---------------------------+-------------------+
   | self_mute                 | bool              |
   +---------------------------+-------------------+
   | self_deaf                 | bool              |
   +---------------------------+-------------------+
   | texture                   | bytes             |
   +---------------------------+-------------------+
   | plugin_context            | bytes             |
   +---------------------------+-------------------+
   | plugin_identity           | string            |
   +---------------------------+-------------------+
   | comment                   | string            |
   +---------------------------+-------------------+
   | hash                      | string            |
   +---------------------------+-------------------+
   | comment_hash              | bytes             |
   +---------------------------+-------------------+
   | texture_hash              | bytes             |
   +---------------------------+-------------------+
   | priority_speaker          | bool              |
   +---------------------------+-------------------+
   | recording                 | bool              |
   +---------------------------+-------------------+

Server sync
-----------

The client has now received a copy of the parts of the server state he
needs to know about. To complete the synchronization the server transmits
a ServerSync message containing the session id of the clients session,
the maximum bandwidth allowed on this server, the servers welcome text
as well as the permissions the client has in the channel he ended up.

For more information pease refer to the Mumble.proto file [#f1]_.

Ping
----

If the client wishes to maintain the connection to the server it is required
to ping the server. If the server does not receive a ping for 30 seconds it
will disconnect the client.

..      rubric:: Footnotes

.. [#f1] https://raw.github.com/mumble-voip/mumble/master/src/Mumble.proto
