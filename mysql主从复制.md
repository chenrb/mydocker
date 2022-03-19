### 1、新建主服务器容器实例3307
```
docker run -p 3306:3306 -d --name mysql-master --env-file /data/mysql/env --privileged=true -v /data/mysql/conf/:/etc/mysql/conf.d -v /data/mysql/log:/var/log/mysql -v /data/mysql/data:/var/lib/mysql mysql:5.7
```
### 2、修改主数据库conf，并重启docker
```
[mysqld]
user=mysql
default-storage-engine=INNODB
character-set-server=utf8

port            = 3306

basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
skip-name-resolve  # 这个参数是禁止域名解析的，远程访问推荐开启skip_name_resolve。

# 主从同步
server-id=1
binlog-do-db=share
log-bin=mysql-bin

[client]
port = 3306
default-character-set=utf8

[mysql]
no-auto-rehash
default-character-set=utf8
```
`docker restart xxxx`

### 3、创建从数据库
```
挂载目录：/data/mysql-slave/conf
```
```
[mysqld]
user=mysql
default-storage-engine=INNODB
character-set-server=utf8

port            = 3306

basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
skip-name-resolve  # 这个参数是禁止域名解析的，远程访问推荐开启skip_name_resolve。

# 主从同步
server-id=2
binlog-do-db=share
log-bin=mysql-bin

[client]
port = 3306
default-character-set=utf8

[mysql]
no-auto-rehash
default-character-set=utf8
```
```
docker run -p 3307:3306 -d --name mysql-slave --env-file /data/mysql/env --privileged=true -v /data/mysql-slave/conf/:/etc/mysql/conf.d -v /data/mysql-slave/log:/var/log/mysql -v /data/mysql-slave/data:/var/lib/mysql mysql:5.7
```
### 4、创建主服务器复制用户及相关权限
```
create user 'slave'@'%' identified by '123456';//创建用户
grant replication slave,replication client on *.* to 'slave'@'%';//设置用户权限
flush privileges;//刷新权限
show grants for 'slave'@'%';//查看用户权限
```
### 5、主服务器查看状态
```
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      769 | share        |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```
### 6、从服务器开始复制
```
进入从服务器
mysql> change master to master_host='xxxxxx', master_port=3306, master_user='slave', master_password='xxxxx', master_log_file='mysql-bin.000001', master_log_pos=769;
mysql> start slave;
mysql> show slave status \G;
```