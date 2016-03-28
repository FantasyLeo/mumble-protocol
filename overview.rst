Overview 概览
=============

Mumble is based on a standard server-client communication model. It
utilizes two channels of communication, the first one is a TCP connection
which is used to reliably transfer control data between the client and the
server. The second one is a UDP connection which is used for unreliable,
low latency transfer of voice data.

Mumble基于标准的服务端-客户端（CS)的通讯模型。它利用了两种通信通道：一种是
用于在服务端与客户端传输可靠的控制数据的TCP连接；另一种是用户传输语音数据的
不可靠的、低延时的UDP连接。

.. figure:: resources/mumble_system_overview.png
   :alt: Mumble system overview
   :align: center

   Mumble system overview   Mumble 系统概要图

Both are protected by strong cryptography, this encryption is mandatory and cannot be disabled. The TCP control channel uses TLSv1 AES256-SHA [#f1]_ while the voice channel is encrypted with OCB-AES128 [#f2]_.

两种方式都通过加密保护，加密是强制的且不能被停用。 TCP控制数据通道使用 TLSv1 AES256-SHA [#f1]_ 加密，而音频数据通道使用OCB-AES128 [#f2]_ 加密。

.. figure:: resources/mumble_crypt_types.png
   :alt: Mumble crypt types
   :align: center

   Mumble crypto types   Mumble加密类型

While the TCP connection is mandatory the UDP connection can be compensated by tunnelling the UDP packets through the TCP connection as described in the protocol description later.

虽然TCP连接是强制必须有的，但是正如后面的协议描述的那样， UDP数据包可以通过TCP连接隧道来传输，UPD连接以此得到TCP连接来的补偿。

.. rubric:: Footnotes 脚注

.. [#f1] http://en.wikipedia.org/wiki/Transport_Layer_Security
.. [#f2] http://www.cs.ucdavis.edu/~rogaway/ocb/ocb-back.htm
