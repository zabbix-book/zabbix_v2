> 《Zabbix企业级分布式监控系统第2版》随书代码
>
> 代码仓库地址 https://github.com/zabbix-book/zabbix_v2
>
> 书籍购买地址 https://item.jd.com/12653708.html

#573页

表15-1　硬件配置

| 应 用 名 称   | 操 作 系 统  | CPU    | 内　　存  | 硬　　盘           |
| ------------- | ------------ | ------ | --------- | ------------------ |
| Zabbix-Server | RHEL 6.5 x64 | 8核心  | DDR3 16GB | SATA 500GB´2 RAID1 |
| Zabbix-DB     | RHEL 6.5 x64 | 24核心 | DDR3 64GB | SAS 300GB´4 RAD1+0 |
| Zabbix-GUI    | RHEL 6.5 x64 | 4核心  | DDR3 4GB  | SATA 500GB         |
| Zabbix-Proxy  | RHEL 6.5 x64 | 8核心  | DDR3 16GB | SAS 300GB´2 RAID1  |

表15-2　各服务器的IP地址规划

| 应 用 名 称   | 角　　色        | IP地址        |             |            |
| ------------- | --------------- | ------------- | ----------- | ---------- |
| 物理IP地址    | 虚拟IP地址      | 公网IP地址    |             |            |
| Zabbix-Server | 主服务器        | 192.168.0.3   | 192.168.0.5 | 58.75.2.89 |
| Zabbix-Server | 从服务器        | 192.168.0.4   |             |            |
| Zabbix-DB     | MySQL数据库主库 | 192.168.0.240 | 无          | 无         |
| Zabbix-DB     | MySQL数据库从库 | 192.168.0.241 |             |            |
| Zabbix-GUI    | 前端管理界面    | 192.168.0.2   | 无          | 无         |
| Zabbix-Proxy  | 代理节点        | 10.10.10.2    | 无          | 61.61.52.9 |
| Zabbix-Agent  | 被监控端        | 10.10.10.10   | 无          | 无         |

# 574页

表15-3　Zabbix的数据库规划

| 角　　色     | IP地址        | 域　　名                        | 运行的服务 |
| ------------ | ------------- | ------------------------------- | ---------- |
| MySQL-Master | 192.168.0.240 | zabbix-mysql-master.itnihao.com | MySQL服务  |
| MySQL-Slave  | 192.168.0.241 | zabbix-mysql-slave.itnihao.com  | MySQL服务  |

```shell
shell# rpm -ivh http://repo.mysql.com/yum/mysql-8.0-community/el/7/x86_64/ mysql80-community-release-el7-1.noarch.rpm
Shell# yum install mysql-community-server
```

```
shell# vim  /etc/my.cnf 
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql

symbolic-links=0
character-set-server=utf8  #设置字符集为UTF-8
innodb_file_per_table=1    #让InnoDB的每个表文件单独存储
innodb_data_file_path=ibdata1:10M:autoextend
server_id=1 #从库设置为非1

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

```

更多配置可参考https://github.com/zabbix-book/MySQL_conf

# 575页

```shell
shell# systemctl  start mysqld
```

```shell
shell# MysqlPassword=admin@2018ABCD
shell# mysqladmin -u root password  ${MysqlPassword} 
mysql> create database zabbix character set utf8;
mysql> grant all privileges on zabbix.* to zabbix@'192.168.0.2' identified  by  'zabbix';
mysql> grant all privileges on zabbix.* to zabbix@'192.168.0.3' identified  by  'zabbix';
mysql> grant all privileges on zabbix.* to zabbix@'192.168.0.4' identified  by  'zabbix';
mysql> grant all privileges on zabbix.* to zabbix@'192.168.0.5' identified  by  'zabbix';
mysql> grant all privileges on zabbix.* to zabbix@'127.0.0.1' identified  by  'zabbix';
mysql> flush privileges;
```

