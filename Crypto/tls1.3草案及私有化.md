#tls13 草案地址

https://tlswg.github.io/tls13-spec/


# 协议设计

## 协议组成
```
| header | body |
```
### header  结构
```
｜type (1) | version (1) | attribute count (1) | reserve (1) | length (4) |
```
注解：（X）代表几个字节

### body 结构，使用对称加密算法加密
```
| attributes | body | 
```
### header 字段解释
```
 enum {
       client_hello(0x01),
       server_hello(0x02),
       client_reuse(0x03),
       alert(0x04),
       application_data(0x05), 
       (0xff)
} type;
```
```
version: MLS1.0, 二进制：00000000，以后依次递增
attribute count： 属性的个数，0-255
reserver：保留位
length： data的长度
```

### body字段解释

attribute: 属性，结构如下：

```
|key (1) | length (2) |value |
```

body: 客户端的数据

例子：

```
| 0x01 | 2 | aa|
```
### attribute 的说明

```
enum {
    crypto method(1), //uint8, 见下面解释

    server pub key(2), //string, 在server hello里面使用，更新客户端的server pub key,
    // 需要持久保存, 服务端的私钥泄漏时，使用这个参数
    server pub key version(3), //uint8 pub key version,用来更新key

    client pub key(4), //string, 传给服务端计算aeskey
    client random(5), //string, 传给服务端，加上server random计算签名
    server random(6), string
    ticket(6), //string, 用来保存aeskey，实现session reuse
    session id(7), //string, 用来标示reuse和新建连接的唯一客户端
    code(8), //uint16_t, server hello的返回码, 200(ok),400(failed), 300(update server key)
    errreason(9), //uint8_t code=400, errreason 描述了具体信息
    signature(10), //string, server hello的签名
    level(11), //uint8, alert使用,见下面解释
    description(12), //uint8, alert使用,见下面解释
    aeskey timeout(13), //uint32, aeskey在客户端的存储时间
}
```

#### client hello 正常握手
attribute包括：

```
enum {
    crypto_method,
    client pub key,
    client pub key index,
    random,
    (63)
}
```
属性名称：

```
crypto_method 
key = 0x01
value = uint8

value enum {
    TLS_ECDHE_PSK_WITH_NONE(0),  //不加密
    TLS_ECDHE_PSK_WITH_AES_256_GCM_SHA384(1), // 客户端不支持
    TLS_ECDHE_PSK_WITH_CHACHA20_POLY1305_SHA256(2),
    (63)
}
```

### client hello: reuse

attribute包括：

```
enum {
    session id,
    ticket,
    client random,
    server pub key version,
    (63)
}
```
#### server hello

attribute包括：

```
enum {
    session id,
    code,
    server random,
    signature,
    ticket,
    server pub key,
    server pub key index,
    aeskey timeout,
    (63)
}
```

属性名称：

```
code
key = 0x02
value = uint16
```
可选，code代表服务端连接的状态，客户端需要处理返回值
200: OK
401: forbidden


#### alert：
attribute包括：

```
enum {
    alert level(1),
    alert description(2),
    (63)
}
```

属性名称：

```
alert level
key = 0x01
value = uint8 
value enum {
    warning(1),
    fatal(2),
    (255)
}
```
属性名称

```
alert description
key = 0x02
value = uint8

 value  enum {
       close_notify(0),
       end_of_early_data(1),
       unexpected_message(10),               /* fatal */
       bad_record_mac(20),                   /* fatal */
       record_overflow(22),                  /* fatal */
       handshake_failure(40),                /* fatal */
       bad_certificate(42),
       unsupported_certificate(43),
       certificate_revoked(44),
       certificate_expired(45),
       certificate_unknown(46),
       illegal_parameter(47),                /* fatal */
       unknown_ca(48),                       /* fatal */
       access_denied(49),                    /* fatal */
       decode_error(50),                     /* fatal */
       decrypt_error(51),                    /* fatal */
       protocol_version(70),                 /* fatal */
       insufficient_security(71),            /* fatal */
       internal_error(80),                   /* fatal */
       inappropriate_fallback(86),           /* fatal */
       user_canceled(90),
       missing_extension(109),               /* fatal */
       unsupported_extension(110),           /* fatal */
       certificate_unobtainable(111),
       unrecognized_name(112),
       bad_certificate_status_response(113), /* fatal */
       bad_certificate_hash_value(114),      /* fatal */
       unknown_psk_identity(115),
       (255)
   } AlertDescription;
```
  


