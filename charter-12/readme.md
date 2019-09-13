> 《Zabbix企业级分布式监控系统第2版》随书代码
>
> 代码仓库地址 https://github.com/zabbix-book/zabbix_v2
>
> 书籍购买地址 https://item.jd.com/12653708.html

# 514页

```php
//代码位于src/frontends/php/include/func.inc.php中
//performance [指性能数据吗？]  //从Zabbix-Server中获取NVPS的数据
if (array_key_exists('required performance', $server_status)) {
    $status['vps_total'] = 0;
    foreach ($server_status['required performance'] as $stats) {
        if ($stats['attributes']['proxyid'] == 0) {
            $status['vps_total'] += $stats['count'];
        }
    }
}
```

```c
//代码位于src/libs/zbxdbcache/dbconfig.c的9675~9688行
Double DCget_required_performance(void)
{
    double    nvps;
    WRLOCK_CACHE;
    dc_status_update();
    nvps = config->status->required_performance;
    UNLOCK_CACHE;
    return nvps;
}
//以及位于src/libs/zbxdbcache/dbconfig.c的9481~9486行
{
    config->status->required_performance += 1.0 / delay;
    if (NULL != dc_proxy)
        dc_proxy->required_performance += 1.0 / delay;
}
```

```sql
mysql> SELECT SUM(1.0/i.delay) AS qps FROM items i,hosts h  WHERE i.status= 'ITEM_STATUS_ACTIVE' AND i.hostid=h.hostid  AND h.status='HOST_STATUS_MONITORED';
+----------+
| qps       |
+----------+
| 53.24389 |
```

# 515页

```shell
StartPollers=160
StartPollersUnreacheable=80
StartTrappers=20
StartPingers=100
StartDiscoverers=120
CacheSize=1024M
StartDBSyncers=16
HistoryCacheSize=1024M
TrendCacheSize=1024M
HistoryTextCacheSize=512M
```

```shell
### Option: StartPreprocessors
#       Number of pre-forked instances of preprocessing workers.
#       The preprocessing manager process is automatically started when preprocessor worker is started.
#
# Mandatory: no
# Range: 1-1000
# Default:
# StartPreprocessors=3
```

```shell
shell# systemctl restart zabbix-server
```

https://www.zabbix.org/wiki/How_to/configure_shared_memory

# 518页

```shell
shell# mkdir -p /etc/zabbix/modules
shell# mkdir -p /usr/src/zabbix 
shell# cd /usr/src/zabbix
shell# wget https://sourceforge.net/projects/zabbix/files/ZABBIX%20Latest% 20Stable/4.0.0/zabbix-4.0.0.tar.gz
shell# tar xvf zabbix-4.0.0.tar.gz
shell# cd zabbix-4.0.0
shell# ./configure --enable-agent
shell# mkdir src/modules/zabbix_module_stress
shell# cd src/modules/zabbix_module_stress
shell# wget https://raw.githubusercontent.com/monitoringartist/zabbix-server-stress-test/master/src/modules/zabbix_module_stress/zabbix_module_ stress.c
shell# wget https://raw.githubusercontent.com/monitoringartist/zabbix-server-stress-test/master/src/modules/zabbix_module_stress/Makefile
shell# make
shell# ls -l 
-rw-r--r-- 1 root root   132 10月  9 14:07 Makefile
-rw-r--r-- 1 root root  9107 10月  9 14:06 zabbix_module_stress.c
-rwxr-xr-x 1 root root 12960 10月  9 14:07 zabbix_module_stress.so
shell# cp  zabbix_module_stress.so /etc/zabbix/modules
```

```shell
shell# vim /etc/zabbix/zabbix_agentd.conf
### Option: LoadModulePath
#       Full path to location of agent modules.
#       Default depends on compilation options.
#
# Mandatory: no
# Default:
# LoadModulePath=${libdir}/modules
LoadModulePath=/etc/zabbix/modules  #配置模块加载的路径

### Option: LoadModule
#       Module to load at agent startup. Modules are used to extend functionality of the agent.
#       Format: LoadModule=<module.so>
#       The modules must be located in directory specified by LoadModulePath.
#       It is allowed to include multiple LoadModule parameters.
#
# Mandatory: no
# Default:
# LoadModule=
LoadModule=zabbix_module_stress.so  #加载压力测试模块
```

```shell
shell# systemctl  restart zabbix-agent
shell# zabbix_get  -s 127.0.0.1 -k stress.ping[1]
1
```

# 526页

