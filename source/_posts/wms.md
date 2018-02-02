---
title: WMS 部署手册
---

# WMS 部署手册

## 架构概述
- 全部设计面向云架构，服务化、模块化
- 全部核心采用成熟、免费、开源的框架和组件
- 全部开发、部署、运维过程采用容器技术
- 自主知识产权的企业级服务框架
- 系统级别多租户设计
- 免插件

### 技术架构
![部署架构](技术架构.png)

### 部署架构
![部署架构](部署架构.png)

### 网络架构
![部署架构](web架构.png)

## 环境准备
- OS: CentOS Linux 7.3 64bit
- JDK: jdk 8u152
- Tomcat: Tomcat 8.0.48
- 应用目录：/apps/
  - conf/ : 应用配置文件目录
  - svr/  : 编译后的软件目录
  - sh/   : 运维常用脚本目录
  - logs/ : 日志文件目录
  - rpm/  : 常用软件工具包
  - tmp/  : 临时目录
  - run/  : 进程PID文件

### 环境数量

| 环境类型     | 目的                                                                                              | 服务器数量 | 注释                                                                                                            |
| ------------ | ------------------------------------------------------------------------------------------------- | ---------- | --------------------------------------------------------------------------------------------------------------- |
| UAT测试环境  | 由测试工程师使用的测试环境，专门用于做集成测试/由业务代表执行的测试，用于测试功能是否满足业务需求 | 7          | 此环境将使用同最终生产环境相同的操作系统版本和产品版本。相比生产环境它具有较少的内存和CPU，但具有较真实的数据。 |
| PROD生产环境 | 最终用户作企业生产使用                                                                            | 20         | 由最终业务用户使用，任何客制化开发出来的代码、升级等，在安装到生产系统前，已经过测试，一般采用集群部署          |

### UAT测试环境

| 部署规划           | 说明 | 硬件类型 | 数量 | 配置                  | 软件环境                      |
| ------------------ | ---- | -------- | ---- | --------------------- | ----------------------------- |
| MySql              |      | 虚拟机   | 1    | 8cpu+32g内存+1T硬盘   | MariaDB 10.2                  |
| MongoDB            |      | 虚拟机   | 1    | 8CPU+16g内存+200G硬盘 | MongoDB 3.6                   |
| Redis、RabbitMQ    |      | 虚拟机   | 1    | 8CPU+16g内存+200G硬盘 | RabbitMQ 3.6.14、Redis 3.2.11 |
| WMS APP            |      | 虚拟机   | 1    | 8CPU+16g内存+200G硬盘 | JDK 8、Tomcat 8.0             |
| WMS RF、WMS REPORT |      | 虚拟机   | 1    | 8CPU+16g内存+200G硬盘 | JDK 8、Tomcat 8.0             |
| WMS JOB            |      | 虚拟机   | 1    | 8CPU+16g内存+200G硬盘 | JDK 8、Tomcat 8.0             |
| Nginx              |      | 虚拟机   | 1    | 8CPU+16g内存+200G硬盘 | Nginx  1.12                   |

### PROD生产环境

| 部署规划                       | 说明                                                                                                                                                                                                             | 硬件类型 | 数量             | 配置                    | 软件环境                       |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ---------------- | ----------------------- | ------------------------------ |
| MySQL(双Master + Keepalived)   | 关系数据库，存储业务及相关数据，采用双主+Keepalived高可用架构，两个节点采用双主模式，并且放置于同一个VLAN中，在master节点发生故障后，利用keepalivedt的高可用机制实现快速切换到slave节点 VIP                      | 物理机   | 2                | 2 * 8cpu+64g内存+1T硬盘 | MariaDB 10.2                   |
| MongoDB(Relica Set + Sharding) | 分布式文档数据库，存储业务配置、业务流程日志、条码数据等，采用副本集+数据库分片技术架构，支持数据冗余故障恢复、读写分离、大数据横向切分,数据基本只做存储，没有很复杂的查询计算逻辑，内存足够，保证足够的磁盘空间 | 虚拟机   | 3                | 8CPU+32g内存+2T硬盘     | MongoDB 3.6                    |
| Redis(Sentinel Master-Slave)   | 缓存中间件，缓存业务相关数据、分布式session，采用哨兵模式架构，支持自动故障迁移                                                                                                                                  | 虚拟机   | 2                | 8CPU+16g内存+200G硬盘   | Redis 3.2.11                   |
| RabbitMQ                       | 分布式消息服务中间件，用于与业务流程解耦、异步处理，保证系统的稳定性和可靠性，提高系统吞吐量                                                                                                                     | 虚拟机   | 与redis共用      | 8CPU+16g内存+200G硬盘   | RabbitMQ 3.6.14                |
| WMS APP                        | wms主应用服务                                                                                                                                                                                                    | 虚拟机   | 3                | 8CPU+16g内存+200G硬盘   | JDK 8、Tomcat 8.0              |
| WMS RF                         | wms 手持RF、PDA服务、PDA                                                                                                                                                                                         | 虚拟机   | 2                | 8CPU+16g内存+200G硬盘   | JDK 8、Tomcat 8.0              |
| WMS REPORT                     | wms报表服务，共享存储                                                                                                                                                                                            | 虚拟机   | 2                | 8CPU+16g内存+200G硬盘   | JDK 8、Tomcat 8.0              |
| WMS JOB                        | wms定时任务服务                                                                                                                                                                                                  | 虚拟机   | 与报表服务器共用 | 8CPU+16g内存+200G硬盘   | JDK 8、Tomcat 8.0              |
| Nginx + Keepalived             | HTTP反向代理、负载均衡服务器                                                                                                                                                                                     | 虚拟机   | 2                | 4CPU+8g内存+200G硬盘    | Nginx  1.12 + Keepalived 1.3.9 |

## 环境部署

### 数据库服务

#### MySQL

#### MySQL双主 + Keepalived主从自动切换
```
+-------------+                       +-------------+
|   mysql01   |-----------------------|   mysql02   |
+-------------+                       +-------------+
       |        .                   .         |
       |            .            .            |
       |                .     .               |
       |                   .                  |
       |                .      .              |
       |             .             .          |
       |          .                    .      |
    MASTER     .      keep|alived         BACKUP
192.168.1.101        192.168.1.100      192.168.1.102
+-------------+    +-------------+    +-------------+
| haproxy01   |----|  VirtualIP  |----| haproxy02   |
+-------------+    +-------------+    +-------------+
                          |
       +------------------+------------------+
       |                  |                  |
+-------------+    +-------------+    +-------------+
|   client    |    |   client    |    |   client    |
+-------------+    +-------------+    +-------------+
```

- **安装**

