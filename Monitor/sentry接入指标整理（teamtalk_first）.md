### 待接入指标整理（TeamTalk First）

#### 展示平台

[Sentry2​]: http://sentry.mogujie.org/sentry2	"Sentry2监控平台"

##### 总连接数

- 枚举
  - old： kMonitorAppType_TotalConns （0x0001）
  - new：kMonitorBizType_TotalConns  （0x0001）

- 计算规则

  监控服务端主动向被监控服务器拉取


- Metric

  AppName_All_Conns

- URL
  /monitor/total.conns	
  ​

##### 活跃连接数

- 计算规则

  活跃连接数的定义是什么？

- 数据流入方式

  ​

- Metric

  AppName_Active_Conns
  
- URL
  /monitor/active.conns
  

##### 总计发送字节数

- 计算规则

- 数据流入方式

- Metric

  AppName_All_SendBytes
  
- URL
  /monitor/all.send.bytes
  

##### 总计接收字节数

- 计算规则

- 数据流入方式

- Metric

  AppName_All_RecvBytes
  
- URL

  /monitor/all.recv.bytes


##### 总计发送消息数

- 计算规则

- 数据流入方式

- Metric

  AppName_All_SendMsg
  
- URL

  /monitor/all.send.msg.count

##### 总计接收消息数

- 计算规则

- 数据流入方式

- Metric

  AppName_All_RecvMsg
    
- URL

  /monitor/all.recv.msg.count


##### 每个消息发送次数

- 计算规则

- 数据流入方式

- Metric

  sentry2

  metric: AppName_Msg_SendCount

##### 每个消息接收次数

- 计算规则

- 数据流入方式

- Metric

  AppName_Msg_RecvCount

##### 总当前等待处理消息数

- 计算规则

- 数据流入方式

- Metric

  AppName_Msg_WaitHandle
  
- URL

  /monitor/msg.wait.handle.count
  
##### 连接异常数

- 计算规则

- 数据流入方式

- Metric

  AppName_Conn_ExceptCount
  
- URL

  /monitor/conn.except.count

##### 协议解析异常数

- 计算规则

- 数据流入方式

- Metric

  AppName_PDU_ParseExceptCount
  
- URL

  /monitor/pdu.parse.except.count  

##### 协议数据异常数

- 计算规则

- 数据流入方式

- Metric

  AppName_PDU_DataExceptCount
  
- URL

  /monitor/pdu.data.except.count  

##### 每个HTTP消息发送次数

- 计算规则

- 数据流入方式

- Metric

  AppName_Msg_HTTPSendCount

##### 每个HTTP消息接收次数

- 计算规则

- 数据流入方式

- Metric

  AppName_Msg_HTTPRecvCount

##### 总计成功登陆用户数

- 计算规则

- 数据流入方式

- Metric

  AppName_LoginCount_Succeed
  
- URL

  /monitor/login.succeed.count  

##### 在线登陆用户数

- 计算规则

- 数据流入方式

- Metric

  AppName_LoginCount_OnlineSucceed
  
- URL

  /monitor/online.succeed.count   

##### 登陆失败次数

- 计算规则

- 数据流入方式

- Metric

  AppName_LoginCount_Failed
  
- URL

  /monitor/login.failed.count     

##### 登陆超时次数

- 计算规则

- 数据流入方式

- Metric

  AppName_LoginCount_Timeout
  
- URL

  /monitor/login.timeout.count  

##### 主动关闭次数

- 计算规则

- 数据流入方式

- Metric

  AppName_Conn_ClosedCount
  
- URL

  /monitor/conn.closed.count   

##### 超时关闭次数

- 计算规则

- 数据流入方式

- Metric

  AppName_Conn_TimeoutCount
  
- URL

  /monitor/conn.timeout.closed.count     

##### 实际在线人数

- 计算规则

- 数据流入方式

- Metric

  AppName_All_OnlineUserCount
  
- URL

  /monitor/online.user.count     

##### 发送消息耗时

- 计算规则

- 数据流入方式

- Metric

  AppName_Msg_SendTimeCost
  
- URL

  /monitor/msg.send.time.cost     

##### 协程超时次数

- 计算规则

- 数据流入方式

- Metric

  AppName_Coroutine_TimeoutCount
  
- URL

  /monitor/coroutine.timeout.count    

##### 每个链接发送字节数

- 计算规则

- 数据流入方式

- Metric

  AppName_Conn_SendBytes

##### 每个链接接收字节数

- 计算规则

- 数据流入方式

- Metric

  AppName_Conn_RecvBytes

##### 每个链接发送消息数

- 计算规则

- 数据流入方式

- Metric

  AppName_Conn_SendMsgCount

##### 每个链接接收消息数

- 计算规则

- 数据流入方式

- Metric

  AppName_Conn_RecvMsgCount

##### 每个链接发送HTTP消息数

- 计算规则

- 数据流入方式

- Metric

  AppName_Conn_HTTPSendMsgCount

##### 每个链接接收HTTP消息数

- 计算规则

- 数据流入方式

- Metric

  AppName_Conn_HTTPRecvMsgCount

##### 每个链接建立至今总时长

- 计算规则

- 数据流入方式

- Metric

  AppName_Conn_Duration

##### 最大链接数

- 计算规则

- 数据流入方式

- Metric

  AppName_Conn_MaxCount

##### 普通用户在线数量

- 计算规则

- 数据流入方式

- Metric

  AppName_OnlineUserCount_Normal
  
- URL

  /monitor/online.normal.user.count    

##### 店主在线数量

- 计算规则

- 数据流入方式

- Metric

  AppName_OnlineUserCount_ShopOwner
  
- URL

  /monitor/shop.owner.online.count  

##### 客服在线数量

- 计算规则

- 数据流入方式

- Metric

  AppName_OnlineUserCount_Service
  
- URL

  /monitor/service.online.count  

##### 小侠在线数量

- 计算规则

- 数据流入方式

- Metric

  AppName_OnlineUserCount_MaleStaff
  
- URL

  /monitor/male.staff.online.count  

##### 小仙在线数量

- 计算规则

- 数据流入方式

- Metric

  AppName_OnlineUserCount_FemaleStaff
  
- URL

  /monitor/female.staff.online.count  

##### 活跃客户端在线数量（win，mac, flash, android, iOS, android, WinPhone, http, CInfo）

- 计算规则

- 数据流入方式

- Metric

  AppName_PlatformName_OnlineUserCount_Active
  
- URL

  /monitor/active.online.user.count  

##### 登陆失败数量（win，mac, flash, android, iOS, android, WinPhone, http, CInfo）与前面的一个重复

- 计算规则

- 数据流入方式

- Metric

  AppName_PlatformName_LoginCount_Failed
  
- URL

  /monitor/login.failed.count  

##### 超时关闭数量（win，mac, flash, android, iOS, android, WinPhone, http, CInfo）与前面的一个重复

- 计算规则

- 数据流入方式

- Metric

  AppName_PlatformName_Conn_TimeoutCount
  
- URL

  /monitor/conn.timeout.close.count  

##### 登陆超时数量（win，mac, flash, android, iOS, android, WinPhone, http, CInfo）与前面的一个重复

- 计算规则

- 数据流入方式

- Metric

  AppName_PlatformName_LoginCount_Timeout
  
- URL

  /monitor/login.timeout.count  

##### 业务执行耗时

- 计算规则

- 数据流入方式

- Metric

  AppName_PlatformName_Biz_Cost