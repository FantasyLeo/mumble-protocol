.. _voice-data:

Voice data 语音数据
==================

Mumble audio channel is used to transmit the actual audio packets over the
network. Unlike the TCP control channel, the audio channel uses a custom
encoding for the audio packets. The audio channel is transport independent and
features such as encryption are implemented by the transport layer. Integers
above 8-bits are encoded using the `Variable length integer encoding`_.

Mumble音频通道是用来在网络中传输真实的音频数据包的。不同于TCP控制通道，音频
通道的数据包使用了自定义编码的方式。音频通道是对立与传输层的，且加密等功能是由
传输层实现的（此处翻译不准确）。超过8位的整形是使用可变长度整型编码
（`Variable length integer encoding`_）。

.. _packet-format:

Packet format 数据包格式
------------------------

The mumble audio channel packets are variable length packets that begin with an
8-bit header field which describes the packet type and target. The most
significant 3 bits define the packet type while the remaining 5 bits define the
target. The header is followed by the packet payload. The maximum size for the
whole audio data packet is 1020 bytes. This allows applications to use 1024
byte buffers for receiving UDP datagrams with the 4-byte encryption header
overhead.

Mumble音频通信包是可变长度的，在它的开头有8位头字段用来描述数据包的类型和目标。
最开始的3位定义了数据包类型，剩下的5位定义了目标。紧跟着头数据的是净负荷（实际的
与语音数据）。对于整个音频数据包最大是1020字节。这允许应用程序使用1024个字节缓冲区
来带有4字节加密头的接收UDP数据。(最后一句，翻译有待考究）

.. _Audio packet structure:
.. table:: Audio packet structure
    :class: bits8

    +-------------------------------+
    | Audio packet structure        |
    +===+===+===+===+===+===+===+===+
    | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
    +---+---+---+---+---+---+---+---+
    |  ``type`` |    ``target``     |
    +-----------+-------------------+
    |          Payload...           |
    +-------------------------------+

type
  The audio packet type. The packets transmitted over the audio channel are
  either ping packets used to diagnose the transport layer connectivity or
  audio packets encoded with different codecs. Different types are listed in
  `Audio packet types`_ table.

类型
   音频数据包类型。音频通道传输的数据包要么是用于诊断传输层连接的ping包，要么
   就是各种不同编码的音频包。不同的类型如下表所列举：

.. _Audio packet types:
.. table:: Audio packet types

   +---------+---------------+--------------------------------------------+
   | Type    |   Bitfield    | Description                                |
   +=========+===============+============================================+
   | ``0``   | ``000xxxxx``  | CELT Alpha encoded voice data              |
   +---------+---------------+--------------------------------------------+
   | ``1``   | ``001xxxxx``  | Ping packet                                |
   +---------+---------------+--------------------------------------------+
   | ``2``   | ``010xxxxx``  | Speex encoded voice data                   |
   +---------+---------------+--------------------------------------------+
   | ``3``   | ``011xxxxx``  | CELT Beta encoded voice data               |
   +---------+---------------+--------------------------------------------+
   | ``4``   | ``100xxxxx``  | OPUS encoded voice data                    |
   +---------+---------------+--------------------------------------------+
   | ``5-7`` |               | Unused                                     |
   +---------+---------------+--------------------------------------------+

target
  The target portion defines the recipient for the audio data. The two constant
  targets are *Normal talking* (``0``) and *Server Loopback* (``31``). The
  range 1-30 is reserved for whisper targets. These targets are specified
  separately in the control channel using the ``VoiceTarget`` packets. The
  targets are listed in `Audio targets`_ table.

目标
  目标部分定义了音频包的接收这。两个常量目标是 *Normal talking* (``0``) 和
  *Server Loopback* (``31``)。区间1-30 为私语目标保留字段。这些目标指定单独
  的控制通道使用``VoiceTarget``包。如下表。

  When a client registers a VoiceTarget on the server, it gives the target an
  ID. This voice target ID can be used as a target in the voice packets to send
  audio to specific users or channels. When receiving whisper-audio the server
  uses target 1 to specify the audio results from a whisper to a channel and
  target 2 to specify that the audio results from a direct whisper to the user.

  当客户端在服务器上注册一个VocieTarget时候，它将分配给目标一个ID。这个语音目标ID
  能够被作为语音数据包的一个目标。使得发送的音频转发给一个特定的用户群或者是频道。
  当接收到一个私语音频包时，服务器使用目标1来指定这个音频来自一个频道的结果，使用
  目标 2 来指定一个直接来自私语者用户的结果。（不会翻译，看不懂啊）

.. _Audio targets:
.. table:: Audio targets

   +-----------+-----------------------------------------------------+
   | Target    | Description                                         |
   +===========+=====================================================+
   | ``0``     | Normal talking                                      |
   +-----------+-----------------------------------------------------+
   | ``1-30``  | Whisper target                                      |
   |           |                                                     |
   |           | - VoiceTarget ID when sending whisper from client.  |
   |           | - 1 when receiving whisper to channel.              |
   |           | - 2 when receiving direct whisper to user.          |
   +-----------+-----------------------------------------------------+
   | ``31``    | Server loopback                                     |
   +-----------+-----------------------------------------------------+

Ping packet Ping包
~~~~~~~~~~~~~~~~~~

Audio channel ping packets are used as part of the connectivity checks on the
audio transport layer. These packets contain only varint encoded timestamp as
data.  See `UDP connectivity checks`_ section below for the logic involved in
the connectivity checks.

音频通道ping包用作检查音频传输层的连接。这个包仅仅包含了varint类型编码的时间戳
作为数据。 参见下章 `UDP连接检查`

.. _Audio transport ping packet:

.. table:: Audio transport ping packet

   +------------+-------------+----------------------------------+
   | Field      | Type        | Description                      |
   +============+=============+==================================+
   | Header     | ``byte``    | ``00100000b`` (``0x20``)         |
   +------------+-------------+----------------------------------+
   | Data       | ``varint``  | Timestamp                        |
   +------------+-------------+----------------------------------+

Header
  Common audio packet header. For ping packets this should have the value of
  0x20.

头部
  公共的音频包头部。对于Ping包这里的值应该是0x20。

Data
  Timestamp. The packet should be echoed back so the timestamp format can be
  decided by the original sender - the only limitation is that it must fit in a
  64-bit integer for the varint encoding.
  
数据
  时间戳。这个数据包将会被回显给最初的发送者，所以时间戳格式可以由发送者决定。
  唯一的限制是：它必须是64位的可变整形编码。

Encoded audio data packet 编码的音频数据包
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Encoded audio packets contain the actual user audio data for the voice
communication. Incoming audio data packets contain the common header byte
followed by varint encoded session ID of the source user and varint encoded
sequence number of the packet. Outgoing audio data packets contain only the
header byte and the sequence number of the packet. The server matches these to
the correct session using the transport layer information.

编码的音频数据包包含了用于语音交流的真实的用户音频数据。接收的音频数据包 包含了
公共的头部字节————源用户的可变编码的会话ID和数据包的可变编码的序列号。服务器
使用传输层信息来正确的匹配这些会话。

The remainder of the packet is made up of multiple encoded audio segments and
optional positional audio information. The audio segment format depends on the
codec of the whole audio packets. The audio segments contain codec
implementation specific information on where the audio segments end so the
possible positional audio data can be read from the end.

剩下的包由混合编码的音频分段和可选的音频位置信息组成。音频分段格式有整个音频包的编码
决定。The audio segments contain codec implementation specific information on where the audio segments end so the
possible positional audio data can be read from the end.

.. _Incoming encoded audio packet（刚接收的编码的音频包）:
.. table:: Incoming encoded audio packet

   +--------------------+--------------+-----------------------------------------------------------+
   | Field              | Type         | Description                                               |
   +====================+==============+===========================================================+
   | Header             | ``byte``     | Codec type/Audio target                                   |
   +--------------------+--------------+-----------------------------------------------------------+
   | Session ID         | ``varint``   | Session ID of the source user.  源用户会话ID              |
   +--------------------+--------------+-----------------------------------------------------------+
   | Sequence Number    | ``varint``   | Sequence number of the first audio data **segment**.      |
   +--------------------+--------------+-----------------------------------------------------------+
   | Payload            | ``byte[]``   | Audio payload          音频净负荷                         |
   +--------------------+--------------+-----------------------------------------------------------+
   | Position Info      | ``float[3]`` | Positional audio information    音频位置信息              |
   +--------------------+--------------+-----------------------------------------------------------+


.. _Outgoing encoded audio packet（编码的音频包）:
.. table:: Outgoing encoded audio packet

   +--------------------+--------------+-----------------------------------------------------------+
   | Field              | Type         | Description                                               |
   +====================+==============+===========================================================+
   | Header             | ``byte``     | Codec type/Audio target                                   |
   +--------------------+--------------+-----------------------------------------------------------+
   | Sequence Number    | ``varint``   | Sequence number of the first audio data **segment**.      |
   +--------------------+--------------+-----------------------------------------------------------+
   | Payload            | ``byte[]``   | Audio payload                                             |
   +--------------------+--------------+-----------------------------------------------------------+
   | Position Info      | ``float[3]`` | Positional audio information                              |
   +--------------------+--------------+-----------------------------------------------------------+

Header
  The common audio packet header
  
头部
  通用的音频包头部

Session ID
  Session ID of the user to whom the audio packet belongs.
  
会话ID
  音频数据包所属于某个用户的会话ID。

Sequence Number
  Audio data sequence number. The sequence number is used to maintain the
  packet order when the audio data is transported over unreliable transports
  such as UDP.

序列号
  音频数据序列号。该序列号用来在UDP不可靠的传输中维持数据包的秩序的。

  The sequence number might increase by more than one between subsequent audio
  packets in case the audio packets contain multiple audio segments. This
  allows the packet loss concealment algorithms to figure out how many audio
  frames were lost between two received packets.
  
  序列号可能在多个后续音频包中增加，为了防止音频包包含重复的音频分段。这使得能够
  使用数据包丢失隐藏算法（PLC）计算出在接收到的两个包之间丢失了多少音频帧。

Payload
  Audio payload. Format depends on the audio codec defined in the Header. The
  payload must be self-delimiting to determine whether the position info exists
  at the end of the packet. 

净负荷
  音频净负荷。其格式决定于头部定义的音频编码。净负荷必须被定界限，以此来判定包
  尾部是否有音频位置信息。

Position Info
  The XYZ coordinates of the audio source. In addition to sending the position
  information, the user must be using a positional plugin defined in the
  ``UserState`` message. The plugins might define different contexts which
  prevent voice communication between users in other contexts.

位置信息
  音频源的XYZ三维坐标系坐标。除了发送位置信息以外，用户必须使用``UserState``消息中
  定义的音频位置插件。插件可以定义不同的防止声音在其他环境中用户之间的通信的上下文中。
  （崩溃了，最后一句有问题）

Speex and CELT audio frames （Speex 和 CELT 音频帧）
""""""""""""""""""""""""""""""""""""""""""""""""""""

Encoded Speex and CELT audio is transported as individual encoded frames. Each
frame is prefixed with a single byte length and terminator header.

编码的Speex 和 CELT 的音频 是被作为独立的编码帧传输的。每一帧是有一个单字节长度前缀
和终结符的头部。

.. _celt-encoded-audio-data:

.. table:: CELT encoded audio data CELT （编码的音频数据）

   +---------+-------------+-----------------------------------------+
   | Field   | Type        | Description                             |
   +=========+=============+=========================================+
   | Header  | ``byte``    | length/continuation header              |
   +---------+-------------+-----------------------------------------+
   | Data    | ``byte[]``  | Encoded voice frame                     |
   +---------+-------------+-----------------------------------------+

Header
  The length of the Data field. The most significant bit (``0x80``) acts as the
  continuation bit and is set for all but the last frame in the payload. The
  remaining 7 bits of the header contain the actual length of the Data frame.

头部
    
 数据字段的长度。最高有效位（``0x80``）充当扩展位，且在净负荷中的所有的帧都是被设置除了
 最后一帧。头部剩下的7位包含了数据帧的实际长度。

  Note the length may be zero, which is used to signal the end of a voice
  transmission. In this case the audio data is a single zero-byte which can be
  interpreted normally as length of 0 with no continuation bit set.
  
  注意帧的长度可能是零，用来表示语音传输结束。这种情况下，音频数据是一个可以被正常解读的
  单零字节。

Data
  Single encoded audio frame. The encoding depends on the codec ``type`` header
  of the whole audio packet
  
数据
  单一编码的音频帧。编码格式决定于整个音频包中的头部中的``type``类型码。
  
Opus audio frames
"""""""""""""""""

Encoded Opus audio is transported as a single Opus audio frame. The frame is prefixed with a variable byte header.

.. _opus-encoded-audio-data:

.. table:: Opus encoded audio data

   +---------+-------------+-----------------------------------------+
   | Field   | Type        | Description                             |
   +=========+=============+=========================================+
   | Header  | ``varint``  | length/terminator header                |
   +---------+-------------+-----------------------------------------+
   | Data    | ``byte[]``  | Encoded voice frame                     |
   +---------+-------------+-----------------------------------------+

Header
  The length of the Data field. 16-bit variable length integer encoded length
  and terminator bit value. The varint encoding is the same as with 64-bit
  values, but only 16-bit unencoded values are allowed.

  The maximum voice frame size is 8191 (``0x1FFF``) bytes requiring the 13 least
  significant bits of the header. The 14th bit (mask: ``0x2000``) is the terminator
  bit which signals whether the packet is the last one in the voice
  transmission.

  Note: In CELT the "continuation bit" in the header defines whether there are
  more audio frames in the current packet. Opus always contains only one frame
  in the packet. In CELT the voice transmission end is signaled with a
  zero-byte CELT packet while in Opus we have a dedicated termination bit in
  the header.

Data
  The encoded Opus data.

Codecs
------

Mumble supports three distinct codecs; Older Mumble versions use Speex for low
bitrate audio and CELT for higher quality audio while new Mumble versions
prefer Opus for all audio. When multiple clients with different capabilities
communicate together the server is responsible for resolving the codec to use.
The clients should respect the server resolution if they are capable.

If the server resolves a codec a client doesn't support, that client is free to
use any codec it prefers. Usually this means the client will not be able to
decode incoming audio, but it can still send encoded audio out.

The CELT bitstream was never frozen which makes most CELT versions incompatible
with each other. The two CELT bitstreams supported by Mumble are: CELT 0.7.0
(CELT Alpha) and CELT 0.11.0 (CELT Beta). While CELT 0.7.0 should technically
be supported by most Mumble implementations, some servers might be configured
to force Opus codec for the users. Mumble has had Opus support since 1.2.4
(June 2013) so it should be safe to assume most clients in use support this
now.

Whispering
----------

Normal talking can be heard by the users of the current channel and all linked
channels as long as the speaker has Talk permission on these channels. If the
speaker wishes to broadcast the voice to specific users or channels, he may
use whispering. This is achieved by registering a voice target using the
VoiceTarget message and specifying the target ID as the target in the first
byte of the UDP packet.

UDP connectivity checks
-----------------------

Since UDP is a connectionless protocol, it is heavily affected by network
topology such as NAT configuration. It should not be used for audio
transmission before the connectivity has been determined.

The client starts the connectivity checks by sending a `Ping packet`_ to the
server. When the server receives this packet it will respond by echoing it back
to the address it received it from. Once the client receives the response from
the server it can start using the UDP transport for audio data. When the server
receives incoming audio data over the UDP transport it can switch the outgoing
audio over to UDP transport as well.

If the client stops receiving replies to the UDP pings at some point, it should
start tunneling the voice communication through the TCP tunnel as described in
the `Tunneling audio over TCP`_ below. When the server receives a tunneled
packet over the TCP connection it must also stop using the UDP for
communication. The client should still continue sending audio ping packets over
the UDP transport in case the UDP connection is restored and the communication
can be switched back to it.

Tunneling audio over TCP
------------------------

If the UDP channel isn't available the voice packets can be transmitted through
the TCP transport used for the control channel. These messages use the normal
TCP prefixing, as shown in figure :ref:`mumble-packet`: 16-bit message type
followed by 32-bit message length. However unlike other TCP messages, the audio
packets are not encoded as protocol buffer messages but instead the raw audio
packet described in `Packet format`_ should be written to the TCP socket
verbatim.

When the packets are received it is safe to parse the type and length fields
normally.  If the type matches that of the audio tunnel the rest of the message
should be processed as an UDP packet without attempting a protocol buffer
decoding.

Implementation note
~~~~~~~~~~~~~~~~~~~

When implementing the protocol it is easier to ignore the UDP transfer layer at
first and just tunnel the UDP data through the TCP tunnel. The TCP layer must
be implemented for authentication in any case. Making sure that the voice
transmission works before implementing the UDP protocol simplifies debugging
greatly.

Encryption
----------

All the packets are encrypted once during transfer. The actual encryption
depends on the used transport layer. If the packets are tunneled through TCP
they are encrypted using the TLS that encrypts the whole control channel
connection and if they are sent directly using UDP they must be encrypted using
the OCB-AES128 encryption.

Variable length integer encoding
--------------------------------

The variable length integer encoding (``varint``) is used to encode long,
64-bit, integers so that short values do not need the full 8 bytes to be
transferred. The basic idea behind the encoding is prefixing the value with a
length prefix and then removing the leading zeroes from the value. The positive
numbers are always right justified. That is to say that the least significant
bit in the encoded presentation matches the least significant bit in the
decoded presentation.  The *varint prefixes* table contains the definitions of
the different length prefixes. The encoded ``x`` bits are part of the decoded
number while the ``_`` signifies a unused bit. Encoding should be done by
searching the first decoded description that fits the number that should be
decoded, truncating it to the required bytes and combining it with the defined
encoding prefix.

See the *quint64* shift operators in
https://github.com/mumble-voip/mumble/blob/master/src/PacketDataStream.h
for a reference implementation.

.. table:: Varint prefixes

   +----------------------------------+--------------------------------------------------------+
   | Encoded                          | Decoded                                                |
   +==================================+========================================================+
   | ``0xxxxxxx``                     | 7-bit positive number                                  |
   +----------------------------------+--------------------------------------------------------+
   | ``10xxxxxx`` + 1 byte            | 14-bit positive number                                 |
   +----------------------------------+--------------------------------------------------------+
   | ``110xxxxx`` + 2 bytes           | 21-bit positive number                                 |
   +----------------------------------+--------------------------------------------------------+
   | ``1110xxxx`` + 3 bytes           | 28-bit positive number                                 |
   +----------------------------------+--------------------------------------------------------+
   | ``111100__`` + ``int`` (32-bit)  | 32-bit positive number                                 |
   +----------------------------------+--------------------------------------------------------+
   | ``111101__`` + ``long`` (64-bit) | 64-bit number                                          |
   +----------------------------------+--------------------------------------------------------+
   | ``111110__`` + ``varint``        | Negative recursive varint                              |
   +----------------------------------+--------------------------------------------------------+
   | ``111111xx``                     | Byte-inverted negative two bit number (``~xx``)        |
   +----------------------------------+--------------------------------------------------------+
