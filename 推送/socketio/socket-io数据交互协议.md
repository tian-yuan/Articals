# Socket.io 1.06数据交互协议

`协议文档`

------

# 文档约定

## 关键字

本协议文档涉及名词 
protocol : http协议:{`http`, `https`}, websocket协议: {`ws`, `wss`} 
host: `域名` 
timestamp : `时间戳`, 为1970-01-01 00:00:00.000 至今的毫秒数 
sid : `sid` 握手成功时服务器返回

## 约定

[]中描述内容, 表示可选项

## 协议指令 `command`

| 名称           | 指令   | 备注   |
| ------------ | ---- | ---- |
| disconnected | 0    | 关闭链路 |
| connected    | 1    | 创建链路 |
| heartbeat    | 2    | 心跳   |
| pong         | 3    | 心跳回应 |
| message      | 4    | 发送消息 |
| upgrade      | 5    | 未知   |
| noop         | 6    | 未知   |

## 消息类型 `messageType`

| 名称         | 值    | 备注   |
| ---------- | ---- | ---- |
| connect    | 0    |      |
| disconnect | 1    |      |
| event      | 2    | 未知   |
| ack        | 3    |      |
| error      | 4    |      |
| binarevent | 5    |      |
| binaryack  | 6    |      |

# handshake 握手

## request

握手URL: 
`protocol`://`host`:`port`/socket.io/1/?EIO=2&transport=polling&t=`timestamp`[&自定义参数列表]

post参数列表: 
无

header参数列表:

```
Cookie:{  name:"auth" ,  value:"56cdea636acdf132" ,  domain:"cim.rd.mailtech.cn" ,  path:"/" }
```

## response

返回的数据为二进制 
\1. data header 
byte1 byte2 byte3 byte4 未知 --服务器测试结果返回 00 09 07 ff, 暂时以首字节00 作为1.0+版本的判断依据, 尾字节ff作为实际参数的分隔符

1. data body

```
{  "sid":"hxXdj-6ZgyTU0wxlAABT",  --sid  "upgrades":["websocket"],      --连接类型  "pingInterval":25000,          --心跳周期  "pingTimeout":60000            --心跳超时}
```

# WebSocket 通信

URL: 
`protocol`://`host`:`port`/socket.io/1/websocket/?EIO=2&transport=websocket&sid=`sid` 
需传入握手请求中获取到的`sid`, 以该url进行websocket通信

例如: 
ws://cim.rd.mailtech.cn:8082/socket.io/1/websocket/?EIO=2&transport=websocket&sid=M8UwZBaOGgT_d7r4AAC6

# disconnect断开握手

URL: 
`protocol`://`host`:`port`/socket.io/1/xhr-polling/`sid`?disconnect

# request

post参数: 
无

header参数: 
无

# response

statuscode == 200 则请求成功

# ping-pong 心跳与回应

"2probe" <--> "3probe"

# send 发送Json信息

发送消息 
消息格式为: `command` `messageType` [`req_id`] `消息体`

举例: 
**42["publish",{"topic":"#person#","payload":"unread=0"}]** 
**421["publish",{"topic":"#person#","payload":"unread=0"}]** 
**422["publish",{"topic":"#person#","payload":"unread=0"}]** 
意义为 `message` `event`

返回消息 
"5[`req_id`]"