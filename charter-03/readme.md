> 《Zabbix企业级分布式监控系统第2版》随书代码
>
> 代码仓库地址 https://github.com/zabbix-book/zabbix_v2
>
> 书籍购买地址 https://item.jd.com/12653708.html



# 30页

```shell
6．时间同步需求
Zabbix-Server对时间的精准要求比较高，时间对数据的计算等都有影响，因此必须设置NTP自动同步时间。

shell# yum install ntp -y
shell# systemctl  enable ntpd
shell# systemctl  start ntpd

当然，也可以使用crontab进行同步，但在实际的生产环境中不推荐定时任务的同步，而是推荐上面的NTP同步方式。如下所示，使用crontab进行时间同步。

*/30 * * * *    /usr/sbin/ntpdate  pool.ntp.org
```



# 31页

```shell
安装Zabbix的RPM包软件仓库官方源，如图3-5所示。

#CentOS 7，4.X版本的安装
shell# rpm -ivh http://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm

#CentOS 7，3.X版本的安装
shell# rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm
注意：由于软件版本的更新，如果以上链接失效，读者可在http://repo.zabbix.com/ zabbix/4.0/rhel/7/x86_64/中找到软件包。
```

# 33-34页

```shell
安装Zabbix-Server服务器端，由于服务器端也是需要监控的，故这里也一并安装了Zabbix-Agent。

#此处是4.0的安装命令，3.0的安装命令与之相同
shell# yum install zabbix-server-mysql zabbix-web-mysql zabbix-agent zabbix-get -y
#在Zabbix-Server服务器上安装Zabbix-Agent，是为了通过Agent方式监控Zabbix-Sever服务器本身的运行情况

此过程安装的软件包如下：

Installing  :  libzip-0.10.1-8.el7.x86_64                          1/37
Installing  :  php-common-5.4.16-45.el7.x86_64                     2/37
Installing  :  php-bcmath-5.4.16-45.el7.x86_64                     3/37
……
Installing  :  php-pdo-5.4.16-45.el7.x86_64                        8/37
Installing  :  php-mysql-5.4.16-45.el7.x86_64                      9/37
……
Installing  :  httpd-2.4.6-80.el7.centos.1.x86_64                 28/37
Installing  :  php-5.4.16-45.el7.x86_64                           29/37
……
Installing  :  zabbix-web-mysql-4.0.0-1.1rc3.el7.noarch           32/37
Installing  :  zabbix-web-4.0.0-1.1rc3.el7.noarch                 33/37
Installing  :  fping-3.10-1.el7.x86_64                            34/37
Installing  :  zabbix-server-mysql-4.0.0-1.1rc3.el7.x86_64        35/37
Installing  :  zabbix-agent-4.0.0-1.1rc3.el7.x86_64               36/37
Installing  :  zabbix-get-4.0.0-1.1rc3.el7.x86_64                 37/37

3.2.2　安装MySQL
在CentOS 7系统包仓库安装源中，我们需要安装mariadb-server，而不是MySQL数据库服务（7.0以后版本用MariaDB替换了MySQL），命令如下：

shell# yum  -y install  mariadb-server

所需的依赖包如下：

Installing : 1:mariadb-5.5.56-2.el7.x86_64                    1/10 
Installing : libaio-0.3.109-13.el7.x86_64                     2/10 
Installing : 1:perl-Compress-Raw-Zlib-2.061-4.el7.x86_64      3/10 
Installing : perl-Net-Daemon-0.48-5.el7.noarch                4/10 
Installing : perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64       5/10 
Installing : perl-IO-Compress-2.061-2.el7.noarch              6/10 
Installing : perl-PlRPC-0.2020-14.el7.noarch                  7/10 
Installing : perl-DBI-1.627-4.el7.x86_64                      8/10 
Installing : perl-DBD-MySQL-4.023-6.el7.x86_64                9/10 
Installing : 1:mariadb-server-5.5.56-2.el7.x86_64            10/10 

修改MySQL配置文件如下（粗体字部分很重要），使用以下命令：

shell# vi  /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
character-set-server=utf8  #设置字符集为UTF-8
innodb_file_per_table=1    #让InnoDB的每个表文件单独存储

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
# 需要注意的是，以上MySQL配置参数仅满足小规模监控环境使用，适合初次接触MySQL的读者调整。
# 如监控环境为中型规模(如1000设备以上)，需要调整更多的MySQL配置参数，可参考https://github.com/zabbix-book/MySQL_conf中提供的参数。
# 如监控环境为大型规模（如8000设备以上），需要使用MySQL集群相关技术，如读写分离，数据库集群等。建议熟悉MySQL集群技术的读者采用。

启动服务，使用以下命令：

shell# systemctl  start mariadb    #启动服务

设置开机自启动，使用以下命令：

shell# systemctl  enable mariadb   #设置开机自启动
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb. service to /usr/lib/systemd/system/mariadb.service.
```



