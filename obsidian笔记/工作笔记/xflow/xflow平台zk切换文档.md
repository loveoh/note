
### 背景：

由于cdh下线，zookeeper集群需要重新搭建，并进行迁移。

### zk集群迁移文档

#### 前期工作：

通过oa申请zk相关的机器3台（172.26.53.28，172.26.64.167,127.26.64.168）

老zk集群节点（172.26.25.109,172.26.18.214,172.26.25.125）

##### zk集群安装

1、下载安装包  
cd /opt/software  
# 下载zookeeper安装包  
wget http://archive.apache.org/dist/zookeeper/stable/apache-zookeeper-3.7.1-bin.tar.gz  
# 解压zookeeper安装包  
tar -zxvf apache-zookeeper-3.7.1-bin.tar.gz  
mv apache-zookeeper-3.7.1-bin /opt/zookeeper  
​  
2、修改配置  
修改zk配置文件，zoo.cfg  
新增如下配置项，zk集群节点
server.1=172.26.53.28:3181:4181  
server.2=172.26.64.167:3181:4181  
server.3=172.26.64.168:3181:4181  
​  
#监控指标暴露  
metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider  
metricsProvider.httpPort=7000  
metricsProvider.exportJvmInfo=true  
# 数据快照文件存放路径  
dataDir=/opt/zookeeper/data  
# 事务日志存放路径  
dataLogDir=/opt/zookeeper/log  
​  
# 生成myid文件，指定myid服务号(每个节点不一致，用来标识zk节点作用)  
echo "1" > /opt/zookeeper/data/myid  
​  
# 新增java.env文件，配置相关环境变量（可以指定zk启动JVM参数）  
touch /opt/zookeeper/apache-zookeeper-3.7.1-bin/conf/java.env  
​  
export JVMFLAGS="-Xms4096m -Xmx5120m  -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:/alidata1/log/zookeeper/out/gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=20M $JVMFLAGS"  
​  
#修改log4j.properties文件，配置zk日志输出地址  
/alidata1/log/zookeeper   
​  
#zk集群相关操作命令  
/opt/zookeeper/apache-zookeeper-3.7.1-bin/bin/zkServer.sh stop  
/opt/zookeeper/apache-zookeeper-3.7.1-bin/bin/zkServer.sh start  
/opt/zookeeper/apache-zookeeper-3.7.1-bin/bin/zkServer.sh status  
​  
​  
zk集群搭建可以参考：  
https://juejin.cn/post/7114254500692492301

##### zk集群切换操作步骤

1. 选择业务低峰期时间段，停止所有ck的写入任务，包括离线任务，及实时任务（实时任务id：2948,10295，2959,10379,10626）
    
2. 通过/opt/zookeeper/apache-zookeeper-3.7.1-bin/bin/zkServer.sh status命令，找到对应的leader节点
    
3. 新zk集群先停机，建议先停机follower节点，后停机leader节点，并且同步删除/opt/zookeeper/data/version-2，/opt/zookeeper/log/version-2文件夹下的所有文件 。
    
4. copy老zk集群leader节点下最新的日志文件及快照文件到对应的新zk集群中（新zk集群中每个节点都需要copy）。
    
    scp /opt/zookeeper/data/version-2/snapshot.3b03f14 172.26.53.28:/opt/zookeeper/data/version-2/
    
    scp /opt/zookeeper/log/version-2/log.3b03969f15 172.26.53.28:/opt/zookeeper/log/version-2/
    
    保证每一台新zk集群的事务日志和快照文件都是最新的
    
5. 通过/opt/zookeeper/apache-zookeeper-3.7.1-bin/bin/zkServer.sh start命令分别启动新zk集群，并观察对应zk的状态。
    
6. 如果zk集群节点为1主2从，并且没有明显异常日志报错，则证明zk集群启动成功。
    
7. 修改ck配置文件/etc/clickhouse-server/metrika.xml，将zookeeper-servers配置修改为新zk集群
    

##### 业务验证

1. 平台功能验证，数据可以查询，可以写入，可以做相关元数据操作，如删除分区，建表，删表，删列等。
    
2. 任务相关验证，实时flink任务可以正常写入，datax任务数据可正常插入
    

##### 异常回滚方案

如zk出现相关异常问题，首先通过相关日志排查对应问题，并评估影响。如问题确实复杂，短期内难以解决，应立即回滚，保证系统稳定

1. 删除老zk集群的快照文件及事务日志，分别对应 /opt/zookeeper/data/version-2， /opt/zookeeper/log/version-2
    
2. 将新zk集群的最新快照文件和事务日志copy到老zk集群对应文件夹中，并重启zk
    

scp /opt/zookeeper/data/version-2/snapshot.3b03f14 172.26.25.109:/opt/zookeeper/data/version-2/

scp /opt/zookeeper/log/version-2/log.3b03969f15 172.26.25.109:/opt/zookeeper/log/version-2/

3.修改ck配置文件/etc/clickhouse-server/metrika.xml，将zookeeper-servers配置修改为老zk集群

相关回滚操作，已经编写完对应的操作脚本，提高操作效率，减少操作错误，确保整个回滚操作可以在10分钟内完成。

##### 监控与保障

1. zk集群节点机器已配置对应的cpu，内存，磁盘告警
    
2. zk集群进程可用性监控及关键指标报警已完成配置
    
3. zk相关指标看板已搭建完成
    
4. 切换前后值班人员：姚伟，周梦林
    
5. 关联系统人员：数仓，彭吉凯。洞察，王德锋