```bash
#!/bin/bash
#
###
# Filename: install_mariadb.sh
# Author: mark.wu - jinqi.wu@midea.com
# Description:
# Last Modified: 2017-12-20 12:10:14
# Version: 1.0
###

MARIADB_GROUP="mysql"
MARIADB_USER="mysql"
MARIADB_VERSION="10.2.11"
ROOT_PASSWD="root_Wms2018"

MARIADB_INSTALL_PATH="/apps/svr/mariadb"
MARIADB_DATA_PATH="/apps/dbdata/mariadb"
MARIADB_CONF_PATH="/apps/conf/mariadb"
MARIADB_LOG_PATH="/apps/logs/mariadb"
MARIADB_TMP_PATH="/apps/tmp/mariadb"
MARIADB_PID_PATH="/apps/run"

# uninstall mariadb-libs
rpm -qa|grep mariadb-libs

# check mariadb user
echo -n "check MariaDB user... "
id -u ${MARIADB_USER} >/dev/null 2>&1
if [ $? -ne 0 ];then
 groupadd ${MARIADB_GROUP}
 useradd -g ${MARIADB_GROUP} ${MARIADB_USER}
fi
echo "ok"

# check install dir
[ ! -d "${MARIADB_INSTALL_PATH}" ] && mkdir -p ${MARIADB_INSTALL_PATH}
[ ! -d "${MARIADB_DATA_PATH}" ] && mkdir -p ${MARIADB_DATA_PATH}
[ ! -d "${MARIADB_CONF_PATH}" ] && mkdir -p ${MARIADB_CONF_PATH}
[ ! -d "${MARIADB_LOG_PATH}" ] && mkdir -p ${MARIADB_LOG_PATH}
[ ! -d "${MARIADB_TMP_PATH}" ] && mkdir -p ${MARIADB_TMP_PATH}
[ ! -d "${MARIADB_PID_PATH}" ] && mkdir -p ${MARIADB_PID_PATH}

# check mariadb file
if [ ! -f mariadb-${MARIADB_VERSION}-linux-x86_64.tar.gz ];then
 wget http://mirrors.tuna.tsinghua.edu.cn/mariadb//mariadb-${MARIADB_VERSION}/bintar-linux-x86_64/mariadb-${MARIADB_VERSION}-linux-x86_64.tar.gz
fi

# untar file
echo -n "untar file ..."
tar -zxvf mariadb-${MARIADB_VERSION}-linux-x86_64.tar.gz
mv mariadb-${MARIADB_VERSION}-linux-x86_64/* ${MARIADB_INSTALL_PATH}/
echo "ok"

# init db and config
echo -n "init my.cnf..."
cp ${MARIADB_INSTALL_PATH}/support-files/my-huge.cnf ${MARIADB_CONF_PATH}/my.cnf

IPADDR=$(ip addr | grep 'state UP' -A2 | tail -n1 | awk '{print $2}' | cut -f1 -d '/')
SERVER_ID=$(echo $IPADDR|awk -F"." '{print $4}')
cat >${MARIADB_CONF_PATH}/my.cnf<<EOF
[client]
port = 3306
socket = /data/dbdata/mysql.sock
default-character-set = utf8

[mysqld]
port = 3306
socket = /data/dbdata/mysql.sock

default-storage-engine = InnoDB
collation-server = utf8_general_ci
init_connect = 'SET NAMES utf8'
character-set-server = utf8

back_log = 512
open_files_limit = 65535
skip-external-locking
skip-name-resolve
pid-file = /data/run/mysql.pid
basedir = /data/svr/mariadb
datadir = /data/dbdata
tmpdir = /data/tmp/

log-error = /data/logs/mysql-error.log
slow_query_log = 0
log-queries-not-using-indexes = 1
long_query_time = 10
slow_query_log_file = /data/logs/mysql-slow.log

server-id = 31
log-bin=/data/logs/mysql-bin
binlog-format = mixed
binlog-checksum = CRC32
expire_logs_days = 7
sync-master-info = 1
sync_relay_log_info = 1
master-verify-checksum = 1
slave-sql-verify-checksum = 1

thread_stack = 20M
sort_buffer_size = 20M
read_buffer_size = 20M
read_rnd_buffer_size = 20M
join_buffer_size = 20M
binlog_cache_size = 20M

query_cache_size = 32M
query_cache_limit = 512K

max_connections = 2048
max_allowed_packet = 120M
max_connect_errors = 5000
concurrent_insert = 2
connect_timeout = 30
max_allowed_packet = 32M

max_heap_table_size = 1G
bulk_insert_buffer_size = 1G
tmp_table_size = 1G

table_open_cache = 2048

thread_concurrency = 8
thread_cache_size = 10

key_buffer_size = 40M

innodb_data_home_dir = /data/dbdata
innodb_data_file_path = ibdata1:128M;ibdata2:10M:autoextend
innodb_log_file_size = 256M
innodb_log_files_in_group = 4
innodb_buffer_pool_size = 10G
innodb_status_file
innodb_file_per_table
innodb_flush_log_at_trx_commit	= 2
innodb_table_locks = 0
innodb_log_buffer_size = 128M
innodb_lock_wait_timeout = 120
innodb_thread_concurrency = 8
innodb_commit_concurrency = 8
innodb_flush_method = O_DIRECT
skip-innodb-doublewrite

wait_timeout = 3600

[mysqldump]
quick
quote-names
max_allowed_packet = 128M

[mysql]
no-auto-rehash
safe-updates

[myisamchk]
key_buffer_size = 256M
sort_buffer_size = 256M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout
EOF
echo "ok"

echo -n "init db..."
${MARIADB_INSTALL_PATH}/scripts/mysql_install_db --defaults-file=${MARIADB_CONF_PATH}/my.cnf --user=${MARIADB_USER} --basedir=${MARIADB_INSTALL_PATH} --datadir=${MARIADB_DATA_PATH}
[ $? -ne 0 ] && exit 1
echo "ok"

# start mariadb
${MARIADB_INSTALL_PATH}/bin/mysqld_safe --defaults-file=${MARIADB_CONF_PATH}/my.cnf --user=${MARIADB_USER} --basedir=${MARIADB_INSTALL_PATH} --datadir=${MARIADB_DATA_PATH} >/dev/null 2>&1 &
[ $? -ne 0 ] && exit 1 || echo "mariadb started ok..."

# set root passwd
sleep 10
ln -s ${MARIADB_DATA_PATH}/mysql.sock /tmp/mysql.sock
${MARIADB_INSTALL_PATH}/bin/mysqladmin -uroot password "${ROOT_PASSWD}"
if [ $? -ne 0 ];then
 echo "change password for root failed..."
 exit 1
else
 echo "change password for root to :${ROOT_PASSWD}..."
fi

# set path
echo "export MARIADB_HOME=/apps/svr/mariadb" >> /etc/profile
echo -e "export PATH=\$PATH:\$MARIADB_HOME/bin" >> /etc/profile
source /etc/profile

echo "mariadb installed successfully..."
```