```shell
shell# cd /usr/share/doc/zabbix-server-mysql-4.0.0 	#进入对应版本的目录
shell# gunzip  create.sql.gz                       		#将SQL文件解压缩
shell# mysql -uzabbix -pzabbix -h127.0.0.1         	#以zabbix用户登录
mysql> use zabbix                                   		#切换到zabbix库
mysql> source  /usr/share/doc/zabbix-server-mysql-4.0.0/create.sql; 
```

```shell
shell# systemctl stop mysqld
```

# 576页

```shell
shell# rsync -av -e "ssh -p 22" /var/lib/mysql/ root@192.168.0.241:/var/ lib/mysql #192.168.0.241为MySQL数据库从库服务器的IP地址
```

```shell
shell# systemctl start mysqld
```

```bash
shell# mysql -uroot -p
mysql> GRANT REPLICATION CLIENT,REPLICATION SLAVE ON *.* TO repl@'192.168.0.241.%'  IDENTIFIED BY  'zabbix_repl'; 
mysql> FLUSH PRIVILEGES;
```

```python
mysql> show master status\G;
*************************** 1. row ***************************
             File: mysql-bin.000003
         Position: 107
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
```

```shell
shell# systemctl start mysqld #启动Master节点
```

```shell
mysql> change master to master_host='192.168.0.240', MASTER_USER='repl', MASTER_PASSWORD='zabbix_repl', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=107, MASTER_CONNECT_RETRY=10;    #107要和mater的Position数值一致
```

# 577页

```shell
mysql> start slave;
```

```python
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.240
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 10
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 888190
               Relay_Log_File: mysql-relay-bin.000002
                Relay_Log_Pos: 107
        Relay_Master_Log_File: mysql-bin.000003
查看如下显示是否均为YES状态
             Slave_IO_Running: Yes
            Slave_SQL_Running: YES
```

```shell
shell# egrep -v "^$|^#" /etc/zabbix/zabbix_server.conf     
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/var/run/zabbix/zabbix_server.pid
DBHost = zabbix-mysql-master.itnihao.com
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
DBSocket=/var/lib/mysql/mysql.sock
SNMPTrapperFile=/var/log/snmptt/snmptt.log
AlertScriptsPath=/etc/zabbix/alertscripts
ExternalScripts=/etc/zabbix/externalscripts
SourceIP=192.168.0.5  #VIP地址
```

# 578页

```shell
shell# yum install keepalived -y
```

**/etc/keepalived/keepalived.conf** 配置参考

https://github.com/zabbix-book/zabbix-HA/blob/master/keepalived.conf-master

```shell
# cat keepalived.conf 
# https://github.com/zabbix-book/zabbix-HA
! Configuration File for keepalived

global_defs {
   notification_email {
        admin@localhost.com
   }
   notification_email_from zabbix_ha_node1@itnihao.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ZABBIX_NODE1
}

vrrp_script zabbix_ha {
       script "/etc/keepalived/ha_switch.sh 1 2 STATUS"
       interval 5
}

vrrp_instance VI_1 {
    state MASTER
    nopreempt
    interface eth0
    virtual_router_id 109
    priority 190
    advert_int 5
    smtp_alert

    authentication {
        auth_type PASS
        auth_pass ZABBIX_HA_ZBX_BOOK
    }

    virtual_ipaddress {
        192.168.0.5
    }

    track_script {
        zabbix_ha
    }

    notify_master /etc/keepalived/ha_switch.sh MASTER
    notify_backup /etc/keepalived/ha_switch.sh BACKUP
    notify_fault /etc/keepalived/ha_switch.sh FAULT
    notify /etc/keepalived/ha_switch.sh
}
```

https://github.com/zabbix-book/zabbix-HA/blob/master/keepalived.conf-backup

