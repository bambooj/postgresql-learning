#### 2.1 介绍
#### 2.2 当前服务器的版本号是多少
```
DIAMOND=# select version();
                                                 version

--------------------------------------------------------------------------------
-------------------------
 PostgreSQL 9.6.1 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (
Red Hat 4.8.5-4), 64-bit
(1 row)
```
PostgreSQL服务程序的版本号结构：
> 主版本号.小版本号.维护版本号

另外：查看客户端版本
> psql --version


#### 2.3 服务程序运行时间是多少
使用pg_postmaster_start_time()函数

```
select date_trunc('second', current_timestamp - pg_postmaster_start_time()) as uptime;
```

程序启动时间
```
DIAMOND-SIT=# select pg_postmaster_start_time();
   pg_postmaster_start_time
-------------------------------
 2018-04-24 17:38:25.961051+08
(1 row)
```


#### 2.4 定位数据库服务的数据文件
CentOS默认数据库的数据文件路径：/var/lib/pgsql/data

#### 2.5 定位数据库服务的日志文件

修改日志配置: vim ./postgresql.conf
```
...
log_destination = 'stderr'              # Valid values are combinations of
                                        # stderr, csvlog, syslog, and eventlog,
                                        # depending on platform.  csvlog
                                        # requires logging_collector to be on.

# This is used when logging to stderr:
logging_collector = on          # Enable capturing of stderr and csvlog
                                        # into log files. Required to be on for
                                        # csvlogs.
                                        # (change requires restart)

# These are only used if logging_collector is on:
log_directory = '/data/postgres/logs'           # directory where log files are written,
             # can be absolute or relative to PGDATA
...        
```

重启postgresql服务
```
/usr/local/postgresql-9.6.1/bin/pg_ctl restart -D /home/postgres/data/postgresql-9.6.1
```

日志文件目录：/data/postgres/logs
```
[postgres@iZwz96377q7u5t1h1ueahaZ postgresql-9.6.1]$ ll /data/postgres/logs/
total 4
-rw-rw-r--+ 1 postgres postgres 212 Apr 24 23:56 postgresql-2018-04-24_235632.log
```

#### 2.6 定位数据库的系统标识
**pg_controldata** 是一个PostgreSQL服务端程序，用于现实服务器的控制文件内容。

```
[postgres@iZwz96377q7u5t1h1ueahaZ logs]$ pg_controldata
WARNING: Calculated CRC checksum does not match value stored in file.
Either the file is corrupt, or it has a different layout than this program
is expecting.  The results below are untrustworthy.

pg_control version number:            960
Catalog version number:               201608131
Database system identifier:           6378697532909377939
Database cluster state:               in production
pg_control last modified:             Tue 24 Apr 2018 11:56:32 PM CST
Latest checkpoint location:           FE1AE2C0/8
Prior checkpoint location:            FE1AE250/8
Latest checkpoint's REDO location:    FE1AE2C0/8
Latest checkpoint's TimeLineID:       1
Latest checkpoint's full_page_writes: on
Latest checkpoint's NextXID:          1/0
Latest checkpoint's NextOID:          3303315
Latest checkpoint's NextMultiXactId:  181614
Latest checkpoint's NextMultiOffset:  1
Latest checkpoint's oldestXID:        0
Latest checkpoint's oldestXID's DB:   1748
Latest checkpoint's oldestActiveXID:  1524585391
Time of latest checkpoint:            Sun 07 Feb 2106 02:28:17 PM CST
Minimum recovery ending location:     0/0
Backup start location:                0/0
Backup end location:                  1/0
End-of-backup record required:        no
Current wal_level setting:            minimal
Current max_connections setting:      0
Current max_prepared_xacts setting:   0
Current max_locks_per_xact setting:   0
Maximum data alignment:               0
Database block size:                  0
Blocks per segment of large relation: 0
WAL block size:                       0
Bytes per WAL segment:                500
Maximum length of identifiers:        8
Maximum columns in an index:          0
Maximum size of a TOAST chunk:        64
Date/time type storage:               floating-point numbers
Float4 argument passing:              by reference
Float8 argument passing:              by reference
```

其中：Database system identifier:           6378697532909377939
就是系统标识符信息。
> 注：不要修改 PostgreSQL 的控制文件。

#### 2.7 列出数据库服务中的数据库
使用psql连接数据库，使用"\l"元命令显示所有数据库。
或者查询pg_database系统表。
```
DIAMOND=# \l
                                     List of databases
    Name     |  Owner   | Encoding |   Collate   |    Ctype    |     Access privileges
-------------+----------+----------+-------------+-------------+---------------------------
 DIAMOND     | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | =Tc/postgres             +
             |          |          |             |             | postgres=CTc/postgres    +
             |          |          |             |             | diamond_app=c/postgres   +
             |          |          |             |             | diamond_report=c/postgres
 DIAMOND-SIT | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 |
 postgis     | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 |
 postgres    | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 |
 template0   | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | =c/postgres              +
             |          |          |             |             | postgres=CTc/postgres
 template1   | postgres | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | =c/postgres              +
             |          |          |             |             | postgres=CTc/postgres
(6 rows)

DIAMOND=# select datname from pg_database;
   datname
-------------
 postgres
 postgis
 template1
 template0
 DIAMOND
 DIAMOND-SIT
(6 rows)
```