- **主服务器Master授权配置**

```bash
# 需要输入root密码
mysql -uroot -p
```

```bash
-- 创建Slave专用备份账号
GRANT REPLICATION SLAVE ON *.* TO 'replicater'@'%' IDENTIFIED BY 'dbpass';
flush privileges;

-- 查看授权情况
SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;

-- 锁定数据库防止master值变化
flush tables with read lock;
-- 获取master状态值
show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000002 |      635 |              |                  |
+------------------+----------+--------------+------------------+
```

一旦获取了备份时正确的Binlog位点（文件名和偏移量），那么就可以用BINLOG_GTID_POS()函数来计算GTID

```bash
SELECT BINLOG_GTID_POS("mysql-bin.000002", 635);
+------------------------------------------+
| BINLOG_GTID_POS("mysql-bin.000002", 635) |
+------------------------------------------+
| 0-56-2                                   |
+------------------------------------------+
```

- **从服务器Slave 配置**

```bash
# 需要输入root密码
mysql -uroot -p
```
```bash
-- 记得修改为你当前系统的值
SET GLOBAL gtid_slave_pos = "0-56-2";
-- 进行主从授权(记得修改相应的host ip)
change master to master_host='172.18.88.44',MASTER_PORT = 3306,master_user='replicater',master_password='dbpass',master_use_gtid=slave_pos;
START SLAVE;
show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.18.88.44
                  Master_User: replicater
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 635
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 654
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
            ..........................
                   Using_Gtid: Slave_Pos
                  Gtid_IO_Pos: 0-56-2

```
如果 Slave_IO_Running 与 Slave_SQL_Running 都为YES,表明从服务已经运行，Using_Gtid列判断GTID值是否一致。
```bash
说明：
master_host 表示master授权地址
MASTER_PORT MySQL端口
master_user 表示master授权账号
master_password 表示密码
master_use_gtid GTID变量值
```

- **解锁主服务器数据库表**

```bash
# 需要输入root密码
mysql -uroot -p
```
```bash
-- 解锁数据表
unlock tables;
-- 查看从服务器连接状态
show slave hosts;
-- 查看客户端
show global status like "rpl%";
```

- **从服务器Slave查看relay的所有相关参数**

```bash
show variables like '%relay%';
```

#### Mongo

![mongo_sharded_cluster](sharding.png)

**mongos**：数据库集群请求的入口，所有的请求都通过mongos进行协调，不需要在应用程序添加一个路由选择器，mongos自己就是一个请求分发中心，它负责把对应的数据请求请求转发到对应的shard服务器上。在生产环境通常有多mongos作为请求的入口，防止其中一个挂掉所有的mongodb请求都没有办法操作。

**config server**：顾名思义为配置服务器，存储所有数据库元信息（路由、分片）的配置。mongos本身没有物理存储分片服务器和数据路由信息，只是缓存在内存里，配置服务器则实际存储这些数据。mongos第一次启动或者关掉重启就会从 config server 加载配置信息，以后如果配置服务器信息变化会通知到所有的 mongos 更新自己的状态，这样 mongos 就能继续准确路由。在生产环境通常有多个 config server 配置服务器，因为它存储了分片路由的元数据，这个可不能丢失！就算挂掉其中一台，只要还有存货，mongodb集群就不会挂掉。

**shard**：这就是传说中的分片了。在mongodb集群只要设置好了分片规则，通过mongos操作数据库就能自动把对应的数据操作请求转发到对应的分片机器上。在生产环境中分片的片键可要好好设置，这个影响到了怎么把数据均匀分到多个分片机器上，不要出现其中一台机器分了1T，其他机器没有分到的情况，这样还不如不分片！

**replica set**：在高可用性的分片架构还需要对于每一个分片构建 replica set 副本集保证分片的可靠性。生产环境通常是 2个副本 + 1个仲裁。

- **安装**
![](mongo.png)

```bash
#!/bin/bash
#
###
# Filename: install_mongo.sh
# Author: mark.wu - jinqi.wu@midea.com
# Description:
# Last Modified: 2017-12-20 12:10:14
# Version: 1.0
###
source init.sh

# check mongo tar file
rm -rf mongodb-linux-x86_64-rhel70-${MONGO_VERSION}
if [ ! -f mongodb-linux-x86_64-rhel70-${MONGO_VERSION}.tar.gz ];then
    wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-${MONGO_VERSION}.tgz
fi

# copy installation files to each server
for host in ${hosts[@]};do
    echo $host
    ssh -T $host mkdir -p ${RPM_PATH}

    scp -rp
    scp -rp mongodb-linux-x86_64-rhel70-${MONGO_VERSION}.tar.gz $host:${RPM_PATH}

    ssh -T $host << EOF
    cd ${RPM_PATH}
    tar -zxvf mongodb-linux-x86_64-rhel70-${MONGO_VERSION}.tar.gz
    mkdir -p ${MONGO_INSTALL_PATH}
    mv mongodb-linux-x86_64-rhel70-${MONGO_VERSION}/* ${MONGO_INSTALL_PATH}/
EOF
done

# install mongodb
for host in ${hosts[@]};do
  echo $host
  ssh -T $host << EOF
    bash install_mongo.sh shard 1
    bash install_mongo.sh shard 2
    bash install_mongo.sh shard 3
    bash install_mongo.sh config 1
EOF
done

# install mongodb
for host in ${hosts[@]};do
  echo $host
  ssh -T $host << EOF
    bash init_conf.sh 1
    bash init_conf.sh 2
    bash init_conf.sh 3
EOF
done
```

```bash
#!/bin/bash
#
###
# Filename: init.sh
# Author: mark.wu - jinqi.wu@midea.com
# Description: 初始化配置
# Last Modified: 2017-12-20 12:10:14
# Version: 1.0
###
MONGO_VERSION="3.6.1"
ROOT_PASSWD="root_Wms2018"
RPM_PATH="/apps/rpm"
MONGO_INSTALL_PATH="/apps/svr/mongodb"
MONGO_DATA_PATH="/apps/svr/mongodb/data"
MONGO_CONF_PATH="/apps/svr/mongodb/conf"
MONGO_LOG_PATH="/apps/svr/mongodb/logs"

hosts=(192.168.1.100 192.168.1.101 192.168.1.102)
host1=$hosts[0]
host2=$hosts[1]
host3=$hosts[2]

mongos_port=30000
config_port=27000
shard1_port=27001
shard2_port=27002
shard3_port=27003

confjs="${MONGO_CONF_PATH}/conf.js"
con_confjs=$confjs
configdb=config/$host1:$config_port,$host2:$config_port,$host3:$config_port
mongo_cmd="${MONGO_INSTALL_PATH}/bin/mongo"
```

