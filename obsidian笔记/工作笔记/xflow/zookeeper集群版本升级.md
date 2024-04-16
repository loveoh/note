#### 背景
生产zk集群部署版本为3.7.1版本。由于发现系统漏洞，需要进行版本升级，新版本为3.8.4，
现今zk集群节点：172.26.53.28，172.26.64.167,127.26.64.168
```shell
1、下载安装包
cd /opt/software
# 下载zookeeper安装包
wget https://dlcdn.apache.org/zookeeper/zookeeper-3.8.4/apache-zookeeper-3.8.4-bin.tar.gz
# 解压zookeeper安装包
tar -zxvf apache-zookeeper-3.8.4-bin.tar.gz
mv apache-zookeeper-3.8.4-bin /opt/zookeeper

2、老数据备份
cp /opt/zookeeper/data /opt/zookeeper/data3.7.1 
cp /opt/zookeeper/log  /opt/zookeeper/log3.7.1

3、迁移数据及配置
 将老版本的配置文件进行复制替换java.env,zoo.cfg,log4j.properties

#zk集群相关操作命令
/opt/zookeeper/apache-zookeeper-3.7.1-bin/bin/zkServer.sh stop
/opt/zookeeper/apache-zookeeper-3.7.1-bin/bin/zkServer.sh start
/opt/zookeeper/apache-zookeeper-3.7.1-bin/bin/zkServer.sh status
```
##### zk集群切换操作步骤
1. 选择业务低峰期时间段，停止所有ck的写入任务，包括离线任务，及实时任务（实时任务id：2948,10295，2959,10379,10626）
2. 将老版本的zk集群的对应配置文件（java.env,zoo.cfg,log4j.properties）迁移到对应的新版本集群中。
3. 通过/bin/zkServer.sh status命令，找到对应的leader节点，从follower节点逐级将集群停止。
4. 通过/bin/zkServer.sh start命令分别启动新zk集群，并观察对应zk的状态。
5. 如果zk集群节点为1主2从，并且没有明显异常日志报错，则证明zk集群启动成功。
##### 业务验证
1. 平台功能验证，数据可以查询，可以写入，可以做相关元数据操作，如删除分区，建表，删表，删列等。
2. 任务相关验证，实时flink任务可以正常写入，datax任务数据可正常插入
##### 异常回滚方案
如zk出现相关异常问题，首先通过相关日志排查对应问题，并评估影响。如问题确实复杂，短期内难以解决，应立即回滚，保证系统稳定
1.  通过bin/zkServer.sh stop命令，从follower节点开始逐级停止zk集群服务。
2. 将备份好的数据进行回滚，cp /opt/zookeeper/data3.7.1  /opt/zookeeper/data
	cp /opt/zookeeper/log3.7.1  /opt/zookeeper/log
3. 将3.7.1版本的zk集群服务重启，并观察集群状态。
##### 监控与保障
1. zk集群节点机器已配置对应的cpu，内存，磁盘告警
2. zk集群进程可用性监控及关键指标报警已完成配置
3. zk相关指标看板已搭建完成
4. 切换前后值班人员：姚伟，周梦林
5. 关联系统人员：数仓，彭吉凯