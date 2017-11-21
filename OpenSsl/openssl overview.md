## OpenSSL的要领：用SSL证书，私钥和CSR的工作
 
### 介绍
OpenSSL是一个多功能的命令行工具，可用于与公钥基础设施（PKI）和HTTPS（HTTP over TLS）相关的大量任务。 这个cheat sheet样式指南提供了对常用的日常场景中有用的OpenSSL命令的快速参考。 这包括生成私钥，证书签名请求和证书格式转换的OpenSSL示例。 它不包括OpenSSL的所有使用。

#### 关于证书签名请求（CSR）

如果要从证书颁发机构（CA）获取SSL证书，则必须生成证书签名请求（CSR）。 CSR主要由密钥对的公钥和一些附加信息组成。 这两个组件在签名时插入到证书中。

无论何时生成CSR，系统都会提示您提供有关证书的信息。 此信息称为识别名称（DN）。 在DN的一个重要领域是通用名称 （CN），这应该是您要使用的证书的主机的确切完全限定域名（FQDN）。 通过经由命令行或从文件传递信息来创建CSR时，也可以跳过交互式提示。

DN中的其他项目提供有关您的业务或组织的其他信息。 如果您从证书颁发机构购买SSL证书，则通常需要这些附加字段（例如“组织”）准确反映您组织的详细信息。

以下是CSR信息提示的外观示例：

```
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:New York
Locality Name (eg, city) []:Brooklyn
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Example Brooklyn Company
Organizational Unit Name (eg, section) []:Technology Division
Common Name (e.g. server FQDN or YOUR name) []:examplebrooklyn.com
Email Address []:
```

如果你想以非交互回答CSR信息提示，你可以这样做通过添加-subj选项任何OpenSSL的命令，要求企业社会责任的信息。 下面是一个使用上面代码块中显示的相同信息的选项示例：
```
-subj "/C=US/ST=New York/L=Brooklyn/O=Example Brooklyn Company/CN=examplebrooklyn.com"
```
现在您已经了解了CSR，请随时跳到本指南中涵盖OpenSSL需求的部分。

##### 生成CSR
本节涵盖与生成CSR相关的OpenSSL命令（以及私钥，如果它们尚不存在）。 CSR可用于从证书颁发机构请求SSL证书。

请记住，您可以用非交互添加CSR信息-subj选项，在上一节中提到。

##### 生成私钥和CSR
如果要使用HTTPS（HTTP over TLS）来保护Apache HTTP或Nginx Web服务器，并且希望使用证书颁发机构（CA）颁发SSL证书，请使用此方法。 生成的CSR可以发送到CA，以请求签发CA签署的SSL证书。 如果您的CA支持SHA-2，添加-sha256选项和SHA-2签署CSR。

此命令创建一个2048位密钥（ domain.key ）和CSR（ domain.csr从头开始）：
```
openssl req \
       -newkey rsa:2048 -nodes -keyout domain.key \
       -out domain.csr
```      
回答CSR信息提示以完成该过程。

该-newkey rsa:2048选项指定的关键应该是2048位，使用RSA算法生成的。 该-nodes选项指定私钥不应该密码短语进行加密。 该-new选项，这里没有包括在内，但暗示，表示正在生成CSR。

从现有的私钥生成CSR
如果您已拥有要用于从CA请求证书的私钥，请使用此方法。

此命令创建一个新的CSR（ domain.csr基于现有的私有密钥() domain.key ）：

openssl req \
       -key domain.key \
       -new -out domain.csr
回答CSR信息提示以完成该过程。

所述-key选项指定现有的私钥（ domain.key将用于生成一个新的CSR）。 该-new选项表示正在生成一个CSR。

从现有证书和私钥生成CSR
如果您要续订现有证书，但是您或您的CA因某种原因没有原始的CSR，请使用此方法。 它基本上可以节省您重新输入CSR信息的麻烦，因为它从现有证书中提取信息。

此命令创建一个新的CSR（ domain.csr基于现有证书() domain.crt ）和私钥（ domain.key ）：

openssl x509 \
       -in domain.crt \
       -signkey domain.key \
       -x509toreq -out domain.csr
该-x509toreq选项指定使用的X509证书，使企业社会责任。

生成SSL证书

 
如果要使用SSL证书来保护服务，但不需要CA签发的证书，则有效（免费）解决方案是签署您自己的证书。