```bash
#!/bin/bash
#
###
# Filename: init_mongo.sh
# Author: mark.wu - jinqi.wu@midea.com
# Description: 安装config、shred
# Last Modified: 2017-12-20 12:10:14
# Version: 1.0
###
source init.sh

mode=$1
instance=$2

key=""
ip_addr=$(ip addr | grep 'state UP' -A2 | tail -n1 | awk '{print $2}' | cut -f1 -d '/')

if [ -f "${MONGO_CONF_PATH}/keyfile" ]; then
  key=" --keyFile ${MONGO_CONF_PATH}/keyfile"
fi

if [[ "$mode" == "config" ]]; then
  mkdir -p "${MONGO_CONF_PATH}/config"
  mkdir -p "${MONGO_LOG_PATH}/config"
  mkdir -p "${MONGO_DATA_PATH}/config"
  tmp_conf = "${MONGO_CONF_PATH}/config/config.conf"
  cat > ${tmp_conf} <<EOF
  systemLog:
      quiet: false
      path: ${MONGO_LOG_PATH}/config/mongod.log
      logAppend: false
      destination: file
  processManagement:
      fork: false
      pidFilePath: ${MONGO_LOG_PATH}/config/mongod.pid
  net:
      bindIp: ${ip_addr}
      port: ${config_port}
      maxIncomingConnections: 65536
      wireObjectCheck: true
      ipv6: false
  storage:
      dbPath: ${MONGO_DATA_PATH}/config/db
      indexBuildRetry: true
      journal:
          enabled: true
          commitIntervalMs: 100
      directoryPerDB: true
      engine: wiredTiger
      syncPeriodSecs: 60
  operationProfiling:
      slowOpThresholdMs: 100
      mode: off
  sharding:
      clusterRole: configsvr
  replication:
     replSetName: config
EOF
fi

if [[ "$mode" == "shard" ]]; then
  mkdir -p  "${MONGO_CONF_PATH}/shard${instance}"
  mkdir -p  "${MONGO_LOG_PATH}/shard${instance}"
  mkdir -p  "${MONGO_DATA_PATH}/shard${instance}"
  tmp_conf = "${MONGO_CONF_PATH}/shard${instance}/shard${instance}.conf"
  eval port=\$shard${instance}_port
  cat > ${tmp_conf} <<EOF
  systemLog:
      quiet: false
      path: ${MONGO_LOG_PATH}/shard${instance}/mongod.log
      logAppend: true
      destination: file
  processManagement:
      fork: false
      pidFilePath: ${MONGO_LOG_PATH}/shard${instance}/mongod.pid
  net:
      bindIp: ${ip_addr}
      port: ${port}
      maxIncomingConnections: 65536
      wireObjectCheck: true
      ipv6: false
  storage:
      dbPath: ${MONGO_DATA_PATH}/shard${instance}/db
      #indexBuildRetry: true
      journal:
          enabled: true
          commitIntervalMs: 100
      directoryPerDB: true
      engine: wiredTiger
      syncPeriodSecs: 60
  operationProfiling:
      slowOpThresholdMs: 100
      mode: slowOp
  sharding:
      clusterRole: shardsvr
  replication:
     replSetName: shard$instance
EOF
fi

if [[ "$mode" == "mongos" ]]; then
  mkdir -p  "${MONGO_CONF_PATH}/mongos"
  mkdir -p  "${MONGO_LOG_PATH}/mongos"
  tmp_conf = "${MONGO_CONF_PATH}/mongos/shard${instance}.conf"
  eval port=\$shard${instance}_port
  cat > ${tmp_conf} <<EOF
  systemLog:
      quiet: false
      path: ${MONGO_LOG_PATH}/mongos/mongos.log
      logAppend: true
      destination: file
  processManagement:
      fork: false
      pidFilePath: ${MONGO_LOG_PATH}/mongos/mongos.pid
  net:
      bindIp: ${ip_addr}
      port: ${mongos_port}
      maxIncomingConnections: 65536
      wireObjectCheck: true
      ipv6: false
EOF
fi

if [[ "$mode" == "mongos" ]]; then
    echo 'start mongos...'
    echo "mongos --configdb $configdb -f $tmp_conf $key --fork"
    $mongo_cmd --configdb $configdb -f $tmp_conf $key --fork
else
    echo 'start $mode...'
    $mongo_cmd -f $tmp_conf $key --fork
fi
```

```bash
#!/bin/bash
#
###
# Filename: init_conf.sh
# Author: mark.wu - jinqi.wu@midea.com
# Description: 安装config、shred
# Last Modified: 2017-12-20 12:10:14
# Version: 1.0
###
source init.sh

serv_no=$1
echo $serv_no

if [[ $serv_no == "1" ]]; then
    echo "
    db = connect('${host1}:${shard1_port}/admin');
    db.getSiblingDB('admin');
    conf={
        _id: 'shard1',
        members: [
        {_id: 1, host: '${host1}:${shard1_port}',priority:30, arbiterOnly: false},
        {_id: 2, host: '${host2}:${shard1_port}',priority:20, arbiterOnly: false},
        {_id: 3, host: '${host3}:${shard1_port}',priority:10, arbiterOnly: true}
        ]
    };
    rs.initiate(conf);
    " > $confjs

    cat $confjs
    $mongo_cmd --nodb $con_confjs
    rm $confjs

    #set config replSets
    echo "
    db = connect('${host1}:${config_port}/admin');
    db.getSiblingDB('admin');
    conf={
        _id: 'config',
        members: [
        {_id: 1, host: '${host1}:${config_port}',priority:20, arbiterOnly: false},
        {_id: 2, host: '${host2}:${config_port}',priority:30, arbiterOnly: false},
        {_id: 3, host: '${host3}:${config_port}',priority:10, arbiterOnly: false}
        ]
    };
    rs.initiate(conf);
    " > $confjs

    cat $confjs
    $mongo_cmd --nodb $con_confjs
    rm $confjs
fi

if [[ $serv_no == "2" ]]; then
    echo "
    db = connect('${host2}:${shard2_port}/admin');
    db.getSiblingDB('admin');
    conf={
        _id: 'shard2',
        members: [
        {_id: 1, host: '${host2}:${shard2_port}',priority:30, arbiterOnly: false},
        {_id: 2, host: '${host1}:${shard2_port}',priority:10, arbiterOnly: true},
        {_id: 3, host: '${host3}:${shard2_port}',priority:20, arbiterOnly: false}
        ]
    };
    rs.initiate(conf);
    " > $confjs

    cat $confjs
    $mongo_cmd --nodb $con_confjs
    rm $confjs
fi

if [[ $serv_no == "3" ]]; then
    echo "
    db = connect('${host3}:${shard3_port}/admin');
    db.getSiblingDB('admin');
    conf={
        _id: 'shard3',
        members: [
        {_id: 1, host: '${host3}:${shard3_port}',priority:30, arbiterOnly: false},
        {_id: 2, host: '${host1}:${shard3_port}',priority:20, arbiterOnly: false},
        {_id: 3, host: '${host2}:${shard3_port}',priority:10, arbiterOnly: true}
        ]
    };
    rs.initiate(conf);
    " > $confjs

    cat $confjs
    $mongo_cmd --nodb $con_confjs
    rm $confjs
fi
```

