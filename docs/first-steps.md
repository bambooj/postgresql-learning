## 迈出第一步
英文目录：
https://www.amazon.cn/dp/B00X3TVFXQ/ref=redir_mobile_desktop?_encoding=UTF8&%2AVersion%2A=1&%2Aentries%2A=0

#### 1.1 介绍
开源免费的先进的数据库、使用成本低；
主要特性：
良好支持SQL 2011 及以下版本标准；
客户端/服务端架构；
高并发设计，读与写不阻塞；
对许多类型的应用程序具有高度的可配置性和可扩展性；
出色的可扩展性和具有广泛调优特性的性能；
支持大量类型的数据模型：关系型，文档型（JSON和XML）以及key/value 型。

#### 1.2 获取PostgreSQL
下载地址： https://www.postgresql.org/download/

#### 1.3 链接到PostgreSQL
连接URL格式：
psql postgres://username:password@host:5432/mydb
查询当前数据库命令：
select current_database();
显示当前用户ID：
select current_user;
显示当前连接到的IP地址和端口，如果你使用的是Unix域套接字，则两个值都是NULL
select inet_server_addr(),inet_server_port();
9.1版本之后，直接使用\conninfo
结果显示：You are connected to database "DIAMOND-SIT" as user "postgres" on host "localhost" at port "5435".

#### 1.4 启用网络/远程用户访问
初始配置设置为禁止远程访问。

#### 1.4.1 修改postgresql.conf文件(该文件非常重要)，修改如下


如果不知道postgresql.conf文件位置，可以切换到postgres用户：su -l postgres，然后查询文件：find / -name postgresql.conf
#### 1.4.2 修改pg_hba.conf文件，从而允许所有用户和使用加密的密码访问所有数据库，password可以为md5

#### 1.4.3 重启postgres服务

#### 1.5 使用图形化管理工具
#### 1.6 使用psql查询和脚本工具
连接数据库格式：
psql -h hostname -p 5432 -d mydatabase -U myuser
等同于
psql postgres://myuser:mypassword@hostname:port/mydatabase

连接数据库
psql -h localhost -p 5435 -d DIAMOND-SIT -U postgres
帮助信息
\? psql元命令帮助信息
\h sql命令帮助信息
#### 1.7 安全地修改你的密码
#### 1.8 避免硬编码你的密码
#### 1.9 使用连接服务文件
#### 1.10 链接失败故障排查