```shell

# cat keepalived.conf 
# https://github.com/zabbix-book/zabbix-HA
! Configuration File for keepalived

global_defs {
   notification_email {
        admin@localhost.com
   }
   notification_email_from zabbix_ha_node2@itnihao.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id ZABBIX_NODE2
}

vrrp_script zabbix_ha {
       script "/etc/keepalived/ha_switch.sh 1 2 STATUS"
       interval 5
}

vrrp_instance VI_1 {
    state BACKUP
    nopreempt
    interface eth0
    virtual_router_id 109
    priority 100
    advert_int 5
    smtp_alert

    authentication {
        auth_type PASS
        auth_pass ZABBIX_HA_ZBX_BOOK
    }

    virtual_ipaddress {
        192.168.0.5
    }

    track_script {
        zabbix_ha
    }

    notify_master /etc/keepalived/ha_switch.sh MASTER
    notify_backup /etc/keepalived/ha_switch.sh BACKUP
    notify_fault /etc/keepalived/ha_switch.sh FAULT
    notify /etc/keepalived/ha_switch.sh
}
```

# 579页

书中的脚本处理并不是最佳实践，可使用此处更新后的脚本

```shell
# cat ha_switch.sh 
#!/bin/bash
#https://github.com/zabbix-book/zabbix-HA
#author: itnihao

STATE="$3"

ulimit -n 40960
VIP="192.168.0.5"
ZBX_SERVER="zabbix_server"
ZBX_SERVER_PID="/var/run/zabbix/zabbix_server.pid"
#SERVER_PORCESS_NUM=$(pidstat -C "zabbix_server"|grep -c "$ZBX_SERVER")

if [ -f "${ZBX_SERVER_PID}" ];then
    SERVER_PID=$(cat "${ZBX_SERVER_PID}")
else
    SERVER_PID=$(pidof "${ZBX_SERVER}")
fi

if [ "${SERVER_PID}" == "" ];then
     SERVER_PID=$(pidof "${ZBX_SERVER}")
fi

case $STATE in
    "MASTER")
        if [ "${SERVER_PID}" == ""  ];then
            systemctl start zabbix-server
        fi
        echo "MASTER" >/etc/zabbix/.ha_role
        exit 0
        ;;
    "BACKUP")
        systemctl stop zabbix-server
        killall -9 zabbix_server
        echo "BACKUP" >/etc/zabbix/.ha_role
        arping "${VIP}" -c 2
        exit 0
        ;;
    "FAULT")
        systemctl stop zabbix-server
        killall -9 zabbix_server
        exit 0
        ;;
    "STATUS")
         #echo "$(date) status">>/tmp/date.log
         ROLE=$(cat /etc/zabbix/.ha_role)
         if [ "${SERVER_PID}" == ""  ];then
            if [ "$ROLE" == "MASTER" ];then
                killall -9 "${ZBX_SERVER}" && rm -f "${ZBX_SERVER_PID}" 
                systemctl start zabbix-server
                exit 0
            elif [ "$ROLE" == "BACKUP" ];then
                ps -ef |grep "/usr/sbin/zabbix_server"|grep -v "grep"|awk '{print $2}'|xargs kill -9 && rm  -f "${ZBX_SERVER_PID}" 
                systemctl stop zabbix-server
                ps -ef |grep "/usr/sbin/zabbix_server"|grep -v "grep"|awk '{print $2}'|xargs kill -9
                if [ -f "${ZBX_SERVER_PID}" ];then
                    rm  -f "${ZBX_SERVER_PID}" 
                fi
                exit 0
            else
                exit 0
            fi
         fi
         ;;
    *)
         echo "unknown state"
         exit 1
         ;;
esac
```

# 582页

```shell
{Template OS Linux:proc.num[].avg(5m)}>300 #进程数量
{Template OS Linux:agent.ping.nodata(5m)}=1 #由于网络抖动而引起的误报
{Template OS Linux:kernel.maxfiles.last(0)}<1024 #文件描述符，实际大于此参数值
{Template OS Linux:kernel.maxproc.last(0)}<256 #进程数，实际大于此参数值
```