证书可以发出自己的一种常见类型是自签名证书 。 自签名证书是使用其自己的私钥签名的证书。 自签名证书可用于加密数据以及CA签名的证书，但您的用户将显示一条警告，表示证书不受其计算机或浏览器信任。 因此，只有在您不需要向其用户（例如非生产或非公用服务器）证明您的服务的身份时，才应使用自签名证书。

本节包括与生成自签名证书相关的OpenSSL命令。

生成自签名证书
如果要使用HTTPS（HTTP over TLS）来保护Apache HTTP或Nginx Web服务器，并且不要求您的证书由CA签名，请使用此方法。

此命令创建一个2048位密钥（ domain.key ）和一个自签名证书（ domain.crt从头开始）：

openssl req \
       -newkey rsa:2048 -nodes -keyout domain.key \
       -x509 -days 365 -out domain.crt
回答CSR信息提示以完成该过程。

该-x509选项告诉req创建一个自签署cerificate。 该-days 365选项指定证书的有效期为365天。 生成临时CSR以收集要与证书关联的信息。

从现有的私钥生成自签名证书
如果您已经有要使用它生成自签名证书的私钥，请使用此方法。

此命令创建一个自签名证书（ domain.crt从现有的私有密钥() domain.key ）：

openssl req \
       -key domain.key \
       -new \
       -x509 -days 365 -out domain.crt
回答CSR信息提示以完成该过程。

该-x509选项告诉req创建一个自签署cerificate。 该-days 365选项指定证书的有效期为365天。 该-new选项使CSR信息提示。

从现有的私钥和CSR生成自签名证书
如果您已有私钥和CSR，并且希望使用它们生成自签名证书，请使用此方法。

此命令创建一个自签名证书（ domain.crt从现有的私有密钥() domain.key ）和（ domain.csr ）：

openssl x509 \
       -signkey domain.key \
       -in domain.csr \
       -req -days 365 -out domain.crt
该-days 365选项指定证书的有效期为365天。

查看证书
证书和CSR文件以PEM格式编码，这不易于人类阅读。

本节介绍将输出PEM编码文件的实际条目的OpenSSL命令。

查看CSR条目
该命令允许您查看和验证CSR（内容domain.csr纯文本）：

openssl req -text -noout -verify -in domain.csr
查看证书条目
此命令可以查看证书（内容domain.crt纯文本）：

openssl x509 -text -noout -in domain.crt
验证CA签署的证书
使用此命令来验证证书（ domain.crt ）是由一个特定的CA证书（签署ca.crt ）：

openssl verify -verbose -CAFile ca.crt domain.crt
私钥

 
本节包括专用于创建和验证私钥的OpenSSL命令。

创建私钥
使用此命令创建一个密码保护，2048位私钥（ domain.key ）：

openssl genrsa -des3 -out domain.key 2048
系统提示完成此过程时输入密码。

验证私钥
使用此命令检查私钥（ domain.key ）是一个有效的关键：

openssl rsa -check -in domain.key
如果您的私钥是加密的，将提示您输入其密码。 成功后，未加密的密钥将在终端上输出。

验证私钥匹配证书和CSR
使用这些命令来验证，如果私钥（ domain.key ）证书（符合domain.crt ）和CSR（ domain.csr ）：

openssl rsa -noout -modulus -in domain.key | openssl md5
openssl x509 -noout -modulus -in domain.crt | openssl md5
openssl req -noout -modulus -in domain.csr | openssl md5
如果每个命令的输出相同，则私钥，证书和CSR相关的可能性极高。

加密私钥
这需要一个未加密的私钥（ unencrypted.key ），并输出它（的加密版本encrypted.key ）：

openssl rsa -des3 \
       -in unencrypted.key \
       -out encrypted.key
输入您想要的密码短语，用加密私钥。

解密私钥
这需要一个加密私钥（ encrypted.key ），并输出它（解密版本decrypted.key ）：

openssl rsa \
       -in encrypted.key \
       -out decrypted.key
出现提示时，输入加密密钥的密码短语。

转换证书格式
我们一直使用的所有证书都是ASCII PEM编码的X.509证书。 还有各种其他的证书编码和容器类型; 一些应用程序比其他应用程序更喜欢某些格式。 此外，许多这些格式可以在单个文件中包含多个项目，例如私钥，证书和CA证书。