# 35页

```shell
1．创建Zabbix数据库
首先设置MySQL的root用户密码，然后创建zabbix数据库，设置访问策略，使用以下命令：

shell# mysqladmin -uroot password admin #设置root用户密码为admin
shell# mysql -uroot -padmin               #登录MySQL
#创建数据库，名称为zabbix，并将其字符集设置为UTF8
mysql> create  database  zabbix  character  set  utf8;
#设置zabbix数据库的所有权限，允许用户zabbix的IP地址127.0.0.1和localhost访问，并将zabbix账号的密码设置为zabbix
mysql> grant all privileges  on zabbix.*  to  zabbix@'localhost'  identified  by  'zabbix'; 
mysql> grant all privileges  on zabbix.*  to  zabbix@'127.0.0.1'  identified  by  'zabbix'; 
#刷新权限，使其立即生效
mysql> flush  privileges;
```

```shell
导入Zabbix库的数据文件，使用以下命令：

shell# cd /usr/share/doc/zabbix-server-mysql-4.0.0    #进入对应版本目录
shell# gunzip  create.sql.gz                          #将SQL文件解压缩
shell# mysql -uzabbix -pzabbix -h127.0.0.1            #以zabbix用户登录
mysql> use zabbix                                     #切换到zabbix数据库
mysql> source  /usr/share/doc/zabbix-server-mysql-4.0.0/create.sql;
                                                      #导入SQL文件
#上面的4.0.0代表实际的版本，读者拿到本书后，其版本会发生变化，因此可以通过命令ls /usr/ share/doc|grep zabbix 来查看实际路径
#create.sql是zabbix源码包中的3个SQL文件的合集，即分别为schame.sql、images.sql和data.sql，其中schema.sql是表结构；images.sql是图片相关数据； data.sql是模板等相关数据
# 如果是以源码方式安装Zabbix-Server的，则需要将这3个文件全部导入
# 如果是以源码方式安装Zabbix-Porxy的，则只能导入schames.sql

Zabbix-Proxy的安装和配置是后续章节的内容，这里仅列出SQL导入命令，以便加深印象。

shell# cd /usr/share/doc/zabbix-proxy-mysql-4.0.0  #进入对应版本目录
shell# gunzip  schema.sql.gz                       #将SQL文件解压缩
mysql> use zabbix-porxy                            #切换到zabbix-proxy库
mysql> source /usr/share/doc/zabbix-proxy-mysql-4.0.0/schema.sql;
#导入SQL文件
```

# 36页

```shell
配置zabbix_server.conf文件如下：
1．默认参数

shell# egrep  -v  "(^#|^$)"  /etc/zabbix/zabbix_server.conf
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/var/run/zabbix/zabbix_server.pid
SocketDir=/var/run/zabbix
DBName=zabbix
DBUser=zabbix
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
Timeout=4
AlertScriptsPath=/usr/lib/zabbix/alertscripts
ExternalScripts=/usr/lib/zabbix/externalscripts
LogSlowQueries=3000
2．修改后的参数（可参考）

shell# egrep -v "(^#|^$)" /etc/zabbix/zabbix_server.conf 
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/var/run/zabbix/zabbix_server.pid
DBHost=127.0.0.1                  	 #需要修改
DBName=zabbix                   		 #默认
DBUser=zabbix                    		 #默认
DBPassword=zabbix                		 #需要修改
DBSocket=/var/lib/mysql/mysql.sock	 #默认
DBPort=3306                         	 #默认
StartPollers=100                  	 #约5台服务器开一个进程，最大1000
StartIPMIPollers=10               	 #视IPMI监控主机个数而定
StartPollersUnreachable=10        	 #不可达主机重试获取数据进程个数
StartTrappers=10                    	 #Trapper进程个数
StartPingers=10                    	 #Ping进程个数
StartDiscoverers=10               	 #自动发现进程个数
SNMPTrapperFile=/var/log/snmptt/snmptt.log
MaxHousekeeperDelete=500  
CacheSize=256M                     	 #可根据实际情况修改
HistoryCacheSize=128M               	 #可根据实际情况修改
TrendCacheSize=128M                 	 #可根据实际情况修改
HistoryTextCacheSize=128M          	 #可根据实际情况修改
ValueCacheSize=2048M               	 #可根据实际情况修改
Timeout=30                       		 #此处需要修改，最大执行时间长(30秒以内)
TrapperTimeout=300
UnreachablePeriod=45
UnavailableDelay=60
UnreachableDelay=15
AlertScriptsPath=/etc/zabbix/alertscripts
ExternalScripts=/etc/zabbix/externalscripts
FpingLocation=/usr/sbin/fping
LogSlowQueries=10000
StartProxyPollers=50
ProxyConfigFrequency=3600

以上参数只需关注粗体字部分，这部分为性能参数，需要根据实际情况进行调整。
```