```bash
#!/bin/bash
#
###
# Filename: init_mongos.sh
# Author: mark.wu - jinqi.wu@midea.com
# Description: 安装config、shred
# Last Modified: 2017-12-20 12:10:14
# Version: 1.0
###
source init.sh

mongos_host=127.0.0.1
echo "
db = connect('${mongos_host}:${mongos_port}/admin');
db.getSiblingDB('admin');
sh.addShard('shard1/${host1}:${shard1_port},${host2}:${shard1_port},${host3}:${shard1_port}')
sh.addShard('shard2/${host1}:${shard2_port},${host2}:${shard2_port},${host3}:${shard2_port}')
sh.addShard('shard3/${host1}:${shard3_port},${host2}:${shard3_port},${host3}:${shard3_port}')
db.printShardingStatus()
" > $confjs

cat $confjs
$mongo_cmd --nodb $con_confjs
rm $confjs
```

```bash
#!/bin/bash
#
###
# Filename: init_user.sh
# Author: mark.wu - jinqi.wu@midea.com
# Description: 安装config、shred
# Last Modified: 2017-12-20 12:10:14
# Version: 1.0
###
source init.sh

mongos_host=$1

echo "
db = connect('${mongos_host}:${mongos_port}/admin');
db.getSiblingDB('admin');
db.createUser( {
    user: 'root',
    pwd: '${ROOT_PASSWD}',
    roles: [ 'userAdminAnyDatabase','dbAdminAnyDatabase','readWriteAnyDatabase']
});
" > $confjs

cat $confjs
$mongo_cmd --nodb $con_confjs
rm $confjs
```

### 基础服务

#### Redis

- **安装**

```bash
#!/bin/bash
#
###
# Filename: install_redis.sh
# Author: mark.wu - jinqi.wu@midea.com
# Description:
# Last Modified: 2017-12-20 12:10:14
# Version: 1.0
###

REDIS_VERSION="3.2.10"
REDIS_INSTALL_PATH="/apps/svr/redis"

# install needed packages
yum install -y gcc gcc-c++ pcre zlib pcre-devel tcl >/dev/null 2>&1

# check tar file
if [ ! -e redis-${REDIS_VERSION}.tar.gz ];then
 echo "downloading redis..."
 wget -c http://download.redis.io/releases/redis-${REDIS_VERSION}.tar.gz
fi

# install
tar -zxvf redis-${REDIS_VERSION}.tar.gz
echo -n "install redis..."
make MALLOC=libc
make PREFIX=${REDIS_INSTALL_PATH} install >/dev/null 2>&1
if [ $? -eq 0 ];then
 make test >/dev/null 2>&1
 [ $? -eq 0 ] && echo "ok" || echo "failed"
else
 echo "instal redis-${REDIS_VERSION} failed..."
 exit 1
fi
```

#### RabbitMQ

- **安装**

```bash
#!/bin/bash
#
###
# Filename: install_rabbitmq.sh
# Author: mark.wu - jinqi.wu@midea.com
# Description: RabbitMQ安装脚本
# Last Modified: 2017-12-20 12:10:14
# Version: 1.0
###

ERLANG_VERSION="20.2"
RABBITMQ_VERSION="3.6.14"
ERLANG_INSTALL_PATH="/apps/svr/erlang"
RABBITMQ_INSTALL_PATH="/apps/svr/rabbitmq-server"

#install needed packages
yum install -y gcc gcc-c++ perl glibc-devel make m4 ncurses ncurses-devel openssl-devel zlib autoconf unixODBC unixODBC-devel

#check tar file
rm -rf otp-OTP-${ERLANG_VERSION}
rm -rf rabbitmq-server-${RABBITMQ_VERSION}
if [ ! -f OTP-${ERLANG_VERSION}.tar.gz ];then
    wget https://github.com/erlang/otp/archive/OTP-${ERLANG_VERSION}.tar.gz
fi
if [ ! -f rabbitmq-server-generic-unix-${RABBITMQ_VERSION}.tar.gz ];then
    wget https://github.com/rabbitmq/rabbitmq-server/releases/download/rabbitmq_v${RABBITMQ_VERSION}/rabbitmq-server-generic-unix-${RABBITMQ_VERSION}.tar.xz
fi

#install
if [ ! -d $ERLANG_INSTALL_PATH ];then
    mkdir -p $ERLANG_INSTALL_PATH
fi
if [ ! -d $RABBITMQ_INSTALL_PATH ];then
    mkdir -p $RABBITMQ_INSTALL_PATH
fi

tar -zxvf OTP-${ERLANG_VERSION}.tar.gz
cd otp-OTP-${ERLANG_VERSION}/
export ERL_TOP='${ERLANG_INSTALL_PATH}'
./otp_build autoconf
./configure --prefix=${ERLANG_INSTALL_PATH} --without-javac
make & make install

cd ../
xz -d rabbitmq-server-generic-unix-${RABBITMQ_VERSION}.tar.gz
tar -xvf rabbitmq-server-generic-unix-${RABBITMQ_VERSION}.tar
mv rabbitmq_server-${RABBITMQ_VERSION}/* ${RABBITMQ_INSTALL_PATH}/

cd ${RABBITMQ_INSTALL_PATH}/sbin
./rabbitmq-plugins enable rabbitmq_management

#启动
./rabbitmq-server -detached
./rabbitmqctl status
```

- **集群配置**

  - 修改 /etc/hosts

    ```bash
    #加入集群2个节点的描述
    192.168.1.1 rabbit01
    192.168.1.2 rabbit02
    ```

  - 设置 Erlang Cookie

    读取其中一个节点的cookie($HOME/.erlang.cookie), 并复制到其他节点

  - 重启各节点

    ```bash
    ./rabbitmqctl stop
    ./rabbitmq-server -detached
    ```

  - 建立集群

    以rabbit01为主节点，在rabbit02上执行
    ```bash
    ./rabbitmqctl cluster_status
    ./rabbitmqctl stop_app
    ./rabbitmqctl join_cluster rabbit@rabbit01
    ./rabbitmqctl start_app
    ```

  - 查看集群状态`./rabbitmqctl cluster_status`
  - 设置镜像队列策略

    ```bash
    ./rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'
    ```

### WMS服务

#### JDK

