#lurker架构设计
￼
1、通过 logAgent 收集日志；
2、通过 kafaka 实时从 logAgent 拉去日志信息；
3、使用 storm 实时分析日志；
4、推送到 hbase 进行实时保存；
5、使用 MAP-REDUCE 集群进行聚合计算；