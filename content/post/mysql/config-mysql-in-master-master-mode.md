---
title: "基于MySQL配置数据库主主同步"
date: 2024-12-16T13:17:29+08:00
lastmod: 2024-12-16T13:17:29+08:00
draft: false
keywords: ["Docker","MySQL","主主同步"]
description: "基于个人实际使用经验，简单记录在Docker环境下对MySQL配置主主同步的操作过程"
tags: ["mysql","docker"]
categories: ["数据库"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false
centerImage: false
borderImage: true
codeTabSeperator: "::"
moreMeta: true

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

主要参考[这篇文章](https://cloud.tencent.com/developer/article/2254761)中的相关说明，记录如何在`MySQL`中实现`主主同步`，即两个数据库可同时操作然后同时同步数据给对方。

<!--more-->

## 前提条件

|        |                       数据库A                        |                           数据库B                           |
| :----: | :--------------------------------------------------: | :---------------------------------------------------------: |
|  端口  |                         3370                         |                            3380                             |
| 数据库 |                      vde_test_1                      |                         vde_test_1                          |
|   表   |                       table_1                        |                           table_1                           |
| 表数据 | INSERT INTO table_1(name) VALUES('a1'),('a2'),('a3') | INSERT INTO table_1(name) VALUES('b1'),('b2'),('b3'),('b4') |

## 相关文件

{{% codetabs %}}  

```ini::my.cnf配置文件
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
init_connect="SET collation_connection = utf8mb4_unicode_ci"
init_connect="SET NAMES utf8mb4"
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
```

```yaml::docker-compose-db1.yml
version: '3'
services:
   mysql_test_1:
      image: mysql:5.7.35
      restart: always 
      container_name: mysql_vde_sync_test_1 
      environment:
        MYSQL_ROOT_PASSWORD: 123456 
        TZ: Asia/Shanghai
      ports:
        - 3370:3306 
      volumes:
        - $PWD/data:/var/lib/mysql
        - $PWD/conf/my.cnf:/etc/my.cnf
```

```yaml::docker-compose-db2.yml
version: '3'
services:
   mysql_test_2:
      image: mysql:5.7.35
      restart: always 
      container_name: mysql_vde_sync_test_2 
      environment:
        MYSQL_ROOT_PASSWORD: 123456 
        TZ: Asia/Shanghai
      ports:
        - 3380:3306 
      volumes:
        - $PWD/data:/var/lib/mysql
        - $PWD/conf/my.cnf:/etc/my.cnf
```

{{% /codetabs %}}

## 环境配置

1.共使用2个数据库进行测试验证，其位置与配置如下

  ![数据库位置和路径](/blog_img/mysql/config-mysql-in-master-master-mode/database-location-and-file-list.png "数据库位置和路径") 

2.在当前路径下执行下述指令启动数据库并验证

```bash
# 数据库1
cd mysql_db_1 && docker-compose -f docker-compose-db1.yml up -d

# 数据库2
cd ../mysql_db_2 && docker-compose -f docker-compose-db2.yml up -d && cd ..

# 查看启动结果
docker ps|grep mysql_vde_sync_test_
```

执行结果如下，可看出两个数据库均正常启动，接下来可进行测试

  ![数据库启动结果](/blog_img/mysql/config-mysql-in-master-master-mode/start-docker-dbs-and-check.png "数据库启动结果") 

可基于下述指令来关闭服务与重置测试环境

```bash
# 停止docker服务
cd mysql_db_1 && docker-compose -f docker-compose-db1.yml down
cd ../mysql_db_2 && docker-compose -f docker-compose-db2.yml down

# 删除旧数据
cd ..
rm -rf mysql_db_1/data && rm -rf mysql_db_2/data
rm -rf mysql_db_1/conf/my.cnf && rm -rf mysql_db_2/conf/my.cnf

# 重新创建配置文件
read -r -d '' sql_config << EOF
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
init_connect="SET collation_connection = utf8mb4_unicode_ci"
init_connect="SET NAMES utf8mb4"
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
EOF

echo "$sql_config" >> $PWD/mysql_db_1/conf/my.cnf \
&& echo "$sql_config" >> $PWD/mysql_db_2/conf/my.cnf
```

## 同步配置

1.执行下述指令，以便允许远程连接

```bash
# 在每个数据库中要执行的指令
sql_command="GRANT ALL PRIVILEGES ON *.* to 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;\
FLUSH PRIVILEGES;\
SHOW GRANTS FOR root;\G
CREATE DATABASE IF NOT EXISTS vde_test_1;\
SHOW DATABASES;\
SHOW VARIABLES WHERE Variable_name = 'hostname';"

# 分别进入2个mysql数据库并执行上述指令
printf "=====================mysql_vde_sync_test_1运行结果================\n" \
&& docker exec -it mysql_vde_sync_test_1 bash -c "mysql -uroot -p123456 -e \"${sql_command}\"" \
&& printf "\n\n=====================mysql_vde_sync_test_2运行结果================\n" \
&& docker exec -it mysql_vde_sync_test_2 bash -c "mysql -uroot -p123456 -e \"${sql_command}\"";
```

2.在两个数据库的`my.cnf`中分别添加如下代码

{{% codetabs %}}  

```ini::第1个my.cnf
server-id=1
log-bin=mysql-bin
binlog_format=mixed
binlog-do-db=vde_test_1
replicate-do-db=vde_test_1
auto_increment_increment=2
auto_increment_offset=1
```

```ini::第2个my.cnf
server-id=2
log-bin=mysql-bin
binlog_format=mixed
binlog-do-db=vde_test_1
replicate-do-db=vde_test_1
auto_increment_increment=2
auto_increment_offset=2
```

{{% /codetabs %}}

相关说明如下：

- `binlog-do-db`与`replicate-do-db`，要同步的库，若有多个，就多写几行
- `auto_increment_offset`，设置自增起始值。
- `auto_increment_increment`，主键自增的步长，用于防止Master与Master之间出现主键冲突(重复)，通常有多少台主服务器，设置为多少

相关操作指令如下

{{% codetabs %}} 

```bash::数据库1配置更新
cat >> $PWD/mysql_db_1/conf/my.cnf<< EOF
server-id=1
log-bin=mysql-bin
binlog_format=mixed
binlog-do-db=vde_test_1
replicate-do-db=vde_test_1
auto_increment_increment=2
auto_increment_offset=1
EOF
docker restart mysql_vde_sync_test_1
```

 ```bash::数据库2配置更新
cat >> $PWD/mysql_db_2/conf/my.cnf<< EOF
server-id=2
log-bin=mysql-bin
binlog_format=mixed
binlog-do-db=vde_test_1
replicate-do-db=vde_test_1
auto_increment_increment=2
auto_increment_offset=2
EOF
docker restart mysql_vde_sync_test_2
 ```

{{% /codetabs %}}

3.分别执行下述指令，查看其偏移量

{{% codetabs %}}  

```bash::数据库1写入数据
show_master_status="show master status;"
# 数据库mysql_vde_sync_test_1创建表并增加数据
read -r -d '' insert_data_1 << EOF
USE vde_test_1;
DROP TABLE IF EXISTS table_1;
CREATE TABLE IF NOT EXISTS table_1 (
  id int(10) unsigned NOT NULL AUTO_INCREMENT,
  name varchar(20) COLLATE utf8_bin NOT NULL,
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
INSERT INTO table_1(name) VALUES('a1'),('a2'),('a3');
SELECT * FROM \`\`table_1\`\`;
EOF

docker exec -it mysql_vde_sync_test_1 bash -c "mysql -uroot -p123456 -e\"${insert_data_1};${show_master_status}\"";
```

```bash::数据库1写入数据
show_master_status="show master status;"
# 数据库mysql_vde_sync_test_2创建表并增加数据
read -r -d '' insert_data_2 << EOF
USE vde_test_1;
DROP TABLE IF EXISTS table_1;
CREATE TABLE IF NOT EXISTS table_1 (
  id int(10) unsigned NOT NULL AUTO_INCREMENT,
  name varchar(20) COLLATE utf8_bin NOT NULL,
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
INSERT INTO table_1(name) VALUES('b1'),('b2'),('b3'),('b4');
SELECT * FROM \`\`table_1\`\`;
EOF

docker exec -it mysql_vde_sync_test_2 bash -c "mysql -uroot -p123456 -e\"${insert_data_2};${show_master_status}\"";
```

{{% /codetabs %}}

执行结果分别类似如下

  ![数据库1 master status查看](/blog_img/mysql/config-mysql-in-master-master-mode/db-1-master-status.png "数据库1 master status查看") 
  ![数据库2 master status查看](/blog_img/mysql/config-mysql-in-master-master-mode/db-2-master-status.png "数据库2 master status查看") 

从结果中可以获取如下信息

- 数据库`mysql_vde_sync_test_1`其`File`值为`mysql-bin.000001`,`Position`值为`1091`
- 数据库`mysql_vde_sync_test_2`其`File`值为`mysql-bin.000001`,`Position`值为`1098`

4.对2个数据库分别执行如下操作(即需要反着配置，互相同步对方的数据)

{{% codetabs %}}  

```bash::数据库1操作
read -r -d '' sync_datbase_1 << EOF
stop slave;

CHANGE MASTER TO \
MASTER_HOST='aeectss.hirain.local',\
MASTER_PORT=3380,\
MASTER_USER='root',\
MASTER_PASSWORD='123456',\
MASTER_LOG_FILE='mysql-bin.000001',\
MASTER_LOG_POS=1098;

start slave;
EOF

# 数据库1同步的数据为数据库2
docker exec -it mysql_vde_sync_test_1 bash -c "mysql -uroot -p123456 -e\"${sync_datbase_1}\"";
```

```bash::数据库2操作
read -r -d '' sync_datbase_2 << EOF
stop slave;

CHANGE MASTER TO \
MASTER_HOST='aeectss.hirain.local',\
MASTER_PORT=3370,\
MASTER_USER='root',\
MASTER_PASSWORD='123456',\
MASTER_LOG_FILE='mysql-bin.000001',\
MASTER_LOG_POS=1091;

start slave;
EOF

# 数据库2同步的数据为数据库1
docker exec -it mysql_vde_sync_test_2 bash -c "mysql -uroot -p123456 -e\"${sync_datbase_2}\"";
```

{{% /codetabs %}}

若一切正常，执行结果类似如下

  ![数据库slave同步修改](/blog_img/mysql/config-mysql-in-master-master-mode/db-slave-change-result.png "数据库slave同步修改") 

5.执行下述指令重启数据库并验证数据库`slave`状态

```bash
docker restart mysql_vde_sync_test_1 mysql_vde_sync_test_2

# 分别查看其状态
docker exec -it mysql_vde_sync_test_1 bash -c "mysql -uroot -p123456 -e\"show slave status\G\"";
docker exec -it mysql_vde_sync_test_2 bash -c "mysql -uroot -p123456 -e\"show slave status\G\"";
```

若结果类似如下，则表示数据库主主同步配置正常

```properties
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

  ![数据库主从同步正常](/blog_img/mysql/config-mysql-in-master-master-mode/mysql-slave-satus-check.png "数据库主从同步正常") 

## 同步验证

### 原始数据

分别登录这两个数据库，切换到对应的`vde_test_1`数据库后查看`table_1`表中的数据，结果如下。

可看出**最开始的数据并没有在两个数据库之间同步**,原因为设置`MASTER_LOG_POS`时是从最新的数据开始的，之前的数据不会同步。

若想同步之前的数据，则需要重新设置对应的`MASTER_LOG_POS`值，此种场景一般不常见，通常建议**在最开始同步时两个数据库要保持一致**。

  ![数据库初始状态验证](/blog_img/mysql/config-mysql-in-master-master-mode/master-dbs-init-check.png "数据库初始状态验证") 

### 新增数据

两个数据库互相新增数据，然后验证

  ![数据库新增数据验证](/blog_img/mysql/config-mysql-in-master-master-mode/master-dbs-insert-check.png "数据库新增数据验证") 

### 修改数据

两个数据库互相修改数据，然后验证
  ![数据库修改数据验证](/blog_img/mysql/config-mysql-in-master-master-mode/master-dbs-update-check.png "数据库修改数据验证") 

### 新增表

数据库1中新增表，之后会自动同步到数据库2中去

  ![数据库新增表验证](/blog_img/mysql/config-mysql-in-master-master-mode/master-dbs-create-table-check.png "数据库新增表验证") 

### 修改表

数据库2修改`table_2`表结构，数据库1也能自动同步。

  ![数据库修改表验证](/blog_img/mysql/config-mysql-in-master-master-mode/master-dbs-alter-table-check.png "数据库修改表验证") 

## 参考指令

{{% codetabs %}}  

```bash::数据库操作
# 直接登录数据库
docker exec -it mysql_vde_sync_test_1 bash -c "mysql -uroot -p123456 vde_test_1"
docker exec -it mysql_vde_sync_test_2 bash -c "mysql -uroot -p123456 vde_test_1"

# 进入数据库后的相关操作
insert into table_1(`name`) values('a4');
insert into table_1(`name`) values('b5');
update table_1 set name='A4' where id=7;
update table_1 set name='B5' where id=10;
select * from table_1;

DROP TABLE IF EXISTS table_2;
CREATE TABLE IF NOT EXISTS table_2 (
  id int(10) unsigned NOT NULL AUTO_INCREMENT,
  name varchar(20) COLLATE utf8_bin NOT NULL,
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
INSERT INTO table_2(name) VALUES('data_1'),('data_2'),('data_3');

select * from table_2l
show tables;
alter table table_2 add column description varchar(100);
desc table_2;
```

```bash::binlog日志操作
# 查看日志，必须加上--no-defaults否则会报错
mysqlbinlog --no-defaults  mysql-bin.000002

# 从指定位置查看日志
mysqlbinlog --no-defaults --start-position=1080  mysql-bin.000002
```

```bash::一键配置
#!/bin/bash

# 将前面的步骤整合到一个shell脚本中一键运行配置环境，节省时间

# 停止docker服务
cd mysql_db_1 && docker-compose -f docker-compose-db1.yml down
cd ../mysql_db_2 && docker-compose -f docker-compose-db2.yml down

# 删除旧数据
cd ..
rm -rf mysql_db_1/data && rm -rf mysql_db_2/data
rm -rf mysql_db_1/conf/my.cnf && rm -rf mysql_db_2/conf/my.cnf

# 重新创建配置文件
read -r -d '' sql_config << EOF
[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4

[mysqld]
init_connect="SET collation_connection = utf8mb4_unicode_ci"
init_connect="SET NAMES utf8mb4"
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
EOF

echo "$sql_config" >> $PWD/mysql_db_1/conf/my.cnf \
&& echo "$sql_config" >> $PWD/mysql_db_2/conf/my.cnf


# 数据库1
cd mysql_db_1 && docker-compose -f docker-compose-db1.yml up -d

# 数据库2
cd ../mysql_db_2 && docker-compose -f docker-compose-db2.yml up -d && cd ..

# 查看启动结果
#docker ps|grep mysql_vde_sync_test_

sleep 20

# 在每个数据库中要执行的指令
sql_command="GRANT ALL PRIVILEGES ON *.* to 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;\
FLUSH PRIVILEGES;\
SHOW GRANTS FOR root;\G
CREATE DATABASE IF NOT EXISTS vde_test_1;\
SHOW DATABASES;\
SHOW VARIABLES WHERE Variable_name = 'hostname';"

# 分别进入2个mysql数据库并执行上述指令
printf "=====================mysql_vde_sync_test_1运行结果================\n" \
&& docker exec -it mysql_vde_sync_test_1 bash -c "mysql -uroot -p123456 -e \"${sql_command}\"" \
&& printf "\n\n=====================mysql_vde_sync_test_2运行结果================\n" \
&& docker exec -it mysql_vde_sync_test_2 bash -c "mysql -uroot -p123456 -e \"${sql_command}\"";


cat >> $PWD/mysql_db_1/conf/my.cnf<< EOF
server-id=1
log-bin=mysql-bin
binlog_format=mixed
binlog-do-db=vde_test_1
replicate-do-db=vde_test_1
auto_increment_increment=2
auto_increment_offset=1
EOF
docker restart mysql_vde_sync_test_1

cat >> $PWD/mysql_db_2/conf/my.cnf<< EOF
server-id=2
log-bin=mysql-bin
binlog_format=mixed
binlog-do-db=vde_test_1
replicate-do-db=vde_test_1
auto_increment_increment=2
auto_increment_offset=2
EOF
docker restart mysql_vde_sync_test_2

sleep 20

show_master_status="show master status;"
# 数据库mysql_vde_sync_test_1创建表并增加数据
read -r -d '' insert_data_1 << EOF
USE vde_test_1;
DROP TABLE IF EXISTS table_1;
CREATE TABLE IF NOT EXISTS table_1 (
  id int(10) unsigned NOT NULL AUTO_INCREMENT,
  name varchar(20) COLLATE utf8_bin NOT NULL,
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
INSERT INTO table_1(name) VALUES('a1'),('a2'),('a3');
SELECT * FROM \`\`table_1\`\`;
EOF

docker exec -it mysql_vde_sync_test_1 bash -c "mysql -uroot -p123456 -e\"${insert_data_1};${show_master_status}\"";

show_master_status="show master status;"
# 数据库mysql_vde_sync_test_2创建表并增加数据
read -r -d '' insert_data_2 << EOF
USE vde_test_1;
DROP TABLE IF EXISTS table_1;
CREATE TABLE IF NOT EXISTS table_1 (
  id int(10) unsigned NOT NULL AUTO_INCREMENT,
  name varchar(20) COLLATE utf8_bin NOT NULL,
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
INSERT INTO table_1(name) VALUES('b1'),('b2'),('b3'),('b4');
SELECT * FROM \`\`table_1\`\`;
EOF

docker exec -it mysql_vde_sync_test_2 bash -c "mysql -uroot -p123456 -e\"${insert_data_2};${show_master_status}\"";

read -r -d '' sync_datbase_1 << EOF
stop slave;

CHANGE MASTER TO \
MASTER_HOST='aeectss.hirain.local',\
MASTER_PORT=3380,\
MASTER_USER='root',\
MASTER_PASSWORD='123456',\
MASTER_LOG_FILE='mysql-bin.000001',\
MASTER_LOG_POS=1098;

start slave;
EOF

# 数据库1同步的数据为数据库2
docker exec -it mysql_vde_sync_test_1 bash -c "mysql -uroot -p123456 -e\"${sync_datbase_1}\"";

read -r -d '' sync_datbase_2 << EOF
stop slave;

CHANGE MASTER TO \
MASTER_HOST='aeectss.hirain.local',\
MASTER_PORT=3370,\
MASTER_USER='root',\
MASTER_PASSWORD='123456',\
MASTER_LOG_FILE='mysql-bin.000001',\
MASTER_LOG_POS=1091;

start slave;
EOF

# 数据库2同步的数据为数据库1
docker exec -it mysql_vde_sync_test_2 bash -c "mysql -uroot -p123456 -e\"${sync_datbase_2}\"";

docker restart mysql_vde_sync_test_1 mysql_vde_sync_test_2

sleep 40

docker exec -it mysql_vde_sync_test_1 bash -c "mysql -uroot -p123456 -e\"show slave status\G\"";
docker exec -it mysql_vde_sync_test_2 bash -c "mysql -uroot -p123456 -e\"show slave status\G\"";
```

{{% /codetabs %}}