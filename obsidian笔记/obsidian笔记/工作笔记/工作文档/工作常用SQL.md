```sql #按天查看直营表PV UV
select count(SID),count(distinct SID) ,toDate(event_time) as t from sdk_event_record_zhiying where event_time > '2023-03-20 10:00:00' 
and event_time < '2023-03-20 12:59:59' and source_id = 'moc4dy1np1d8iszz'  group by t;

select count(SID),count(distinct SID) ,toHour(event_time) as t from sdk_event_record_health where event_time > '2023-03-21 00:00:00' 
and event_time < '2023-03-21 23:59:59'  group by t;

#新增表字段
ALTER TABLE xflow.sdk_event_record_zad_local on cluster xflow_cluster add COLUMN receive_time DateTime;

ALTER TABLE xflow.sdk_event_record_zad_local_hour_paritition on cluster xflow_cluster add COLUMN receive_time DateTime;

ALTER TABLE xflow.sdk_event_record_zad_local_move1 on cluster xflow_cluster add COLUMN receive_time DateTime;

ALTER TABLE xflow.sdk_event_record_zad_local_move2 on cluster xflow_cluster add COLUMN receive_time DateTime;

ALTER TABLE xflow.sdk_event_record_zad_local_move3 on cluster xflow_cluster add COLUMN receive_time DateTime;



select count() , toMinute(event_time) as t from  query_log where toDate(event_time) = '2023-03-29' and event_time > '2023-03-29 10:25:00' and event_time < '2023-03-29 10:35:59' and type = 'QueryFinish' and user in ('xflow','insight') and query like 'select%' and query like '%from xflow%' group by t limit 100 \G

drop table xflow.sdk_event_record_zad_local_hour_paritition on cluster xflow_cluster;

// move离线数据
ALTER TABLE xflow.sdk_event_record_zad_local ON CLUSTER xflow_cluster drop PARTITION 20240318;

ALTER TABLE xflow.sdk_event_record_zad_local_move3 ON CLUSTER xflow_cluster MOVE PARTITION 20240318 TO TABLE xflow.sdk_event_record_zad_local;

ALTER TABLE xflow.sdk_event_record_zad_local_move2 ON CLUSTER xflow_cluster MOVE PARTITION 20230525 TO TABLE xflow.sdk_event_record_zad_local;

insert into xflow.sdk_event_record_zhiying_local_move3 select * from xflow.sdk_event_record_zhiying_local_hour_paritition where toDate(event_time) = '2023-08-10'

// 查询表大小及所在次磁盘路径
 SELECT 
 -- partition,
disk_name,
 table,
    sum(rows) AS `总行数`,
    formatReadableSize(sum(data_uncompressed_bytes)) AS `原始大小`,
    formatReadableSize(sum(data_compressed_bytes)) AS `压缩大小`,
    round((sum(data_compressed_bytes) / sum(data_uncompressed_bytes)) * 100, 0) AS `压缩率`
    -- ,disk_name
FROM system.parts
 where active
  -- and database = 'xflow'
  -- and table = 'sdk_event_record_health_local'
  -- and partition>='20210201' and partition<='20220331' 
  -- and partition<'20230325'
   and disk_name ='hdd_disk_1'
 group by 
 -- partition,
 table,
 disk_name
order by sum(data_compressed_bytes) desc limit 10




-- datax离线任务推送表统计
select
	toHour(event_time) as t ,
	min(event_time) as minTime,
	max(event_time) as maxTime,
	tables,
	count() as c
from
	system.query_log
where
	event_date = '2023-08-25'
	and event_time > '2023-08-25 00:00:00'
	and event_time < '2023-08-25 23:00:00'
	and (address = toIPv6('::ffff:172.20.51.33')
		or address = toIPv6('::ffff:10.253.124.160')
			or address = toIPv6('::ffff:172.26.53.27')
				or address = toIPv6('::ffff:172.20.50.90')
					or address = toIPv6('::ffff:172.26.64.160')
						or address = toIPv6('::ffff:192.168.0.2')
							or address = toIPv6('::ffff:172.26.64.159'))
group by
	t,
	tables
order by
	t ,
	minTime;


alter table xflow.sdk_event_record_health_local on cluster xflow_cluster drop partition 

-- 神盾局开通白名单
INSERT INTO `risk_platform`.`risk_auth_white_list` (user_id, user_name, is_deleted, gmt_created, gmt_modified, creator, modifier) VALUES(NULL, 'wupeili', 'N', now(), now(), 'system', 'system');

-- 修复ck表的ttl策略

alter table xflow.sdk_event_record_other_local_hour_paritition on cluster xflow_cluster modify  TTL event_time + toIntervalDay(3)

kill query where query_id = '8dedb84a-d1d0-42dd-ad83-a991e8dd26df'

insert into xflow.sdk_event_record_zad_local_move3 select * from xflow.sdk_event_record_zad_local_hour_paritition where toDate(event_time) = '2024-03-24'

```
