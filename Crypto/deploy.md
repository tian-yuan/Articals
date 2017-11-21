### 新框架限流及线上部署预估

#### device-info-center

#### message-scheduler

当前单个 message-scheduler 实例限流 24w每分钟；当前线上有道云笔记设备数在 9千万左右，如果一个广播推送所有设备，就有9千万消息，如果要保证在半小时内推送完，
scheduler需要部署13个，为了保证服务质量，线上可以部署20个（老框架broadcast-receiver+filter，30+个实例）。

#### apns-proxy

线上当前一个apns-server实例，10s可以推送10k+消息，一分钟大概6w+，apns-proxy消息推送到苹果服务器的性能主要受到网络环境的影响，和apns-server的性能基本一致，当前apns推送量峰值将近3千万，保证在30分钟内推送完，需要部署17个，线上可以部署30个。

#### message-sender

#### connector

在不修改node参数的情况下，可每个nodejs节点可以常规承受的连接数为 30000。如果持续对内存使用进行优化的情况下，可以达到与mqtt的连接承受能力同样的水平。

线上峰值连接数为650w，需要部署220个，老框架部署数量将近600。

#### 列表
 
|服务|限流|最低部署数量|部署数量|
|---|---|---|
|message-scheduler|24w/m|13|20|
|apns-proxy|6w/m|17|30|
|message-sender||||
|connetor||220|300?|