``` bash
#!/bin/bash
#
###
# Filename: install_jdk8.sh
# Author: mark.wu - jinqi.wu@midea.com
# Description:
# Last Modified: 2017-12-20 12:10:14
# Version: 1.0
###

# uninstall openjdk
rpm -qa | grep jdk | xargs rpm -e --nodeps
rpm -qa | grep java | xargs rpm -e --nodeps

# check jdk8
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u152-b16/aa0333dd3019491ca4f6ddbe78cdb6d0/jdk-8u152-linux-x64.tar.gz
tar -zxvf jdk-8u152-linux-x64.tar.gz -C /apps/svr/

#配置环境变量
echo "export JAVA_HOME=/apps/svr/jdk1.8.0_152" >> /etc/profile
echo -e "export JRE_HOME=\$JAVA_HOME/jre" >> /etc/profile
echo -e "export CLASSPATH=.:\$JAVA_HOME/lib:\$JRE_HOME/lib" >> /etc/profile
echo -e "export PATH=\$PATH:\$JAVA_HOME/bin" >> /etc/profile

source /etc/profile
java -version
```

#### Tomcat

- 下载、安装

```bash
wget http://mirrors.shuosc.org/apache/tomcat/tomcat-8/v8.0.48/bin/apache-tomcat-8.0.48.tar.gz
tar -zxvf apache-tomcat-8.0.48.tar.gz -C /apps/svr/
```

- 启动
  - 启动服务./tomcat.sh start
  - 停止服务./tomcat.sh stop
  - 重启服务./tomcat.sh restart

```
#!/bin/bash

#tomcat.sh

CATALINA_ADMIN="${BASH_SOURCE-$0}"
CATALINA_ADMIN="$(dirname "${CATALINA_ADMIN}")"
CATALINA_ADMIN_DIR="$(cd "${CATALINA_ADMIN}"; pwd)"

TOMCAT_HOME=$CATALINA_ADMIN_DIR/../
SHUTDOWN=$TOMCAT_HOME/bin/shutdown.sh
STARTUP=$TOMCAT_HOME/bin/startup.sh
CATALINA_PID=$TOMCAT_HOME/bin/CATALINA_PID

prog="tomcat"

start() {
    if [ -e $CATALINA_PID ];then
       echo "$prog already running...."
       exit 1
    fi

    echo -n $"Starting $prog..."
    $STARTUP
    echo
}

stop() {
    echo -n $"Stopping $prog..."
    $SHUTDOWN
    ps -ef | grep $TOMCAT_HOME | grep -v 'grep' | awk '{print $2}' | xargs kill -s 9
    rm -rf $TOMCAT_HOME/logs/*
    rm -rf $TOMCAT_HOME/work/*
    echo
}

case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    restart)
        stop
        sleep 5
        start
        ;;
    *)

    echo $"Usage: $prog {start|stop|restart}"
    exit 1
esac
```

- 优化
  - 添加相关变量, 修改`vi bin/catalina.sh`，108行开始添加如下内容

```bash
CATALINA_ADMIN="${BASH_SOURCE-$0}"
CATALINA_ADMIN="$(dirname "${CATALINA_ADMIN}")"
CATALINA_ADMIN_DIR="$(cd "${CATALINA_ADMIN}"; pwd)"

CATALINA_HOME=$CATALINA_ADMIN_DIR/../
CATALINA_PID=$CATALINA_HOME/bin/CATALINA_PID

#16G内存JVM配置
CATALINA_OPTS="-Dfile.encoding=UTF-8 -server -Xms8192m -Xmx8192m -XX:NewSize=2048m -XX:MaxNewSize=4096m -XX:MetaspaceSize=512m -XX:MaxMetaspaceSize=512m -XX:MaxTenuringThreshold=10 -XX:NewRatio=2 -XX:+DisableExplicitGC -Xloggc:$CATALINA_HOME/logs/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"

CATALINA_OPTS="-Dfile.encoding=UTF-8 -server -Xms4096m -Xmx4096m -XX:NewSize=1024m -XX:MaxNewSize=2048m -XX:MetaspaceSize=512m -XX:MaxMetaspaceSize=512m -XX:MaxTenuringThreshold=10 -XX:NewRatio=2 -XX:+DisableExplicitGC -Xloggc:$CATALINA_HOME/logs/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"
```

  - 如果使用 shutdown.sh 还无法停止 tomcat，可以修改其配置`vim bin/shutdown.sh`

```bash
#把最尾巴这一行
exec "$PRGDIR"/"$EXECUTABLE" stop "$@"
#改为
exec "$PRGDIR"/"$EXECUTABLE" stop 10 -force
```

  - 优化`vim conf/server.xml`

```xml
<?xml version='1.0' encoding='utf-8'?>
<Server port="8005" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

  <GlobalNamingResources>
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>

  <Service name="Catalina">
    <Executor name="tomcatThreadPool"
              namePrefix="catalina-exec-"
              maxThreads="500"
              minSpareThreads="100"
              maxIdleTime="60000"
              prestartminSpareThreads="true"
              maxQueueSize = "100"/>

    <Connector executor="tomcatThreadPool"
               port="9000"
               protocol="org.apache.coyote.http11.Http11Nio2Protocol"
               connectionTimeout="20000"
               maxConnections="10000"
               redirectPort="8443"
               enableLookups="false"
               acceptCount="100"
               maxPostSize="10485760"
               acceptorThreadCount="2"
               disableUploadTimeout="true"
               compression="on"
               compressionMinSize="2048"
               noCompressionUserAgents="gozilla, traviata"
               compressableMimeType="text/html,text/xml,text/plain,text/css,text/javascript,text/json,application/x-javascript,application/javascript,application/json"
               URIEncoding="UTF-8"/>

    <!-- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> -->

    <Engine name="Catalina" defaultHost="localhost">
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
    </Engine>
  </Service>
</Server>
```

#### 部署WMS

```bash
cd /apps/svr/wms
unzip wms.war
cd /apps/svr
mv apache-tomcat-8.0.48 wms-9000
rm -rf wms-9000/webapps/ROOT/*
mv /apps/svr/wms/wms/* /apps/svr/wms-9000/webapps/ROOT/
cd wms-9000/
./tomcat.sh start
```

#### 部署WMS RF

```bash
cd /apps/svr/wms
unzip wms-rf.war
cd /apps/svr
mv apache-tomcat-8.0.48 wms-rf-9000
rm -rf wms-rf-9000/webapps/*
mv /apps/svr/wms/wms-rf/* /apps/svr/wms-rf-9000/webapps/wms-rf/
cd wms-rf-9000/
./tomcat.sh start
```

#### 部署WMS REPORT