# 37页

```shell
shell# mkdir -p /etc/zabbix/alertscripts /etc/zabbix/externalscripts
```



```shell
3．开启Zabbix-Server服务
启动Zabbix-Server相关服务，使用以下命令：

#CentOS7
shell# systemctl start zabbix-server
shell# systemctl start httpd
#查看进程
shell# ps -ef |grep zabbix
#查看日志
shell# tail -f /var/log/zabbix/zabbix_server.log
#如遇到此提示connection to database 'zabbix' failed: [1045] Access denied for user 'zabbix'@'localhost' (using password: NO)，请检查/etc/zabbix/zabbix_ server.conf配置文件数据库连接相关信息是否配置正确

添加开机启动项，使用以下命令：

shell# systemctl enable zabbix-server 
Created symlink from /etc/systemd/system/multi-user.target.wants/zabbix-server.service to /usr/lib/systemd/system/zabbix-server.service.
shell# systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```



# 38页

```shell
1．防火墙的设置
如CentOS操作系统存在防火墙，则需允许相关端口能够访问，配置命令如下：

#CentOS 6操作系统防火墙规则设置
shell# iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
shell# iptables -A IN_public_allow -p tcp -m tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT 
shell# iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 10051 -j ACCEPT
shell# iptables -A OUTPUT -m state --state NEW -m tcp -p tcp --dport 10050 -j ACCEPT
shell# iptables -A INPUT -m state --state NEW -m tcp -p tcp --sport 10050 -j ACCEPT
#规则设置完成后，均需永久保存，用如下语句
shell# iptables-save  >/tmp/ip.txt
shell# iptables-restore </tmp/ip.txt

#CentOS 7操作系统防火墙规则设置
shell# firewall-cmd --permanent --add-port=80/tcp
shell# firewall-cmd --permanent --add-port=10050/tcp
shell# firewall-cmd --permanent --add-port=10051/tcp
shell# firewall-cmd –reload
上述规则中，10050是Agent的端口，Agent采用被动方式，Server主动连接Agent的10050端口；10051是Server的端口，Agent采用主动或Trapper方式，会连接Server的10051端口。对于具有防火墙的网络环境，采取相同的防火墙规则策略即可。
2．SELinux的设置
如果操作系统已开启SELinux，则需要设置语句开启允许SELinux相关策略。

shell# setsebool -P httpd_can_connect_zabbix on
shell# setsebool -P httpd_can_network_connect_db on
```

# 39页

```shell
shell# setenforce 0 	#设置为警告模式，只给出提示，不会阻止操作，不用重启服务器即生效
shell# getenforce 	#获取当前SELinux的运行状态
  【Enforcing|Permissive|Disabled】
直接关闭SELinux的方法：

shell# vim /etc/selinux/config  
SELINUX=disabled
注意：此种方式需要重启服务器才能生效。
3．php.ini配置文件的设置
修改php.ini相关配置参数，命令如下：

shell# vim  /etc/php.ini
date.timezone = Asia/Shanghai
max_execution_time = 300
post_max_size = 16M
max_input_time=300
memory_limit = 128M #如果Web页面提示内存不够使用，请调整此值
mbstring.func_overload = 0 #Zabbix 2.2版本请设置为1，以后版本设置为0
在LAMP环境中，也可以按下述方式配置PHP的参数，将相关参数配置于Apache配置文件中。在Zabbix的官方提供的RPM中，这一步已经默认配置过了，一般只需要修改date.timezone即可，配置参数如下：

shell# vim  /etc/httpd/conf.d/zabbix.conf
<Directory "/usr/share/zabbix">
    Options FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
    php_value date.timezone Asia/Shanghai  #修改此参数
    php_value max_execution_time 300
    php_value post_max_size 16M
    php_value max_input_time 300
    php_value memory_limit 128M
    php_value upload_max_filesize 2M
</Directory>
shell# systemctl restart httpd
```



