# WebRTC学习资料大全



 版权声明：本文为foruok原创文章，转载请通过订阅号“程序视界”联系foruok获取授权。 https://blog.csdn.net/foruok/article/details/53005728

在学习WebRTC，找了些资料，记录一下，供以后查询。

> 有些需要翻墙才能看

# WebRTC 介绍

官网在这里：https://webrtc.org/。然后这里有一个官方的Getting Started：https://webrtc.org/start/。

Google关于WebRTC的幻灯片：

- http://io13webrtc.appspot.com/

然后是WebRTC的SPEC：

- https://www.w3.org/TR/webrtc/

WebRTC项目源码地址：https://chromium.googlesource.com/external/webrtc。

Native开发文档：https://webrtc.org/native-code/development/。

JS端的API文档：http://w3c.github.io/webrtc-pc/。

维基百科对WebRTC的介绍：https://en.wikipedia.org/wiki/WebRTC。

WebRTC工作组：https://www.w3.org/2011/04/webrtc/。

20170331补录，官方 native api 文档，很好解释了 native 的流程：https://webrtc.org/native-code/native-apis/。

20170428补录，webrtc-internals 的详细介绍：http://testrtc.com/webrtc-internals-parameters/。对应的中文翻译：https://segmentfault.com/a/1190000008178082。

# 教程

入门的，首选codelabs的Real time communication with WebRTC：

- https://codelabs.developers.google.com/codelabs/webrtc-web

然后是html5rocks上的基础教程：

- https://www.html5rocks.com/en/tutorials/webrtc/basics/

开发文档、入门教程，参考这里：

- https://developer.mozilla.org/en-US/docs/Web/Guide/API/WebRTC/Peer-to-peer_communications_with_WebRTC

这里，http://piratefsh.github.io/projects/2015/08/27/webrtc-stun-turn-servers.html，是一个人部署stun和turn server的记录。

进阶一点的，看这个，比较完整的介绍和实践：

- http://blog.mgechev.com/2014/12/26/multi-user-video-conference-webrtc-angularjs-yeoman/

还有这个，介绍如何用WebRTC一步一步实现视频会议：

- https://www.cleveroad.com/blog/webrtc-step-by-step-implementation-of-video-conference

还找到一个中文版的教程，通过WebRTC实现实时视频通信：