```bash
cd /apps/svr/wms
unzip wms-report.war
cd /apps/svr
mv apache-tomcat-8.0.48 wms-report-9000
rm -rf wms-report-9000/webapps/*
mv /apps/svr/wms/wms-report/* /apps/svr/wms-rf-9000/webapps/wms-report/
cd wms-report-9000/
./tomcat.sh start
```

#### 部署WMS JOB

```bash
cd /apps/svr/wms
unzip wms-job.war
cd /apps/svr
mv apache-tomcat-8.0.48 wms-job-9000
rm -rf wms-job-9000/webapps/ROOT/*
mv /apps/svr/wms/wms-job/* /apps/svr/wms-job-9000/webapps/ROOT/
cd wms-job-9000/
./tomcat.sh start
```

### WEB服务
![nginx](Nginx + Keepalived 高可用.png)

#### Nginx

- **安装**

install_nginx.sh
```bash
#!/bin/bash

#更新源
yum -y update

#安装依赖环境
yum install openssl openssl-devel libxml2-devel libxslt-devel perl-devel perl-ExtUtils-Embed -y

#定义版本及安装路径
NGINX_VERSION="1.12.2"
NGINX_INSTALL_PATH="/apps/svr/nginx"
NGINX_LOG_PATH="/apps/logs/nginx"
if [ ! -d $NGINX_INSTALL_PATH ];then
    mkdir -p $NGINX_INSTALL_PATH
fi
if [ ! -d $NGINX_LOG_PATH ];then
    mkdir -p $NGINX_LOG_PATH
fi

#下载
rm -rf nginx-${NGINX_VERSION}
if [ ! -f nginx-${NGINX_VERSION}.tar.gz ];then
  wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
fi
if [ ! -f pcre-8.41.tar.gz ];then
    wget -c https://ftp.pcre.org/pub/pcre/pcre-8.41.tar.gz && tar -zxf pcre-8.41.tar.gz
fi
if [ ! -f zlib-1.2.11.tar.gz ];then
    wget -c http://www.zlib.net/zlib-1.2.11.tar.gz && tar -zxf zlib-1.2.11.tar.gz
fi

#创建用户apps
groupadd -r apps && useradd -r -g apps -s /bin/false -M apps

#编译安装
tar -zxvf nginx-${NGINX_VERSION}.tar.gz
cd nginx-${NGINX_VERSION}
./configure --user=apps \
--group=apps \
--prefix=${NGINX_INSTALL_PATH} \
--with-http_stub_status_module \
--without-http-cache \
--with-http_ssl_module \
--with-http_v2_module \
--with-http_gzip_static_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_realip_module \
--with-http_addition_module \
--with-http_xslt_module \
--with-http_stub_status_module \
--with-http_sub_module \
--with-http_random_index_module \
--with-http_degradation_module \
--with-http_secure_link_module \
--with-http_gzip_static_module \
--with-http_perl_module \
--with-pcre=../pcre-8.36 \
--with-zlib=../zlib-1.2.8 \
--with-debug \
--with-file-aio \
--with-mail \
--with-mail_ssl_module \
--with-stream \
--with-ld-opt="-Wl,-E"
CPU_NUM=$(cat /proc/cpuinfo | grep processor | wc -l)
if [ $CPU_NUM -gt 1 ];then
    make -j$CPU_NUM
else
    make
fi
make install

#添加系统服务
mkdir -p ${NGINX_LOG_PATH}/access/
chmod 775 ${NGINX_LOG_PATH}
cat > /etc/init.d/nginx <<EOF
#!/bin/bash

nginxd=${NGINX_INSTALL_PATH}/sbin/nginx
nginx_config=${NGINX_INSTALL_PATH}/conf/nginx.conf
nginx_pid=/apps/run/nginx.pid

RETVAL=0
prog="nginx"

[ -x \$nginxd ] || exit 0
# Start nginx daemons functions.
start() {
    if [ -e \$nginx_pid ] && netstat -tunpl | grep nginx &> /dev/null;then
        echo "nginx already running..."
        exit 1
    fi
    echo -n \$"Starting \$prog..."
    \$nginxd -c \${nginx_config}
    RETVAL=\$?
    echo
    [ \$RETVAL = 0 ] && touch /var/lock/subsys/$prog
    return \$RETVAL
}
# Stop nginx daemons functions.
stop() {
    echo -n \$"Stopping \$prog..."
    \$nginxd -s stop
    RETVAL=\$?
    echo
    [ \$RETVAL = 0 ] && rm -f /var/lock/subsys/$prog
}
# reload nginx service functions.
reload() {
    echo -n $"Reloading \$prog..."
    #kill -HUP \`cat \${nginx_pid}\`
    \$nginxd -s reload
    RETVAL=\$?
    echo
}
# See how we were called.
case "\$1" in
start)
        start
        ;;
stop)
        stop
        ;;
reload)
        reload
        ;;
restart)
        stop
        start
        ;;
*)
        echo $"Usage: \$prog {start|stop|restart|reload|help}"
        exit 1
esac
exit \$RETVAL
EOF
chmod 755 ${NGINX_INSTALL_PATH}/sbin/nginx
chmod +x /etc/init.d/nginx
mkdir -p ${NGINX_INSTALL_PATH}/conf/vhosts/
```

- **部署静态资源**

```bash
cd /apps/svr/wms
unzip wms-static.war
mkdir -p /apps/svr/nginx/root/release
mv /apps/svr/wms/wms-static/* /apps/svr/nginx/root/release/
```

- **配置**

nginx.conf
```nginx
user  apps apps;
worker_processes auto;
worker_rlimit_nofile 8192;

events {
    use epoll;
    worker_connections  8192;
}

#error_log  logs/error.log;
error_log  /apps/logs/nginx/error.log  notice;
#error_log  logs/error.log  info;

pid        /apps/run/nginx.pid;

http {
    include       mime.types;
    default_type  application/octet-stream;

    include       gzip.conf;
    include       proxy.conf;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /apps/logs/nginx/access.log  main;

    sendfile        on;
    tcp_nopush      on;
    keepalive_timeout  65;

    include       gzip.conf;
    include       vhosts/*.conf;
}
```

gzip.conf
```nginx
  gzip               on;
  gzip_buffers       8 16k;
  gzip_disable       msie6;
  gzip_comp_level    5;
  gzip_min_length    1024;
  gzip_proxied       any;
  gzip_vary          on;
  gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;
```
proxy.conf
```nginx
proxy_redirect             off;
proxy_set_header           Host $host;
proxy_set_header           X-Real-IP $remote_addr;
proxy_set_header           X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header 	         X-Forwarded-Proto $scheme;
proxy_set_header           Accept-Encoding 'gzip';
client_max_body_size       100m;
client_body_buffer_size    256k;
proxy_connect_timeout      500;
proxy_send_timeout         2000;
proxy_read_timeout         2000;
proxy_ignore_client_abort  on;

proxy_buffer_size          128k;
proxy_buffers              4 256k;
proxy_busy_buffers_size    256k;
proxy_temp_file_write_size 256k;
```