# 43页

```shell
Zabbix-Web连接数据库和Zabbix-Server端口的相关配置信息如下：
shell# cat /etc/zabbix/web/zabbix.conf.php 
<?php
// Zabbix GUI configuration file.[ 要写成中文注释吗？][原配置文件如此，可不加注释] //Zabbix数据库的连接信息
global $DB;

$DB['TYPE']     = 'MYSQL';
$DB['SERVER']   = '127.0.0.1';
$DB['PORT']     = '3306';
$DB['DATABASE'] = 'zabbix';
$DB['USER']     = 'zabbix';
$DB['PASSWORD'] = 'zabbix';

// Schema name. Used for IBM DB2 and PostgreSQL.[同上] //仅对DB2和PostgreSQL
$DB['SCHEMA'] = '';

$ZBX_SERVER      = '127.0.0.1';
$ZBX_SERVER_PORT = '10051';
$ZBX_SERVER_NAME = 'zbx4';

$IMAGE_FORMAT_DEFAULT = IMAGE_FORMAT_PNG;
```



# 44-45页

```shell
（1）/etc/zabbix/zabbix_server.conf 中的参数。

DBHost = 127.0.0.1  	#数据库的IP地址（域名），如不在本机中，则填实际的IP地址
DBName=zabbix       	#数据库的名称
DBUser=zabbix      	#数据库的用户
DBPassword=zabbix   	#数据库的密码
（2）/usr/share/zabbix/conf/zabbix.conf.php中的配置。

<?php
// Zabbix GUI configuration file[中文注释？]                            //Zabbix数据库的连接信息
global $DB;

$DB['TYPE']     = 'MYSQL';             	//数据库类型
$DB['SERVER']   = '127.0.0.1';  			//数据库的IP地址（域名）
$DB['PORT']     = '3306';              	//数据库的端口
$DB['DATABASE'] = 'zabbix';            	//数据库的名称
$DB['USER']     = 'zabbix';             	//数据库的用户
$DB['PASSWORD'] = 'zabbix';           	//数据库的密码

// SCHEMA is relevant only for IBM_DB2 database[同上] //仅对DB2和PostgreSQL
$DB['SCHEMA'] = '';

$ZBX_SERVER      = '127.0.0.1';//Zabbix-Server的IP地址（域名）
$ZBX_SERVER_PORT = '10051';  //Zabbix-Server的端口
$ZBX_SERVER_NAME = 'zbx4';    //Zabbix-Server Web界面的标识

$IMAGE_FORMAT_DEFAULT = IMAGE_FORMAT_PNG;
?>
```



# 49-50页

