## Eureka

### the difference between Eureka1.x and Eureka2.x

#### the limitation of Eureka1.x

* Only support homogenous client views: Eureka servers only expects the client to always get the entire 
registry and does not allow to get specific applications/VIP addresses. This imposes a memory 
limitation on all clients registering with Eureka, even if they need a very small subset of the 
Eureka’s registry.
仅仅支持全量更新；

* Only supports scheduled updates: Eureka client follows a poll model to fetch updates from the 
server routinely. This imposes an overhead on the client of doing a scheduled call to the server, 
even if there are no changes and also delays the updates by the poll interval.
仅仅支持定时更新，客户端轮询模式获取registry，不支持发布通知机制；

* Replication algorithm limits scalability: Eureka follows a broadcast replication model i.e. 
all the servers replicate data and heartbeats to all the peers. This is simple and effective 
for the data set that eureka contains however replication is implemented by relaying all the 
HTTP calls that a server receives as is to all the peers. This limits scalability as every node 
has to withstand the entire write load on eureka.
备份算法限制了可扩展性，所有的服务备份所有的数据到所有的服务端，任何一个节点需要承担整个写负载；

so, need to upgrade.