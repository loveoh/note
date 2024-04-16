**背景：**
通过监控发现ck节点内存及poolTask指标异常，然后进行排查。03/04节点指标明显异于其他节点。
![[Pasted image 20240325185311.png]]
```sql
 select node_name,source_replica,create_time from system.replication_queue where table = 'sdk_event_record_zad_local' order by create_time\G

```
![[Pasted image 20240325192821.png]]

![[Pasted image 20240325185549.png]]
```sql
-- 
SELECT replica_path || '/queue/' || node_name
FROM system.replication_queue JOIN system.replicas
USING (database, table) WHERE source_replica = 'cluster01-02-1'
```
![[Pasted image 20240325192238.png]]
通过python脚本，将zk上的节点删除。并重启ck节点

```python

#
from kazoo.client import KazooClient  

zk = KazooClient(hosts='172.26.64.168:2181')  
path215_1 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-2/queue/queue-0015625246'  
path215_2 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-2/queue/queue-0015625360'  
path215_3 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-2/queue/queue-0015625389'  
path215_4 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-2/queue/queue-0015625424'  
path215_5 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-2/queue/queue-0015625853'  
path215_6 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-2/queue/queue-0015629427'  
  
path214_1 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-1/queue/queue-0015625253'  
path214_2 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-1/queue/queue-0015625332'  
path214_3 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-1/queue/queue-0015625367'  
path214_4 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-1/queue/queue-0015625545'  
path214_5 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-1/queue/queue-0015625860'  
path214_6 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-1/queue/queue-0015629434'  
path214_7 = '/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-1/queue/queue-0015625396'  
  
zk.start()  
# data, stat = zk.get('/clickhouse_xflow/clickhouse/tables/809f69eb-5d32-4619-8fde-eac17c908bfd/02/replicas/cluster01-02-2/queue/queue-0015625325')  
# print(data)  
zk.delete(path215_1)  
zk.delete(path215_2)  
zk.delete(path215_3)  
zk.delete(path215_4)  
zk.delete(path215_5)  
zk.delete(path215_6)  
print("end214")  
zk.delete(path214_1)  
zk.delete(path214_2)  
zk.delete(path214_3)  
zk.delete(path214_4)  
zk.delete(path214_5)  
zk.delete(path214_6)  
zk.delete(path214_7)  
print("end215")  
zk.stop()
```