```shell
shell# cat /etc/zabbix/zabbix_server.conf
############ GENERAL PARAMETERS #################
ListenPort=10051 #监控端口，默认值为10051，范围是1024~32767.源码文件src/zabbix_server/server.c中硬编码实现，可修改代码调整。
SourceIP= #当Zabbix-Server有两个IP地址时，需要定义选择IP地址作为出口，如果使用了VIP，则需要配置SourceIP为VIP的地址
LogFile=/var/log/zabbix/zabbix_server.log #日志文件的保存路径
LogFileSize=0  #每个日志文件最大的容量。当设置为0时，即关闭日志轮转自动切割的功能。单位是MB
# DebugLevel=3 #默认值为3，共有6个级别，从0到5
#       0 — 基本信息，Zabbix进程的启停信息
#       1 — 严重致命的软件运行错误信息
#       2 — 错误信息
#       3 — 警告信息
#       4 — 调试信息，输出大量的日志
#       5 — 开发调试，输出大量的详细日志
PidFile=/var/run/zabbix/zabbix_server.pid #Pid文件路径
SocketDir=/var/run/zabbix #IPC Socket的文件夹路径
DBHost=localhost #DB的主机名，或者IP地址
DBName=zabbix #DB的名称
# DBSchema=  #仅在数据库IBM DB2和PostgreSQL中使用
DBUser=zabbix #DB的用户名
DBPassword=zabbix #DB的密码
DBSocket= #DB的Socket路径，一般不需要设置，如DBHost为localhost，则需要配置
DBPort= #DB的端口
# HistoryStorageURL= #当使用Elasticsearch时配置
# HistoryStorageTypes=uint,dbl,str,log,text #将哪些数据类型存储到Elasticsearch上
# HistoryStorageDateIndex=0 #是否开启按日期索引功能，0表示关闭，1表示开启
# ExportDir= #将实时监控数据输出到JSON文件，此处填写文件夹的路径
# ExportFileSize=1G #导出的JSON文件单个最大值
############ ADVANCED PARAMETERS ################
# StartPollers=5 #被动模式进程开启的个数，通常一个进程可以获取10~20台主机的监控数据，最大值为1000，最小值为0，默认值为5，可以在硬件配置足够好的情况下，一次将此值调整为1000
# StartIPMIPollers=0 #IPMI进程开启的个数，默认值为0，表示关闭IPMI，取值范围为0~1000
# StartPreprocessors=3 #预处理进程的个数，默认值为3，需要根据实际情况调整，建议设置为100左右，取值范围为0~1000
# StartPollersUnreachable=1 #当主机不可达时，会再次请求，使用此进程，建议设置为主机数量的3%，即100台主机设置为3。取值范围为0~1000
# StartTrappers=5 #Trapper进程的数量，使用zabbix_sender命令发送的数据，将由此进程进行处理。默认值为5，如果采用Trapper进程的数量较多，则应增大此值，取值范围为0~1000
# StartPingers=1 #ICMP ping的进程个数，默认值为1，取值范围为0~1000
# StartDiscoverers=1 #自动发现的进程个数，取值范围为0~250
# StartHTTPPollers=1 #HTTP监控进程的个数，默认值为1，取值范围为0~1000
# StartTimers=1 #与时间有关的触发器函数和维护时间的进程，取值范围为1~1000
# StartEscalators=1 #告警升级的进程个数，取值范围为0~100
# StartAlerters=3 #告警的进程个数，取值范围为0~100
# JavaGateway=127.0.0.1 #JavaGateway的IP地址，仅支持一个参数
# JavaGatewayPort=10052 #JavaGateway的端口
# StartJavaPollers=1 #JavaGateway的处理进程个数，取值范围为0~1000
# StartVMwareCollectors=0 #VMware监控的进程个数，取值范围为0~200
# VMwareFrequency=60 #Zabbix-Server每隔多久从VMware获取一次数据，取值范围为10~86400
# VMwarePerfFrequency=60 #Zabbix-Server每隔多久从VMware获取一次性能数据，取值范围为10~86400
# VMwareCacheSize=8M #存储VMware监控数据的内存大小，取值范围为256KB~2GB
# VMwareTimeout=10 #连接到VMware服务的超时时间，取值范围为1~300
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log #SNMP Trapper的日志文件路径
# StartSNMPTrapper=0 #是否开启SNMP Trapper，1表示开启，0表示关闭
# ListenIP=127.0.0.1 #监听的IP地址，如果为空，则所有网卡都被允许接收用户端的数据[不通顺]
# HousekeepingFrequency=1 #删除数据的频率，单位为小时，取值范围为0~24
# MaxHousekeeperDelete=5000 #每次最大删除数据条数，0表示删除全部，取值范围
为0~1 000 000
# CacheSize=8M #配置缓存的大小，用于存储Host、Item和Trigger等的数据，这里一般不用设置得太大，一般环境下200MB足够，取值范围为128KB~8GB
# CacheUpdateFrequency=60  #缓存更新的频率，单位为秒，取值范围为1~3600
# StartDBSyncers=4 #数据库同步进程，也叫历史数据同步进程，取值范围为1~100。功能是将数据从缓存同步到数据库，负责触发器的计算，处理趋势数据，更新Item状态，存储告警事件结果。1个DBSyncer可以支持1000个NVPS，4个DBSyncer可以支持4000的NVPS，[请顺一下这句话]前提是数据库能够及时同步。注意，这个值不能设置得太大，太大会造成数据库压力过大，而导致整体处理速度下降
# HistoryCacheSize=16M #历史数据的缓存大小，取值范围为128KB~2GB，可以一次设置到最大值2G
# HistoryIndexCacheSize=4M #历史数据缓存的索引大小，取值范围为128KB~2GB。一个Item需要分配100B的空间，2000万个的Item需要分配大约2GB的内存空间，计算方式为：(100.00B/ 1024KB/ 1024MB)*2000 0000=1907MB。如果读者的Item为100万个，则（100.00/1024/1024）*100 0000[对吗？]=[正确] 95MB。因此这个值可以不用计算，直接将其设置为256MB，基本可以满足200~300万个Item数据的存储需求。默认值为4MB，只能支撑4万个Item。如果此值过小，则会导致zabbix_server进程退出运行
# TrendCacheSize=4M #趋势数据的缓存大小，取值范围为128KB~2GB，和历史数据的缓存大小一样，可以一次设置到最大值2GB
# ValueCacheSize=8M #Item的历史数据，取值范围为128KB~64GB，0表示关闭。这个值可以根据实际情况而定，但建议将其设置得大些，在2GB以上。一般可以设置存取监控指标一天的数据容量，比如数据库一天更新量有5GB，则可以将此值设置为5GB。此值设置得越大，Zabbix的性能越好。如按照时间范围查询监控指标数据，将优先从内存取值，如果内存中无此时间范围的数据，再从数据库查询获取，这样可以有效减少数据库的访问
# 当缓存利用率较高时，会采取删除策略进行删除释放——删除最后一天无访问的数据、删除命中率低的数据，不再添加新的数据到缓存中
# 当内存容量小于设置的数值时，将每5分钟写入一次警告信息到日志
# Timeout=3 #超时时间，对Agent、SNMP设置和扩展检测有效，取值范围为1~30s
# TrapperTimeout=300 #处理Trapper数据的超时时间[不通顺]，取值范围为1~300s
# UnreachablePeriod=45 #多久检测一次不可达主机，取值范围为1~3600s
# UnavailableDelay=60 #当主机状态不可用时，检测可达的频率，取值范围为1~3600s
# UnreachableDelay=15 #当主机不可达时，检测可达的频率，取值范围为1~3600s
AlertScriptsPath=/usr/lib/zabbix/alertscripts #告警脚本路径
ExternalScripts=/usr/lib/zabbix/externalscripts #扩展脚本路径
# FpingLocation=/usr/sbin/fping #fping二进制文件的路径
# Fping6Location=/usr/sbin/fping6 #fping6二进制文件的路径
# SSHKeyLocation= #SSH密钥的路径
LogSlowQueries=3000 #慢查询的时间，取值范围为1~3 600 000ns
# TmpDir=/tmp #临时文件的路径
# StartProxyPollers=1 #Proxy进程的个数，用于被动模式下，取值范围为0~250
# ProxyConfigFrequency=3600 #将Zabbix-Server端的配置数据同步到Proxy端的周期[不通顺]，取值范围为1~3600*24*7
# ProxyDataFrequency=1 #多久从Zabbix-Proxy获取一次监控的历史数据，取值范围为1~3600s
# AllowRoot=0 #是否使用root启动，0表示不允许，1表示使用root启动进程
# User=zabbix #Zabbix进程运行的用户
# 子配置文件路径
# Include=/usr/local/etc/zabbix_server.general.conf
# Include=/usr/local/etc/zabbix_server.conf.d/
# Include=/usr/local/etc/zabbix_server.conf.d/*.conf
# SSLCertLocation=${datadir}/zabbix/ssl/certs #SSL证书路径
# SSLKeyLocation=${datadir}/zabbix/ssl/keys  #SSL证书key路径
# SSLCALocation= #SSL证书CA路径

####### LOADABLE MODULES #######
# LoadModulePath=${libdir}/modules  #动态模块路径
# LoadModule=  #需要加载的动态模块 module.so，如果有多个，则写多行
......后续的证书认证配置省略......
#需要注意，以上配置中，凡是增大进程数量的配置，都会增加数据库的连接数，如数据库的默认连接数过小，则会造成Zabbix-Server启动的问题。
```