```shell
shell# zabbix_server -R housekeeper_execute
zabbix_server [1572]: command sent successfully
shell# tail -f /var/log/zabbix/zabbix_server.log
1325:20181013:094504.049 forced execution of the housekeeper
  1325:20181013:094504.049 executing housekeeper
  1325:20181013:094504.494 housekeeper [deleted 2255 hist/trends, 0 items/triggers, 27 events, 1716 problems, 0 sessions, 0 alarms, 6 audit items in 0.444276 sec, idle for 1 hour(s)]
在线执行重载配置缓存，命令如下：

shell## zabbix_server -R config_cache_reload
shell# tail -f /var/log/zabbix/zabbix_server.log #查看日志
1321:20181013:094908 forced reloading of the configuration cache
在线调整日志运行级别，命令如下：

#降低日志运行级别，执行一次，降低一个级别。当前日志级别为4
shell# zabbix_server -R log_level_decrease
zabbix_server [2111]: command sent successfully
shell# tail -f /var/log/zabbix/zabbix_server.log #查看日志
1349:20181013:095908.3 log level has been decreased to 3 (warning)

#降低日志运行级别，执行一次，降低一个级别。当前日志级别为3
shell# zabbix_server -R log_level_decrease
zabbix_server [2120]: command sent successfully
shell# tail -f /var/log/zabbix/zabbix_server.log #查看日志
1361:20181013:095920.244 log level has been decreased to 2 (error)

#增加日志运行级别，执行一次，增加一个级别。当前日志级别为0
shell# zabbix_server -R log_level_increase
zabbix_server [2256]: command sent successfully
shell# tail -f /var/log/zabbix/zabbix_server.log #查看日志
1360:20181013:100341 log level has been increased to 1 (critical)

#调整某个进程(pid)的日志运行级别
shell# ps -ef |grep zabbix
zabbix  955    /usr/sbin/zabbix_server -c /etc/zabbix/zabbix_server.conf
zabbix  1321 955  0 09:40 ? 00:00:00 /usr/sbin/zabbix_server: configuration syncer [synced configuration in 0.030468 sec, idle 60 sec]
zabbix  1322 955  0 09:40 ? 00:00:00 /usr/sbin/zabbix_server: alerter #1 started
zabbix  1323 955  0 09:40 ? 00:00:00 /usr/sbin/zabbix_server: alerter #2 started
zabbix  1324 955  0 09:40 ? 00:00:00 /usr/sbin/zabbix_server: alerter #3 started

#调整进程pid为1322的日志运行级别
shell# zabbix_server -R log_level_increase=1322
shell# tail -f /var/log/zabbix/zabbix_server.log #查看日志
1322:20181013:100800.998 log level has been increased to 2 (error)

#调整增加poller进程的日志运行级别。
shell# zabbix_server -R log_level_increase=poller

#调整增加poller的第3个进程的日志运行级别。
shell# zabbix_server -R log_level_increase=poller,3
```



# 51页

```shell
在这里，我们对需要进行监控的客户端服务器安装Zabbix-Agent，使用RPM方式进行安装，命令如下所示：

shell# rpm -ivh http://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm #安装Zabbix官方的yum源
shell# yum install -y zabbix zabbix-agent  
# 由于Zabbix-Server服务器本身也需要监控，所以在Zabbix-Server服务器中也同样需要安装Zabbix-Agent
```

```shell
如操作系统开启了防火墙，为允许端口能够正常通信，则需使用下列命令进行配置：

#CentOS 6
shell# vi /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 10050 -j ACCEPT
-A OUTPUT -m state --state NEW -m tcp -p tcp --dport 10051 -j ACCEPT
shell# service iptables restart
#CentOS 7
shell# firewall-cmd --permanent --add-port=10050/tcp
shell# firewall-cmd --reload
```



# 52页

```shell
shell# egrep -v "(^#|^$)" /etc/zabbix/zabbix_agentd.conf 
PidFile=/var/run/zabbix/zabbix_agentd.pid  #pid文件路径
LogFile=/var/log/zabbix/zabbix_agentd.log  #日志文件路径
LogFileSize=0                              #日志切割大小，0表示不切割
Server=127.0.0.1,10.0.2.50        	#被动模式，Zabbix-Server的IP地址
ServerActive=127.0.0.1,10.0.2.50  	#主动模式，Zabbix-Server的IP地址
Hostname=Zabbix server              	#本机的Hostname，使用主动模式则必须配置
Include=/etc/zabbix/zabbix_agentd.d/ 	#包含的子配置文件
UnsafeUserParameters=1              	#启用特殊字符，用于自定义监控
```



```shell
配置完成后，使用如下命令进行启动Zabbix-Agent：
#CentOS 7
shell# systemctl enable zabbix-agent    #添加到开机启动项
shell# systemctl start  zabbix-agent    #启动服务

#CentOS 6
shell# chkconfig zabbix-agent on         #添加到开机启动项
shell# service zabbix-agent start        #启动服务
```



# 53页

```shell
在CentOS下配置SNMP监控，命令如下：

shell# yum -y install net-snmp 
shell# vim /etc/snmp/snmpd.conf 
com2sec mynetwork 192.168.0.240 public_monitor
com2sec mynetwork 127.0.0.1 public
group MyROGroup v2c mynetwork
access MyROGroup "" any noauth prefix all none none
view all included .1 80

#CentOS 7
shell# systemctl enable snmpd    #添加到开机启动项
shell# systemctl start  snmpd    #启动服务

#CentOS 6
shell# chkconfig snmpd on        #添加到开机启动项
shell# service snmpd restart     #启动服务
```

