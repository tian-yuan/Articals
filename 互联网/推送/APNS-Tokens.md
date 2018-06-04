# 苹果APNs’ device token特性和过期更新

发表于[2014 年 3 月 24 日](http://blogs.360.cn/360qtest/2014/03/24/%e8%8b%b9%e6%9e%9capns-device-token%e7%89%b9%e6%80%a7%e5%92%8c%e8%bf%87%e6%9c%9f%e6%9b%b4%e6%96%b0/)由[schwimmer](http://blogs.360.cn/360qtest/author/schwimmer/)

APNs全名是Apple Push Notification Service。用iPhone的应该都习惯了，每次安装完一个新应用启动后，几乎都会弹出个警告框，“XXX应用”想要给您发送推送通知。这个警告框的权限申请就是为了APNs推送，用户授权后，应用提供商就可以通过APNs给用户推送消息。

APNs的工作机制简单来说可以分为两步，第一步是注册推送服务从APNs获取device token来告知应用提供商服务端，第二步是应用提供商服务端通过APNs给设备推送消息，device token是作为设备的唯一标示。
[![token_generation_2x](http://blogs.360.cn/360qtest/files/2014/03/token_generation_2x-300x198.png)](http://blogs.360.cn/360qtest/files/2014/03/token_generation_2x.png)
上图就是device token生成的一个过程。我们以第一次安装启动360儿童卫士应用为例，首先应用会弹出个警告框，请求用户允许发送推送通知，用户允许后–>儿童卫士会向系统注册推送服务，系统接到注册请求后就会自动连接APNs服务器请求获取设备令牌（即device token）–>APNs服务器生成包含device id的device token并下发给设备–>儿童卫士接受到device token，保存在本地同时发送给儿童卫士服务器，到此第一步就完成了。
[![token_trust_2x](http://blogs.360.cn/360qtest/files/2014/03/token_trust_2x-300x187.png)](http://blogs.360.cn/360qtest/files/2014/03/token_trust_2x.png)
上图就是推送消息的示图了，设备通过device token和APNs服务器保持连接状态。还以360儿童卫士为例，当孩子到家了，儿童卫士服务器就需要发到达提醒给家长。这时儿童卫士服务器就会通过device token作为目的设备标示来推送加密的到达提醒消息给APNs，APNs解密后再根据device token推送给指定设备。这样，一次推送就完成了。
了解了APNs工作机制，很明显能够看到device token在其中起了至关重要的串联指向作用。如果device token错误或缺失，推送就无法送达目标设备了。所以测试也罢开发也好，都很有必要了解一下device token的一些特性：

* 每个device token都是唯一的，只会对应一台设备。
* device token与设备系统相关（注意不是和设备绑定的！详解见后文），同一设备系统上不同应用获取的token是同一个。
* 应用卸载重新安装，获取到的device token不会变化，而且不会再弹出推送权限申请的弹窗，会自动继承前一次安装的设置信息。这个特性容易引发一些安全问题，用户卸载重新安装一个应用后，还没有登录应用，就可能接到上次登录帐号的推送消息了。我使用iPhone QQ和Skype都碰到过这种情况。客户端没有办法处理这个问题，因为被卸载时客户端是没法做出反应来通知服务器的。苹果有一个feedback的机制可以解决这个问题，苹果为每个应用程序维护了一个不断更新的推送失败的设备列表。服务端可以去定期检查并更新推送设备列表，这样能解决大部分问题，也能减少不必要的报文开销。
* 第三点客户端不能处理，但退出登录通知服务器就是客户端的工作了。用户退出登录客户端时，客户端应该告知服务器，停止对这个设备继续推送用户退出登录帐号的消息了。这点应该不算device token的特性了，是一个标准处理方法。

相信很多人都有这样一个疑问，作为一个设备推送的唯一标示，device token是否会变化或者过期呢？苹果在这点上有些含糊其辞，只是在官方文档上建议开发者在每次启动应用时应该都向APNs获取device token并上传给服务器。从这句话来看，device token是会变化的，不然不用每次启动都去获取。因为苹果官方没有给出明确的device token变化的情况，所以以下列举的都是一些前人总结的经验，主要援引了stackoverflow上关于这个问题一个回答，回答者称是和苹果的一个工程师交流及自己实验得出的结果。

* 升级系统device token有可能变化，确认的是升级到iOS5会变化，猜测是升级大的系统版本后device token会变化。
* 抹掉所有内容和设置，reset设备后，device token会变化。
* 恢复一个非本机的备份后，device token会变化。
* device token会过期，这个众说纷纭，有说是半年的，有说一年，有说两年的，不过会过期应该是确凿的。
* 备份或者恢复本机的备份，device token不会变化。

所以保险起见，按照苹果的每次启动应用时检查device token并发送到服务器是比较稳妥的做法。