#### 2.8 数据库中有多少张表

```
DIAMOND=# select count(*) from information_schema.tables where table_schema not in ('information_schema','pg_catalog');
-[ RECORD 1 ]
count | 86
```

pg_catalog: 系统信息表；

information_schema: 信息模式的系统对象表；

需要排除这两个。才得出用户表。

#### 2.9 一个数据库占用了多少磁盘空间
查询当前数据库大小
```
DIAMOND=# select pg_database_size(current_database());
-[ RECORD 1 ]----+----------
pg_database_size | 439441944
```

计算所有数据库大小
```
DIAMOND=# select sum(pg_database_size(datname)) from pg_database;
-[ RECORD 1 ]---
sum | 1870945384
```
#### 2.10 一张表占用了多少磁盘空间

```
DIAMOND=# select pg_relation_size('dmd_cities');
 pg_relation_size
------------------
            32768
(1 row)
```

表的总大小，包括所有和其他一些相关的空间占用。
```
DIAMOND=# select pg_total_relation_size('dmd_cities');
 pg_total_relation_size
------------------------
                 139264
(1 row)
```

通过元命令查询

```
DIAMOND=# \dt+ dmd_cities
                      List of relations
 Schema |    Name    | Type  |  Owner   | Size  | Description
--------+------------+-------+----------+-------+-------------
 public | dmd_cities | table | postgres | 64 kB |
(1 row)
```

#### 2.11 那张表示最大的表

使用PostgreSQL专用函数pg_relation_size，语句：
> select table_name ,pg_relation_size(table_schema || '.' || table_name) as size from information_schema.tables where table_schema NOT IN ('information_schema','pg_catalog') order by size DESC limit 10;

```
DIAMOND=# select table_name ,pg_relation_size(table_schema || '.' || table_name) as size from information_schema.tables where table_schema NOT IN ('information_schema','pg_catalog') order by size DESC limit 10;
                  table_name                   |   size    
-----------------------------------------------+-----------
 dmd_application_logs                          | 233283584
 dmd_history_customer_flow_location_statistics |  31195136
 dmd_customer_label_statistics                 |  26255360
 dmd_customer_labels                           |  23199744
 dmd_customer_label_statistics1                |  11739136
 dmd_shop_messages                             |  10076160
 dmd_cornucopia_flow_records                   |   5947392
 dmd_owner_resources                           |   2162688
 dmd_history_customer_flow_statistics          |   1662976
 dmd_customer_attribution_statistics           |   1122304
(10 rows)
```

#### 2.12 表里有多少行记录
```
DIAMOND=# select count(*) from dmd_application_logs;
 count 
-------
 64987
(1 row)
```


#### 2.13 快速估算表里的记录总数

这里是估算表dmd_application_logs的记录。
```
DIAMOND=# select (CASE WHEN reltuples > 0 THEN pg_relation_size('dmd_application_logs')*reltuples/(8192*relpages) ELSE 0 END)::bigint AS estimated_row_count from pg_class where oid = 'dmd_application_logs'::regclass;
 estimated_row_count 
---------------------
               60201
(1 row)
```

> 估计行数 = 数据快数 * 每块的行数

#### 2.14 列出数据库中的扩展模块

通过pg_extension表查询所有已安装的扩展。或者直接使用\dx命令
```
DIAMOND=# select * from pg_extension;
 extname  | extowner | extnamespace | extrelocatable | extversion | extconfig | extcondition 
----------+----------+--------------+----------------+------------+-----------+--------------
 plpgsql  |       10 |           11 | f              | 1.0        |           | 
 pgcrypto |       10 |       123145 | t              | 1.3        |           | 
(2 rows)
```
#### 2.15 理解对象依赖关系

```
DIAMOND=# \d+ dmd_cities
                                        Table "public.dmd_cities"
    Column     |       Type        |         Modifiers          | Storage  | Stats target | Description  
---------------+-------------------+----------------------------+----------+--------------+--------------
 id            | character varying | not null default next_id() | extended |              | 
 name          | character varying | not null                   | extended |              | 
 initials      | character varying |                            | extended |              | 
 english_name  | character varying |                            | extended |              | 
 country_name  | character varying | not null                   | extended |              | 
 province_name | character varying | not null                   | extended |              | 
 status        | boolean           | default false              | plain    |              | 
 code          | character varying |                            | extended |              | 城市对应编码
Indexes:
    "dmd_cities_pkey" PRIMARY KEY, btree (id)
    "dmd_cities_unique" UNIQUE CONSTRAINT, btree (name, country_name, province_name)
Has OIDs: no

DIAMOND=# select * from pg_constraint where confrelid='dmd_cities'::regclass;
 conname | connamespace | contype | condeferrable | condeferred | convalidated | conrelid | contypid | conindid | confrelid | confupdtype | confdeltype | confmatchtype |
 conislocal | coninhcount | connoinherit | conkey | confkey | conpfeqop | conppeqop | conffeqop | conexclop | conbin | consrc 
---------+--------------+---------+---------------+-------------+--------------+----------+----------+----------+-----------+-------------+-------------+---------------+
------------+-------------+--------------+--------+---------+-----------+-----------+-----------+-----------+--------+--------
(0 rows)

```