# 54页

```shell
使用以下命令注册Zabbix Agent服务，运行结果如图3-26所示。
cmd> zabbix_agentd.exe --install -c "c:\Program  Files\zabbix_agents_ 4.0.0.win\conf\zabbix_agentd.win.conf"   #路径中有空格，应该用双引号，-c后面是配置文件路径
```



# 55页

```shell
（1）采用Windows的net命令控制服务。
启动服务：

C:\> net start "Zabbix Agent"
Zabbix Agent 服务正在启动 .
Zabbix Agent 服务已经启动成功。
停止服务：

C:\> net stop "Zabbix Agent"
Zabbix Agent 服务已成功停止。
（2）采用程序命令方式控制服务。
启动服务：

c:\Program Files\zabbix_agents_4.0.0.win\bin\win64> zabbix_agentd.exe -s -c "c:\Program Files\zabbix_agents_4.0.0.win\conf\zabbix_agentd.win.conf"
Zabbix_agentd.exe [8456]: service  [Zabbix Agent]  started successfully
停止服务：

c:\Program Files\zabbix_agents_4.0.0.win\bin\win64> zabbix_agentd.exe -x -c "c:\Program Files\zabbix_agents_4.0.0.win\conf\zabbix_agentd.win.conf"
Zabbix_agentd.exe [9040]: service  [Zabbix Agent]  stopped successfully
卸载服务：

c:\Program Files\zabbix_agents_4.0.0.win\bin\win64> zabbix_agentd.exe -d -c "c:\Program Files\zabbix_agents_4.0.0.win\conf\zabbix_agentd.win.conf"
zabbix_agentd.exe [2440]: service [Zabbix Agent] uninstalled successfully
zabbix_agentd.exe [2440]: event source [Zabbix Agent] uninstalled successfully
```

# 57页

```shell
shell# zabbix_get  -s 192.168.0.240 -k system.uname
Linux zabbix.itnihao.com 2.6.32-358.el6.x86_64 #1 SMP Fri Feb 22 00:31:26 UTC 2013 X86_64
shell# zabbix_get  -s 192.168.0.103 -k system.uname
Windows ITNIHAO-COM 6.1.7601 Microsoft Windows 7 Ultimate Edition Service Pack 1  x64
shell# zabbix_get  -s 192.168.0.240 -p 10050 -I 127.0.0.1 -k system.uname
Linux zabbix.itnihao.com 2.6.32-358.el6.x86_64 #1 SMP Fri Feb 22 00:31:26 UTC 2013 X86_64
```



# 60页

```sql
mysql> select table_name, (data_length+index_length)/1024/1024 as total_mb, table_rows  from  information_schema.tables  where  table_schema='zabbix';
```



# 62-66页

代码在 https://github.com/zabbix-book/partitiontables_zabbix



# 67页

```mysql
清空语句如下：
mysql> use zabbix; 
mysql> truncate table history; 
mysql> optimize table history; 
mysql> truncate table history_str; 
mysql> optimize table history_str; 
mysql> truncate table history_uint; 
mysql> optimize table history_uint; 
mysql> truncate table history_log; 
mysql> optimize table history_log; 
mysql> truncate table history_text; 
mysql> optimize table history_text; 
mysql> truncate table trends; 
mysql> optimize table trends; 
mysql> truncate table trends_uint; 
mysql> optimize table trends_uint; 
```



```sql
2．运行表分区脚本
为了防止网络中断后引起脚本运行中断而造成数据库故障，我们应该选用screen后台执行的方法。如果没有screen程序，请先安装（运维人员要处处具有谨慎的态度）。

shell# screen  -R  zabbix
shell# sh partitiontables_zabbix.sh 
table history      create partitions p20180716
table history_log  create partitions p20180716
table history_str  create partitions p20180716
table history_text create partitions p20180716
table history_uint create partitions p20180716
table history      create partitions p20180717
table history_log  create partitions p20180717
table history_str  create partitions p20180717
table history_text create partitions p20180717
table history_uint create partitions p20180717
#中间省略部分输出内容
table trends        create partitions p201807
table trends_uint   create partitions p201807
table trends        create partitions p201808
table trends_uint  create partitions p201808
```