wms.conf
```nginx
upstream wms.cloud.com {
	server 127.0.0.1:9000;
}

upstream report.wms.cloud.com {
	server 127.0.0.1:9001;
}

server {
	listen       80;
	server_name  localhost;
	root    /apps/svr/nginx/root/release;

	access_log   /apps/logs/nginx/wms.host.access.log  main;

	location   / {
		proxy_pass http://wms.cloud.com;
		proxy_set_header Host                $host;
		proxy_set_header X-Real-IP           $remote_addr;
		proxy_set_header X-Forwarded-For     $proxy_add_x_forwarded_for;
	}
	location   /wms-report {
		proxy_pass http://report.wms.cloud.com/wms-report;
		proxy_set_header Host                $host;
		proxy_set_header X-Real-IP           $remote_addr;
		proxy_set_header X-Forwarded-For     $proxy_add_x_forwarded_for;
	}
	location ~ /images/prod-logo.png {
		proxy_pass http://wms.cloud.com;
	}
	location ~ /js/wms/ {
		proxy_pass http://wms.cloud.com;
	}
	location ~ /css/wms/ {
		proxy_pass http://wms.cloud.com;
	}
	location ~ /api/mongo/file/ {
		proxy_pass http://wms.cloud.com;
	}
	location ~ .*\.(gif|jpg|ico|jpeg|png|bmp|swf|js|json|css|woff|woff2|ttf|html|coffee|map|mp3|wav)$
	{
		expires      1d;
	}
	location ~ ^/(WEB-INF)/ {
		deny all;
	}
	error_page   500 502 503 504  /50x.html;
	location = /50x.html {
	  root   html;
	}
}
```

- **启动**
  - 启动服务：`service nginx start`
  - 停止服务：`service nginx stop`
  - 重载服务：`service nginx reload`
  - 重启服务：`service nginx restart`

#### Keepalived

- 安装

keepalived-install.sh
```bash
#!/bin/bash

#更新源
yum -y update

#安装依赖环境
yum install -y gcc openssl-devel popt-devel libnl libnl-devel libnfnetlink libnfnetlink-devel

#定义版本和路径
KEEPALIVED_VERSION="1.3.9"
KEEPALIVED_INSTALL_PATH="/apps/svr/keepalived"
KEEPALIVED_CONF_PATH="/apps/svr/keepalived/conf"
KEEPALIVED_LOG_PATH="/apps/logs/keepalived"
if [ ! -d $KEEPALIVED_INSTALL_PATH ];then
    mkdir -p $KEEPALIVED_INSTALL_PATH
fi
if [ ! -d $KEEPALIVED_CONF_PATH ];then
    mkdir -p $KEEPALIVED_CONF_PATH
fi
if [ ! -d $KEEPALIVED_LOG_PATH ];then
    mkdir -p $KEEPALIVED_LOG_PATH
fi

#下载
rm -rf keepalived-${KEEPALIVED_VERSION}
if [ ! -f keepalived-${KEEPALIVED_VERSION}.tar.gz ];then
  wget https://github.com/acassen/keepalived/archive/v${KEEPALIVED_VERSION}.tar.gz
fi

#编译安装
tar zxvf keepalived-${KEEPALIVED_VERSION}.tar.gz
cd keepalived-${KEEPALIVED_VERSION}
./configure --prefix=${KEEPALIVED_INSTALL_PATH}
make && make install

#添加系统服务
cat > /etc/init.d/keepalived <<EOF
#!/bin/sh
#
# Startup script for the Keepalived daemon
#

keepalived=${KEEPALIVED_INSTALL_PATH}/sbin/keepalived
keepalived_config=${KEEPALIVED_INSTALL_PATH}/conf/keepalived.conf
keepalived_pid=/apps/run/keepalived.pid

RETVAL=0
prog="keepalived"

start() {
    echo -n $"Starting $prog: "
    daemon $keepalived -f $keepalived_config -p $keepalived_pid -D -S 0
    RETVAL=$?
    echo
    [ $RETVAL -eq 0 ] && touch /var/lock/subsys/$prog
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $keepalived
    RETVAL=$?
    echo
    [ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/$prog
}

reload() {
    echo -n $"Reloading $prog: "
    killproc $keepalived -1
    RETVAL=$?
    echo
}

# See how we were called.
case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    reload)
        reload
        ;;
    restart)
        stop
        start
        ;;
    condrestart)
        if [ -f /var/lock/subsys/$prog ]; then
            stop
            start
        fi
        ;;
    status)
        status $prog
        RETVAL=$?
        ;;
    *)
        echo "Usage: $0 {start|stop|reload|restart|condrestart|status}"
        RETVAL=1
esac

exit $RETVAL
EOF
chmod 755 ${KEEPALIVED_INSTALL_PATH}/sbin/keepalived
chmod +x /etc/init.d/keepalived
```

- 配置

重定向日志路径`vi /etc/syslog.conf`，添加下面内容，重启日志服务`systemctl restart rsyslog`
```
# keepalived -S 0
local0.* /apps/log/keepalived/keepalived.log
```

Nginx健康监测脚本 : /apps/svr/keepalived/sh/nginx_check.sh
添加执行权限：chmod 755 /apps/svr/keepalived/sh/nginx_check.sh
nginx_check.sh
```bash
#!/bin/bash
# 如果进程中没有nginx则将keepalived进程kill掉
A=`ps -C nginx --no-header |wc -l`      ## 查看是否有 nginx进程 把值赋给变量A
if [ $A -eq 0 ];then                    ## 如果没有进程值得为 零
       service keepalived stop          ## 则结束 keepalived 进程
fi
```

keepalived.conf
```nginx
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

#nginx监控实现
vrrp_script check_nginx {
 	# 运行脚本
 	script "/apps/svr/keepalived/sh/nginx_check.sh"
 	# 时间间隔，2秒
 	interval 2
 	# 权重
 	weight 2
 }

vrrp_instance VI_1 {
    # Backup 机子设置为：BACKUP
    state MASTER
    interface eth0
    virtual_router_id 51
    # Backup 机子要小于当前 Master 设置的 100，建议设置为 99
    priority 100
    # Master 与 Backup 负载均衡器之间同步检查的时间间隔，单位是秒
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # （重点）配置虚拟 IP 地址，如果有多个则一行一个
    virtual_ipaddress {
        192.168.1.100
    }
    track_script {
   	    check_nginx
   	}
}
```

- 启动
  - 启动服务：`service keepalived start`
  - 停止服务：`service keepalived stop`
  - 重载服务：`service keepalived reload`
  - 重启服务：`service keepalived restart`

## 安全性

## 备份和恢复

mydumper+binlog