- [通过WebRTC实现实时视频通信（一）](http://www.gbtags.com/gb/share/3909.htm)
- [通过WebRTC实现实时视频通信（二）](http://www.gbtags.com/gb/share/3930.htm)
- [通过WebRTC实现实时视频通信（三）](http://www.gbtags.com/gb/share/3929.htm)

20170331，发现一个很棒的入门文章：

- https://hpbn.co/webrtc/

20170517, webrtchacks：https://webrtchacks.com/，有很多你意想不到的好东东。

20170524，一个贼棒的WebRTC课程，从这里订阅：https://bloggeek.me/enroll-course/。

# STUN/TURN/Signaling解决方案

WebRTC需要Signaling、STUN、TURN等Server，Google有自己的，还有很多开源的，也有收费的。

免费的：

- Signaling Server需要自己实现，如果你用Nodejs和Socket.io的话，比较容易做。https://codelabs.developers.google.com/codelabs/webrtc-web这里就有一个示例。
- Google的STUN服务器：`stun:stun.l.google.com:19302`
- restund：http://www.creytiv.com/restund.html。
- rfc5766-turn-server: https://code.google.com/p/rfc5766-turn-server。挪到这里了：https://github.com/coturn/rfc5766-turn-server/，支持STUN和TURN
- http://www.pjsip.org/, PJSIP，支持STUN、TURN、ICE。
- https://nice.freedesktop.org/wiki/，libnice，支持ICE和STUN。
- http://www.stunprotocol.org/，STUNTMAN，对应的代码，https://github.com/jselbie/stunserver
- https://sourceforge.net/projects/stun/，STUN client and server
- https://github.com/coturn/coturn，C++实现的STUN和TURN服务器，这里有一个安装指南：https://www.webrtc-experiment.com/docs/TURN-server-installation-guide.html

这里有一个WebRTC服务器搭建的文档：[http://io.diveinedu.com/2015/02/05/%E7%AC%AC%E5%85%AD%E7%AB%A0-WebRTC%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA.html](http://io.diveinedu.com/2015/02/05/第六章-WebRTC服务器搭建.html)。

收费的解决方案：

- https://www.onsip.com/blog/webrtc-server

# 开源示例

还有一些开源的示例代码，这个比较全了：

- https://github.com/webrtc/samples

对应的demo在这里（Chrome打开可以看效果）：

- https://webrtc.github.io/samples

回退一级，还有更多资源：

- https://github.com/webrtc

还有一个，SimpleWebRTC：

- https://github.com/andyet/SimpleWebRTC

还有：

- https://github.com/mgechev/angular-webrtc

嗯，接下来这个更赞，支持**多人视频会议**，开源的：

- https://github.com/jitsi，对应的演示地址https://meet.jit.si/

世界上第一个基于HTML5的SIP客户端：https://www.doubango.org/sipml5/。他们的GitHub主页：https://github.com/DoubangoTelecom/doubango。

> 2016.11.10补充

又找到一个国内的小示例，WebRTC实现网页版多人视频聊天室：http://www.cnblogs.com/FuzhePan/p/3899551.html。看样子是多人聊天室，一对一视频的实现。

还有一个WebRTC native demo：http://sourcey.com/webrtc-native-to-browser-video-streaming-example/，能够在C++的客户端和浏览器之间视频通信。

20170602，两个相关的项目： 
\- https://code.google.com/archive/p/telepresence/ 
\- https://github.com/webRTC/webRTC.io

# 在线演示

搜集了一些在线演示的示例：

- [https://apprtc.appspot.com](https://apprtc.appspot.com/)
- http://www.simpl.info/getusermedia
- https://webrtc.github.io/samples
- http://webcamtoy.com/app/
- http://www.shinydemos.com/facekat/
- http://idevelop.ro/ascii-camera/
- https://meet.jit.si/，多人的视频会议

# 围绕WebRTC的框架和服务

框架，视频通信的：

- https://github.com/webrtc/adapter，封装了浏览器差异
- https://github.com/henrikjoreteg/SimpleWebRTC，前面说过这个链接了
- https://github.com/priologic/easyrtc
- https://github.com/webRTC/webRTC.io

Peer间传递数据的：

- http://peerjs.com/
- https://github.com/peer5/sharefest

服务：

- http://www.tokbox.com/
- http://www.vline.com/

RTCMultiConnection，JS封装库和Signaling服务：

- https://github.com/muaz-khan/RTCMultiConnection 
  WebRTC broadcast：
- https://github.com/muaz-khan/WebRTC-Scalable-Broadcast

SIP.js(WebRTC and SIP signaling)：

- https://www.sipjs.com/

freeSWITCH（支持WebRTC）：

- https://freeswitch.org/
- https://www.packtpub.com/books/content/configuring-freeswitch-webrtc

webMCU（support WebRTC）：

- http://www.medooze.com/products/mcu/webrtc-support.aspx

# 背景及相关的rfc文档

- rfc5245（ICE）：http://www.faqs.org/rfcs/rfc5245.html
- rfc3264（Offer/Answer, SDP）：http://www.faqs.org/rfcs/rfc3264.html
- rfc3550（RTP）：http://www.ietf.org/rfc/rfc3550.txt
- rfc5389（STUN）：http://www.faqs.org/rfcs/rfc5389.html
- rfc7478(WebRTC use cases）：https://tools.ietf.org/html/rfc7478
- rfc7742（WebRTC Video Audio）：https://tools.ietf.org/html/rfc7742
- rfc5766（TURN）：https://www.ietf.org/rfc/rfc5766.txt
- rfc7875（WebRTC additional audio codecs）：https://tools.ietf.org/html/rfc7875
- rfc5128（P2P）：http://www.faqs.org/rfcs/rfc5128.html
- https://webrtchacks.com/a-hitchhikers-guide-to-webrtc-standardization/
- rfc6062（TURN allocation for TCP）：https://tools.ietf.org/html/rfc6062
- rfc7065（TURN URI）：https://tools.ietf.org/html/rfc7065
- rfc7064(STUN URI）：https://tools.ietf.org/html/rfc7064。

> add desc for rfc, 2017-03-06

这篇文章很清晰的介绍了STUN、TURN等网络背景，非常值得推荐：https://github.com/rainzhaojy/blogs/issues/4。还有这篇也不错：https://github.com/rainzhaojy/blogs/issues/6。作者的系列博客：https://github.com/rainzhaojy/blogs/issues。

> added, 2017-03-08

20170331，看了关于SDP的资料：

- https://tools.ietf.org/id/draft-nandakumar-rtcweb-sdp-01.html
- https://tools.ietf.org/html/draft-ietf-rtcweb-jsep-01

20170417，SDP rfc：

- http://www.faqs.org/rfcs/rfc2327.html

# 图书

- 《Real-Time Communication with WebRTC》，https://bloggeek.me/book-webrtc-salvatore-simon/
- https://bloggeek.me/best-webrtc-book/，这里介绍了5本书。

# 课程

- https://bloggeek.me/course/webrtc-architecture/