# 68页

```
进入screen，可以查看后台运行的任务：
shell# screen  -R  zabbix

shell# crontab -e
1 0 * * * /usr/sbin/partitiontables_zabbix.sh
Shell# chmod 700 /usr/sbin/partitiontables_zabbix.sh
```



```sql
验证表分区是否成功，可以查看history表结构，输出如下：

MariaDB [zabbix]> show create table history\G;
*************************** 1. row ***************************
Table: history
Create Table: CREATE TABLE `history` (
  `itemid` bigint(20) unsigned NOT NULL,
  `clock` int(11) NOT NULL DEFAULT '0',
  `value` double(16,4) NOT NULL DEFAULT '0.0000',
  `ns` int(11) NOT NULL DEFAULT '0',
  KEY `history_1` (`itemid`,`clock`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
/*!50100 PARTITION BY RANGE ( clock)
(PARTITION p20180716 VALUES LESS THAN (1531756799) ENGINE = InnoDB,
PARTITION p20180717 VALUES LESS THAN (1531843199) ENGINE = InnoDB,
PARTITION p20180718 VALUES LESS THAN (1531929599) ENGINE = InnoDB,
PARTITION p20180719 VALUES LESS THAN (1532015999) ENGINE = InnoDB,
PARTITION p20180720 VALUES LESS THAN (1532102399) ENGINE = InnoDB,
PARTITION p20180721 VALUES LESS THAN (1532188799) ENGINE = InnoDB,
PARTITION p20180722 VALUES LESS THAN (1532275199) ENGINE = InnoDB,
PARTITION p20180723 VALUES LESS THAN (1532361599) ENGINE = InnoDB)*/  
/*粗体字部分为表分区*/
1 row in set (0.00 sec)
ERROR: No query specified
MariaDB [zabbix]>
```



# 69页

```sql
在表分区中，使用的是时间戳格式进行日期区间划分，如读者需要将时间转换为时间戳数值格式，使用如下命令进行转换：

shell# date -d "2018-07-16" +%s 
1531756799
将时间戳数值转换为时间格式，命令如下：

shell# date -d @1531756799 "+%Y-%m-%d"
2018-07-16
下面的SQL查询语句是查询指定时间段的数据，以验证数据是否写入。

mysql> select count(*) from history where clock > 1531670399 and clock <1531756799;            
+----------+
| count(*) |
+----------+
|     1356  |
+----------+
1 row in set (0.13 sec)
mysql> select count(*) from history_uint where clock > 1531670399 and clock <1531756799;          
+----------+
| count(*) |
+----------+
|   6302    |
+----------+
1 row in set (0.05 sec)
```

# 70-72页

参考 https://github.com/zabbix-book/zabbix-mysql-backup

# 73页

```shell
该脚本的使用方法如下：
shell# bash /usr/sbin/zabbix_mysqldump.sh mysqldump    #备份数据
shell# bash /usr/sbin/zabbix_mysqldump.sh mysqlimport  #恢复数据
```

```shell
（1）备份软件相关文件和配置文件，命令如下：

shell# mkdir -p /data/zabbix/backup
shell# cp -r /etc/zabbix  /data/zabbix/backup/zabbix_conf
shell# cp -r /usr/share/zabbix /data/zabbix/backup/zabbix_web
shell# cp -r /usr/sbin/zabbix_server /data/zabbix/backup/zabbix_server
shell# cp -r /usr/sbin/zabbix_proxy /data/zabbix/backup/zabbix_proxy
shell# cp -r /usr/share/doc/zabbix-* /data/zabbix-backup/
```

```shell
（3）升级软件，相关操作命令如下：
shell# rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.centos.noarch.rpm
shell# systemctl stop zabbix-server    #停止zabbix-server[注释对吗？]
shell# systemctl stop zabbix-proxy     #停止zabbix-proxy[注释对吗？]
shell# yum upgrade zabbix-server-mysql zabbix-web-mysql zabbix-agent zabbix-get -y
shell# systemctl start zabbix-server  #开启zabbix-server
shell# systemctl start zabbix-proxy   #开启zabbix-proxy   
shell# ps -ef |grep zabbix            #查看进程
shell# tail -f /var/log/zabbix/zabbix_server.log #查看日志
```

