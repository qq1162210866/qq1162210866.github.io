

redolog和binglog

mvcc

redo log如何保证数据的完整

redo log就是	ib_logfile0,里面存放的是二进制数据

双阶段提交中间失败了如何处理

为什么日志会比直接写数据库快





高性能MySQL





windows下的mysql开启binlog会因为权限的问题导致配置文件不起效

可以在git命令界面执行chmod 0444 *.cnf

然后重启镜像即可



查询binlog的格式

```sql
show variables like "%binlog_format%";
```

```
# 查看binlog是否开启
show variables like '%log_bin%';
# 查看binlog格式
show variables like "%binlog_format%";
# 查看binlog文件有哪些
show binary logs;
# 查看binlog内容
show binlog events in '文件名'
```



查看bilog需要使用mysqlbinlog

查询row需要加上vv参数

mysqlbinlog -vv  /var/lib/mysql/mysql-bin.000001



查看binlog

https://www.cnblogs.com/softidea/p/12624778.html



redo和undolog

https://www.cnblogs.com/f-ck-need-u/p/9010872.html



redo读取工具

https://dev.mysql.com/doc/refman/5.7/en/mysql-cluster-programs-ndb-redo-log-reader.html

根据加锁的范围，MySQL里面的锁大致可以分成全局锁、表级锁和行锁三类







### mysql创建分区表

语法：

```sql
create table if not exists test_partition
(
    id      int          not null,
    address varchar(200) null,
    dt      int          not null
) PARTITION BY RANGE (dt) (
    partition part202110 values less than (20211032),
    partition part202111 values less than (20211132),
    partition part202112 values less than (20211232))


select *
from information_schema.partitions
where table_schema = database()
  and table_name = 'test_partition';

insert into test_partition value (4, "测试", 20211013);

insert into test_partition value (5, "测试", 20210912);

insert into test_partition value (3, "测试", 20221018);

alter table test_partition partition by range (dt)(
    partition part202109 values less than (20210932),
    partition part202110 values less than (20211032),
    partition part202111 values less than (20211132),
    partition part202112 values less than (20211232),
    partition partmax values less than MAXVALUE);

#之前的验证方式
show variables like '%partition%';
## 现在的验证方式
show plugins;




explain partitions select * from test_partition where dt BETWEEN '20140101'  and  '20211011';

explain partitions select * from test_partition where dt =20210912;

show table status

#删除分区
alter table test_partition drop partition partmax;

#增加分区，只适用于往后添加，不能在前面添加
alter table test_partition add partition (partition part202109 values less than (20210932));


SELECT * FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_NAME = 'test_partition'

```

更新分区后，关于分区的一些记录就不准确了，行数都会清为0



删除分区会一起删除记录

分区只能递增，无法递减

增加分区无法在之前添加，只能全部重新构建



RANGE分区是范围分区，某个范围的在同一个分区表中

List分区是离散分区，一些固定值的记录在同一个分区表中

HASH分区的目的是将数据均匀地分布到预先定义的各个分区中

KEY分区和HASH分区相似，但是使用的分区函数是mysql自己的

注意：

- 所有的整形类型，如INT、SMALLINT、TINYINT和BIGINT。而FLOAT和DECIMAL则不予支持。
- 日期类型，如DATE何DATETIME。其余的日期类型不予支持。
- 字符串类型，如CHAR、VARCHAR、BINARY和VARBINARY。而BLOB和TEXT类型不予支持。
- null值的分区插入因为分区的不同结果也会不同



https://blog.csdn.net/youzhouliu/article/details/52757043	关于分区语句的使用和测试

https://www.cnblogs.com/dw3306/p/12620042.html	一些概念的介绍