OpenSSL可用于将证书转换为各种各样的格式。 本节将介绍一些可能的转化。

将PEM转换为DER
如果你想一个PEM编码的证书（转换使用此命令domain.crt ）到DER编码的证书（ domain.der ），二进制格式：

openssl x509 \
       -in domain.crt \
       -outform der -out domain.der
DER格式通常与Java一起使用。

将DER转换为PEM
如果你想要一个DER编码的证书（转换使用此命令domain.der ）到PEM编码的证书（ domain.crt ）：

openssl x509 \
       -inform der -in domain.der \
       -out domain.crt
将PEM转换为PKCS7
如果你想PEM证书（添加使用此命令domain.crt和ca-chain.crt ）到PKCS7文件（ domain.p7b ）：

openssl crl2pkcs7 -nocrl \
       -certfile domain.crt \
       -certfile ca-chain.crt \
       -out domain.p7b
请注意，您可以使用一个或多个-certfile选项来指定要添加到PKCS7文件，证书。

PKCS7文件（也称为P7B）通常用于Java密钥库和Microsoft IIS（Windows）中。 它们是ASCII文件，可以包含证书和CA证书。

将PKCS7转换为PEM
如果你想一个PKCS7文件（转换使用此命令domain.p7b ）到PEM文件：

openssl pkcs7 \
       -in domain.p7b \
       -print_certs -out domain.crt
请注意，如果您的PKCS7文件中有多个项目（例如证书和CA中间证书），则创建的PEM文件将包含其中的所有项目。

将PEM转换为PKCS12
如果你想利用私钥（使用此命令domain.key ）和证书（ domain.crt ），并结合成一个PKCS12文件（ domain.pfx ）：

openssl pkcs12 \
       -inkey domain.key \
       -in domain.crt \
       -export -out domain.pfx
系统将提示您输出导出密码，您可以留空。 请注意，您可以通过一个单一的PEM文件（串联证书加在一起的证书链到PKCS12文件domain.crt在这种情况下）。

PKCS12文件（也称为PFX文件）通常用于在Micrsoft IIS（Windows）中导入和导出证书链。

将PKCS12转换为PEM
如果你想转换一个PKCS12文件（使用此命令domain.pfx ），并将其转换为PEM格式（ domain.combined.crt ）：

openssl pkcs12 \
       -in domain.pfx \
       -nodes -out domain.combined.crt
请注意，如果您的PKCS12文件中有多个项目（例如证书和私钥），则创建的PEM文件将包含其中的所有项目。

OpenSSL版本

 
该openssl version命令可以用来检查正在运行的版本。 您正在运行的OpenSSL版本及其编译的选项会影响您可用的功能（有时还包括命令行选项）。

以下命令显示您正在运行的OpenSSL版本及其编译时使用的所有选项：

openssl version -a
本指南是使用具有以下详细信息（上一个命令的输出）的OpenSSL二进制文件编写的：

OpenSSL 1.0.1f 6 Jan 2014
built on: Mon Apr  7 21:22:23 UTC 2014
platform: debian-amd64
options:  bn(64,64) rc4(16x,int) des(idx,cisc,16,int) blowfish(idx)
compiler: cc -fPIC -DOPENSSL_PIC -DOPENSSL_THREADS -D_REENTRANT -DDSO_DLFCN -DHAVE_DLFCN_H -m64 -DL_ENDIAN -DTERMIO -g -O2 -fstack-protector --param=ssp-buffer-size=4 -Wformat -Werror=format-security -D_FORTIFY_SOURCE=2 -Wl,-Bsymbolic-functions -Wl,-z,relro -Wa,--noexecstack -Wall -DMD32_REG_T=int -DOPENSSL_IA32_SSE2 -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_MONT5 -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DMD5_ASM -DAES_ASM -DVPAES_ASM -DBSAES_ASM -DWHIRLPOOL_ASM -DGHASH_ASM
OPENSSLDIR: "/usr/lib/ssl"
结论
这应该涵盖大多数人使用OpenSSL来处理SSL证书！ 它有许多其他用途没有在这里涵盖，所以随时提出或建议其他用途在评论。

如果您有任何命令的问题，请务必发表评论（并包括您的OpenSSL版本输出）。
