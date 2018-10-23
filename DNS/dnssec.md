# DNSSEC 原理、配置与布署简介

Posted on [May 16, 2011](http://netsec.ccert.edu.cn/duanhx/?p=1479) by [Duan Haixin](http://netsec.ccert.edu.cn/duanhx/?author=2)

作者：段海新，清华大学信息网络工程研究中心

摘要：DNSSEC是为解决DNS欺骗和缓存污染而设计的一种安全机制。本文概要介绍DNSSEC的背景、工作原理、在BIND上的配置，最后介绍国际上的布署情况和它可能对互联网安全体系的影响。

# 1  DNSSEC的背景和目的

域名系统（Domain Name System，DNS）是一些“Too simple, Too Naive”的互联网先驱者设计的，它象互联网的其他协议或系统一样，在一个可信的、纯净的环境里运行得很好。但是今天的互联网环境异常复杂，充斥着各种欺诈、攻击，DNS协议的脆弱性也就浮出水面。对DNS的攻击可能导致互联网大面积的瘫痪，这种事件在国内外都屡见不鲜。

尽管DNS的安全问题一直被互联网研究和工程领域广为关注，但是有一种普遍存在的攻击却始终没有解决，即DNS的欺骗和缓存污染问题。DNS安全扩展（DNS Security Extension, 即DNSSEC）主要是为了解决这一问题而提出的（尽管它还有其他用途）。因此，在介绍DNSSEC的原理之前有必要简单介绍DNS欺骗和缓存污染攻击的原理。

## 1.1   DNS欺骗攻击和缓存污染

用户在用域名（www.example.com）访问某一个网站时，用户的计算机一般会通过一个域名解析服务器（Resolver Server，也称递归服务器（Recursive Server））把域名转换成IP地址。解析服务器一般需要通过查询根域名服务器（root）、顶级域名服务器（TLD）、 权威域名服务器（Authoritative Name Server），通过递归查询的方式最终获得目标服务器的IP地址，然后交给用户的计算机。在此过程中，攻击者都可以假冒应答方（根、顶级域、权威域、或解析服务器）给请求方发送一个伪造的响应，其中包含一个错误的IP地址。发送请求的用户计算机或者解析服务器很傻、很天真地接受了伪造的应答，导致用户无法访问正常网站，甚至可以把重定向到一个伪造的网站上去。由于正常的DNS解析使用UDP协议而不是TCP协议，伪造DNS的响应报文比较容易；如果攻击者可以监听上述过程中的任何一个通信链路，这种攻击就易如反掌。

更加糟糕的是，由于DNS缓存（Cache）的作用，这种错误的记录可以存在相当一段时间（比如几个小时甚至几天），所有使用该域名解析服务器的用户都无法访问真正的服务器。攻击者可能是黑客、网络管理者等等（比如利用用户的拼写错误，把对不存在域名NXDOMAIN的访问重定向到一个广告页面），但本文不去讨论这些攻击者的身份和动机。

## 1.2   DNSSEC的目标、历史和意义

上述攻击能够成功的原因是DNS 解析的请求者无法验证它所收到的应答信息的真实性，而DNSSEC给解析服务器提供了防止上当受骗的武器，即一种可以验证应答信息真实性和完整性的机制。利用密码技术，域名解析服务器可以验证它所收到的应答（包括域名不存在的应答）是否来自于真实的服务器，或者是否在传输过程中被篡改过。

尽管从原理上来说DNSSEC并不复杂，但是从1997年第一个有关DNSSEC的标准RFC 2065 [2] 发布至今已经十多年了，直到最近一两年才有了可观的进展。1999年IETF对DNSSEC标准进行了修订RFC 2535[3]，但很快被证明这次修订是失败的。直到2005年，一个可用的DNSSEC标准才制定出来（RFC 4033-4035）[4][5][6]，目前主流域名服务软件（如BIND ）实现的也是这一版本。2008年，IETF又发布了一个NSEC3（RFC5155）[7]标准，以提高DNSSEC隐私保护能力。 随着DNSSEC的推广，也许还会有一些新的问题和新的修订，DNSSEC标准仍在发展过程中。

尽管DNSSEC的目标仅限于此（即不保护DNS信息的保密性和服务的可用性），但是，DNSSEC的成功布署对互联网的安全还有其他好处，比如提高电子邮件系统的安全性，甚至把DNS作为一个公钥基础设施（PKI）。

本文所介绍的DNSSEC工作原理基于RFC 4033-4035，关于DNS工作原理、配置和数字签名技术超出了本文的范围，感兴趣的读者可以参考[8]。在此基础上简单介绍最常用的域名服务系统（BIND）的基本配置过程，最后DNSSEC在国际上的布署情况以及可能给网络安全带来的影响。

# 2  DNSSEC原理

简单的说，DNSSEC依靠数字签名保证DNS应答报文的真实性和完整性。权威域名服务器用自己的私有密钥对资源记录（Resource Record, RR）进行签名，解析服务器用权威服务器的公开密钥对收到的应答信息进行验证。如果验证失败，表明这一报文可能是假冒的，或者在传输过程、缓存过程中被篡改了。 RFC 4033概要介绍了DNSSEC所提供的安全功能并详细介绍了相关的概念，下面通过一个简化的实例介绍DNSSEC的工作原理 。

如图2所示，一个支持DNSSEC的解析服务器（RFC4033中Security-Aware Resolver）向支持DNSSEC的权威域名服务器（Security-Aware Name Server）请求域名www.test.net.时，它除了得到一个标准的A记录（包含IPv4地址）以外，还收到一个同名的RRSIG记录，其中包含test.net这个权威域的数字签名，它是用test.net.的私有密钥来签名的。为了验证这一签名的正确性，解析服务器可以再次向test.net的域名服务器查询响应的公开密钥，即名为test.net的DNSKEY类型的资源记录。然后解析服务器就可以用其中的公钥验证上述www.test.net. 记录的真实性与完整性。

[![img](http://netsec.ccert.edu.cn/duanhx/files/2011/05/DNSSEC-basic1.jpg)](http://netsec.ccert.edu.cn/duanhx/files/2011/05/DNSSEC-basic1.jpg)图2 DNSSEC基本工作原理

但是，解析服务器如何保证它所获得的test.net.返回的公开密钥（DNSKEY记录）是真实的而不是假冒的呢？ 尽管test.net.在返回DNSKEY记录的同时，也返回对这个公钥的数字签名（名为test.net的RRSIG记录）；但是，攻击者同样可以同时伪造公开密钥和数字签名两个记录而不被解析者发现。

象基于X509的PKI体系一样，DNSSEC也需要一个信任链，必须有一个或多个开始就信任的公钥（或公钥的散列值），在RFC 4033中称这些初始信任的公开密钥或散列值为“信任锚（Trust anchors）”。信任链中的上一个节点为下一个节点的公钥散列值进行数字签名，从而保证信任链中的每一个公钥都是真实的。理想的情况下（DNSSEC全部布署），每个解析服务器只需要保留根域名服务器的DNSKEY就可以了。

在上面的例子中，假设解析服务器开始并不信任test.net.的公开密钥， 它可以到test.net.的上一级域名服务器net.那里查询test.net.的DS（Delegation Signer， 即DS RR）记录，DS RR中存储的是test.net. 公钥的散列值（比如用SHA-1算法计算得到的160比特数据的16进制表示）。假设解析服务器由管理员手工配置了.net的公钥（即Trust Anchor），它就可以验证test.net.公钥（DNSKEY）的正确与否了。

## 2.1   DNSSEC的资源记录

为了实现资源记录的签名和验证，DNSSEC增加了四种类型的资源记录： RRSIG（Resource Record Signature）、DNSKEY(DNS Public Key)、DS(Delegation Signer)、NSEC(Next Secure)。前三种记录已经在上面的实例中提到了，NSEC记录是为响应某个资源记录不存在而设计的。具体的说明参见RFC 4034，本文概要介绍如下。

### 2.1.1  DNSKEY记录

DNSKEY资源记录存储的是公开密钥，下面是一个DNSKEY的资源记录的例子：

example.com. 86400  IN  DNSKEY  256  3  5  ( AQPSKmy….. aNvv4w==  )

其中256是标志（flag）字段，它是一个16比特的数，如果第7位（左起为第0位。这一位是区密钥（Zone Key）标志, 记为ZK）为1，则表明它是一个区密钥，该密钥可以用于签名数据的验证，而且资源记录的所有者（example.com.）必须是区的名字。第15位称为安全入口点（Security Entry Point，SEP）标志，将在下面介绍。

下一个字段“3”是协议（protocol）字段，它的值必须是3，表示这是一个DNSKEY，这是为了与以前版本DNSSEC兼容而保留下来的。其他的值不能用于DNSSEC签名的验证。

下一个字段“5”是算法（Algorithm）字段，标识签名所使用的算法的种类。其中常用的几种：1：RSA/MD5; 3：DSA/SHA-1; 5 RSA/SHA-1;

最后括号中的是公开密钥（Public Key）字段，它的格式依赖于算法字段。

在实践中，权威域的管理员通常用两个密钥配合完成对区数据的签名。一个是Zone-Signing Key(ZSK)，另一个是Key-Signing Key(KSK)。ZSK用于签名区数据，而KSK用于对ZSK进行签名。这样做的好处有二：

（1）用KSK密钥签名的数据量很少，被破解（即找出对应的私钥）概率很小，因此可以设置很长的生存期。这个密钥的散列值作为DS记录存储在上一级域名服务器中而且需要上级的数字签名，较长的生命周期可以减少密钥更新的工作量。

（2）ZSK签名的数据量比较大，因而破解的概率较大，生存期应该小一些。因为有了KSK的存在，ZSK可以不必放到上一级的域名服务中，更新ZSK不会带来太大的管理开销（不涉及和上级域名服务器打交道）。

### 2.1.2 RRSIG记录

RRSIG资源记录存储的是对资源记录集合（RRSets）的数字签名。下面是对一个A记录签名后得到的RRSIG记录：

> host.example.com. 86400 IN RRSIG  A  5  3  86400 20030322173103 (
>
> 20030220173103  2642  example.com.
>
> oJB1W6WNGv+ldvQ3WDG0MQkg5IEhjRip8WTr
>
> …
>
> J5D6fwFm8nN+6pBzeDQfsS3Ap3o= )

从第五个字段（“A”）开始各字段的含义如下：

类型覆盖（The Type Covered Field）：表示这个签名覆盖什么类型的资源记录，本例中是A；

算法：数字签名算法，同DNSKEY记录的算法字段；本例中5表示RSA/SHA-1。

标签数量（The Labels Field）：被签名的资源域名记录所有者（host.example.com.）中的标签数量，如本例中为3，*.example.com.为2，“.”的标签数量为0。

接下来的几个字段分别是被签名记录的TTL、有效期结束时间、开始时间。

然后（2642）是密钥标签（Key Tag），它是用对应公钥数据简单叠加得到的一个16比特整数。如果一个域有多个密钥时（如一个KSK、一个ZSK），Key Tag可以和后面的签名者字段（example.com.）共同确定究竟使用哪个公钥来验证签名。

### 2.1.3  DS记录

DS（Delegation Signer）记录存储DNSKEY的散列值，用于验证DNSKEY的真实性，从而建立一个信任链。不过，不象DNSKEY存储在资源记录所有者所在的权威域的区文件中，DS记录存储在上级域名服务器（Delegation）中，比如example.com的DS RR存储在.com的区中。

下面是一个DS记录的实例：

> dskey.example.com. 86400 IN DS 60485 5 1 ( 2BB183AF5F22588179A53B0A98631FAD1A292118 )

DS 之后的字段依次是密钥标签（Key Tag）、算法、和散列算法（1代表 SHA-1）。后面括号内的内容是dskey.example.com.密钥SHA-1计算结果的16进制表示。Example.com必须为这个记录数字签名，以证实这个DNSKEY的真实性。

### 2.1.4  NSEC记录

NSEC记录是为了应答那些不存在的资源记录而设计的。为了保证私有密钥的安全性和服务器的性能，所有的签名记录都是事先（甚至离线）生成的。服务器显然不能为所有不存在的记录事先生成一个公共的“不存在”的签名记录，因为这一记录可以被重放（Replay）；更不可能为每一个不存在的记录生成独立的签名，因为它不知道用户将会请求怎样的记录。

在区数据签名时，NSEC记录会自动生成。比如在vpn.test.net和xyz.test.net之间会插入下面的这样两条记录：

> vpn.test.net.   10800   IN A       192.168.1.100
>
> 172800  NSEC    xyz.test.net. A RRSIG NSEC
>
> 172800  RRSIG   NSEC 5 5 172800 20110611031416 (
>
> 20110512031416 5271 test.net.
>
> Ujw/aq….15dV5tF7XgWSR78= )
>
> xyz.test.net.  10800   IN A    192.168.1.200

其中NSEC记录包括两项内容：排序后的下一个资源记录的名称（xyz.test.net.）、以及vpn.test.net.这一名称所有的资源记录类型（A、RRSIG、NSEC），后面的RRSIG记录是对这个NSEC记录的数字签名。

在用户请求的某个域名在vpn和xyz之间时，如www.test.net.，服务器会返回域名不存在，并同时包括 vpn.test.net的NSEC记录。

## 2.2   DNSSEC对现有DNS协议的修改

由于新增DNS资源记录的尺寸问题，支持DNSSEC的域名服务器必须支持EDNS0（RFC2671），即允许DNS报文大小必须达到1220字节（而不是最初的512字节），甚至可以是4096字节。

DNSSEC在报文头中增加了三个标志位：

（1）DO（DNSSEC OK, 参见RFC3225）：支持DNSSEC的解析服务器在它的DNS查询报文中，必须把DO标志位置1，否则权威域服务器认为解析器不支持DNSSEC就不会返回RRSIG等记录。

（2）AD （Authentic Data）：AD是认证数据标志，如果服务器验证了DNSSEC相关的数字签名，则置AD位为1，否则为0。这一标志位一般用于自己不做验证的解析器（non-validating security-aware resolvers）和它所信任的递归解析服务器（security-aware recursive name server）之间，用户计算机上的解析器自己不去验证数字签名，递归服务器给它一个AD标志为1的响应，它就接受验证结果。这种场景只有在它们之间的通信链路比较安全的情况下才安全，比如使用了IPSEC和TSIG[]。

（3）CD （Checking Disabled）： 关闭检查标志位用于支持DNSSEC验证功能的解析器（validating security-aware resolver）和递归域名服务器之间，解析器在发送请求时把CD位置1，服务器就不再进行数字签名的验证而把递归查询得到的结果直接交给解析器，由解析器自己验证签名的合法性。

最后，支持验证的DNSSEC 解析器对它所收到的资源记录的签名（RRSIG），必须能够区分区分以下四种结果：

（1）安全的（secure）：解析器能够建立到达资源记录签名者的信任链，并且可以验证数字签名的结果是正确的。

（2）不安全的（insecure）：解析器收到了一个资源记录和它的签名，但是它没有到达签名者的信任链，因而无法验证。

（3）伪造的（Bogus）：解析器有一个到资源记录签名者的信任链，但是签名验证是错的。可能是因为受到攻击了，也可能是管理员配置错误。

（4）不确定（Indeterminate）：解析器无法获得足够的DNSSEC 资源记录，因而不能确定用户所请求的资源记录是否应该签名。

# 3  DNSSEC的配置

尽管DNSSEC的工作原理看起来让人气馁，实际的配置操作却并不复杂。本节以BIND 9为例介绍DNSSEC的基本配置。尽管BIND 9.3以上就开始支持DNSSEC，仍然建议使用当前最高版本的BIND（当前最新的版本是9.8）。

配置或布署DNSSEC有两种场景：

（1）配置安全的域名解析服务器（Resolver），该服务器可以保护使用它的用户，防止被DNS 欺骗攻击。这里只涉及数字签名的验证工作。

（2）配置安全的权威域名服务器（Name Server），对权威域的资源记录进行签名，保护服务器不被域名欺骗攻击。

## 3.1   配置安全解析服务器

### 3.1.1 激活DNSSEC

首先，在BIND的配置文件（一般是/etc/named.conf）中打开DNSSEC选项，比如：

> options {
>
> directory “/var/named”;
>
> dnssec-validation yes;
>
> ….
>
> };

### 3.1.2 配置Trust anchor

其次，要给解析服务器配置可信锚（Trust Anchors），也就是你所信任的权威域的DNSKEY。理想情况下我们可以配置一个根的密钥就够了，但是目前DNSSEC还没有完全布署的情况下，我们需要配置很多安全岛（Secure Island）的密钥。可以从很多公开的网站下载这些可信域的DNSKEY文件，包括：

（1）Root Zone DNSSEC Trust Anchors：https://www.iana.org/dnssec/。2010年7月布署实施。如果DNSSEC全部布署成功，这一个公开密钥就足够了。

（2）The UCLA secspider ： https://secspider.cs.ucla.edu，由美国加州大学洛杉矶分校（UCLA）张丽霞教授的实验室维护。

（3）The IKS Jena TAR：https://www.iks-jena.de/leistungen/dnssec.php

这些文件大概是这样的格式：

> trusted-keys {
>
> “test.net.”  256 3 5  “AQPzzTWMz8qS…3mbz7Fh
>
> ……
>
> ….fHm9bHzMG1UBYtEIQ==”;
>
> “193.in-addr.arpa.” 257 3 5 “AwEAAc2Rn…HlCKscYl
>
> kf2kOcq9xCmZv….XXPN8E=”;
>
> };

假设上述trust anchors的文件为/var/named/trust-anchors.conf，则在/etc/named.conf中增加下面一行：

> include “/var/named/sec-trust-anchors.conf”;

### 3.1.3 测试

在完成上述配置修改之后重新启动named进程，然后选择一个trust anchor文件中有的区或者它下一级的域名，在解析服务器上用dig测试一下，例如：

> \#dig @127.0.0.1 +dnssec   test.net.  SOA

如果配置正确，应该返回test.net的SOA记录和相应的RRSIG记录，另外返回的头部中应该包含AD标志位，表示DNSSEC相关的数字签名验证是正确的，类似下面的样子：

> ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 1397
>
> ;; flags: qr rd ra **ad**; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 3

如果没有得到期望的结果，需要查看DNS的日志来找出问题。BIND为DNSSEC相关的事件增加了一个dnssec类别，可以在/etc/named.conf配置日志如下：

> logging {
>
> channel dnssec_log {
>
> file “log/dnssec” size 20m;   // a DNSSEC log channel 20m;
>
> print-time yes;                   // timestamp the entries
>
> print-category yes;          // add category name to entries
>
> print-severity yes;          // add severity level to entries
>
> severity debug 3;         // print debug message <= 3 t
>
> };
>
> };

category dnssec { dnssec_log; };

## 3.2   配置权威服务器

### 3.2.1 生成签名密钥对

首先为你的区（zone）文件生成密钥签名密钥KSK：

> \# cd /var/named
>
> \# dnssec-keygen -f KSK -a RSASHA1 -b 512 -n ZONE test.net.

> Ktest.edu.+005+15480

然后生成区签名密钥ZSK：

> \# dnssec-keygen -a RSASHA1 -b 512 -n ZONE test.net.
>
> Ktest.edu.+005+03674

其中的-a 参数是签名算法，-b是密钥长度。上述命令共产生两对DNSKEY密钥（共四个文件），分别以.key和.private结尾，表明这个文件存储的是公开密钥或私有密钥。

### 3.2.2 签名

签名之前，你需要把上面的两个DNSKEY写入到区文件中

> \#cat “$INCLUDE Ktest.net.+005+15480.key” >> db.test.net
>
> \# cat “$INCLUDE Ktest.net.+005+03674.key” >> db.test.net

然后执行签名操作：

> \# dnssec-signzone -o test.net. db.test.net
>
> db.test.net.signed

上面的-o选项指定代签名区的名字。生成的db.test.net.signed

然后修改/etc/named.conf如下：

> options  {
>
> directory “/var/named”;
>
> dnssec-enable yes;
>
> };
>
> zone “test.net” {
>
> type master;
>
> file “db.test.net.signed”;
>
> };

记住，你每次修改区中的数据时，都要重新签名：

> \# dnssec-signzone -o test.net -f db.test.net.signed.new db.test.net.signed
>
> \# mv db.test.net.signed db.test.net.signed.bak
>
> \# mv db.test.net.signed.new db.test.net.signed
>
> \# rndc reload test.net

### 3.2.3 发布公钥

要让其他人验证你的数字签名，其他人必须有一个可靠的途径获得你的公开密钥。DNSSEC通过上一级域名服务器数字签名的方式签发你的公钥。

用dnssec-signzone时，会自动生成keyset-文件和dsset-开头的两个文件，分别存储着KSK的DNSKEY记录和DS记录。作为test.net区的管理员，你需要把这两个文件发送给.net的管理员，.net的管理员需要把这两条记录增加到.net区中，并且用.net的密钥重新签名。

> test.net.              86400   IN NS   ns.test.net.
>
> 86400   DS      15480 5 1 (
>
> F340F3A05DB4D081B6D3D749F300636DCE3D
>
> 6C17 )
>
> 86400   RRSIG   DS 5 2 86400 20060219234934 (
>
> 20060120234934 23912   net.
>
> Nw4xLOhtFoP0cE6ECIC8GgpJKtGWstzk0uH6
>
> ……
>
> YWInWvWx12IiPKfkVU3F0EbosBA= )

如果你的上一级域名服务器还没有配置DNSSEC，你不得不另找其他方式了，比如，把上述两个文件提交到一些公开的trust anchor数据库中发布(如上面介绍过的secspider)，或者直接交给愿意相信你的解析服务器的管理员，配置到他们的trust anchor文件中。

# 4 DNSSEC的布署

互联网工程技术领域普遍认为DNSSEC是增强互联网基础设施安全非常关键的一步，比如，美国国土安全部发起了DNSSEC布署计划，美国网络空间国家战略安全明确要求布署DNSSEC。DNSSEC的大规模布署还可能解决其他的安全问题，比如密钥的分发和安全的电子邮件。

尽管互联网工程界认为DNSSEC非常重要，但是大规模的布署仍然非常谨慎。巴西 (.br)、保加利亚 (.bg)、捷克 (.cz)、波多黎各 (.pr) 和瑞典 (.se), 很早就开始了在他们国家的顶级域名上布署DNSSEC，IANA在2007年开始在一个根域名服务器上进行签名试验。许多机构在DNSSEC的布署上付出了巨大的努力，具体请参见[9]。

在ICANN和VeriSign的推动下，2010年7月所有13个根域名服务器都用对根域签名完毕且投入运营[10]。顶级域名（如.org, .net等）也在计划之中，有些国家级的顶级域名服务器（如.cz、.jp、.us、.fr等）已经完成了这一任务。

国际上一些大的ISP已经开始布署支持DNSSEC的解析服务器，以保护他们接入用户的安全，美国Comcast 公司是其中的先驱者之一。

# 5  总结与展望

本文概要介绍了DNSSEC的目标、原理、基本配置与当前的布署情况。由于篇幅和作者的经验、精力有限，无法对一些细节展开讨论， 感兴趣的读者可以阅读后面的参考文献。

作者认为如果DNSSEC能够普遍布署，可能给互联网的安全体系产生重大的影响；因为有了DNSSEC的数字签名，把DNS作为密钥分发的PKI在很多应用场合已经具有了可行性。它不仅仅可以加强DNS系统本身的安全，作为网络的基础设施，还给其他的应用、特别是端到端的移动通信、IPv6网络环境的安全问题提供新的解决办法。

但是作者也认为，离这一目标的实现还有很长的路要走。一方面DNSSEC自身的协议还在发展，另一方面其他的应用要想利用DNSSEC的机制还需要大规模的研究试验和最后的标准化。另外，DNSSEC在某些国家是否会遭遇政策性的阻碍，也是个未知数。



尽管如此，因为DNS对互联网实在太重要了；如果DNSSEC是通往一个安全可靠的互联网基础设施的必由之路（目前作者还没有看到可以替代DNSSEC的其他解决方案），为此所付出的代价，也是别无选择的。

# 6   参考文献

[1] Graham Lowe, Patrick Winters, Michael L. Marcus. The Great DNS Wall of China， December 21, 2007.  http://cs.nyu.edu/~pcw216/work/nds/final.pdf

[2] RFC 2065,Domain Name System Security Extensions, 1997.http://www.ietf.org/rfc/rfc2065.txt

[3] RFC 2535,Domain Name System Security Extensions,1999,http://www.ietf.org/rfc/rfc2535.txt

[4] RFC 4033, DNS Security Introduction and Requirements,2005, http://www.ietf.org/rfc/rfc4033.txt

[5] RFC4034,Resource Records for the DNS Security Extensions,2005,http://www.ietf.org/rfc/rfc4034.txt

[6] RFC4035,Protocol Modifications for the DNS Security Extensions,2005,http://www.ietf.org/rfc/rfc4035.txt

[7] RFC5155,DNS Security (DNSSEC) Hashed Authenticated Denial of Existence,2008,http://www.ietf.org/rfc/rfc5155.txt

[8] Paul Albitz, Cricket Liu, DNS and BIND 5th Edition, O’Reilly Publication, 2008

[9] Wikipedia, Domain Name System Security Extensions, http://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions

[10] Root DNSSEC update, 2010, http://www.root-dnssec.org/2010/07/16/status-update-2010-07-16/