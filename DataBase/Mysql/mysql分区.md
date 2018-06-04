### MYSQL 分区操作

* 创建分区表

```
CREATE TABLE employees (
    id INT NOT NULL,
    job_code INT,
    store_id INT
)
 
PARTITION BY LIST(store_id)
 (  PARTITION pNorth VALUES IN (3,5,6,9,17),
    PARTITION pEast VALUES IN (1,2,10,11,19,20),
    PARTITION pWest VALUES IN (4,12,13,14,18),
    PARTITION pCentral VALUES IN (7,8,15,16)
);
```

* 添加数据

```
insert into employees (id, job_code, store_id) values (1, 1, 2);
insert into employees (id, job_code, store_id) values (1, 1, 3)
```

此时有一条数据在 pNorth 里面，有一条数据在 pEast 里面

* 修改数据

```
update employees set store_id = 10 where store_id=3
```

此时原本在 pNorth 分区中的数据现在在 pEast 分区里面了



## 刘泽波

### 上周工作

* 新闻环境问题修复
  * 修改 push-server refreshSession 导致数据不一致问题
  * 修改 push-server 内存泄漏问题
* IOT 分支相关
  * 发布 IOT push-server-sdk 和 push-common JAR 包
  * 联调 RPC 功能

### 本周计划

* 新闻环境push-device操作数据库优化及BUG修复
  * 修复 bind_expire 问题
  * 优化 mysql 操作，减少不必要的数据库操作
  * 设备表调整，优化数据库操作性能，包括增加自增ID，增加redis缓存，数据库读写分离开发等

* push-scheduler优化
  * 使用 JAVA 重写 push-scheduler，本周完成模块框架的开发