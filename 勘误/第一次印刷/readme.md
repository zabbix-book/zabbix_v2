# 81页 拼写错误
`Micros`修改为`Macros`

# 118页 
单击`Configuration`->Authentication 修改为`Administration`->Authentication

# 127页 增加说明

· Numeric (unsigned)：64位无符号整数

· Numeric (float)：浮点数，值范围为-999999999999.9999~999999999999.9999

· Character：字符（字符串）类型，数据限制为255B

· Log：日志`数据，数据库表中的TEXT数据类型，参考图3-30表结构`

Text：文本`数据，数据库表中的TEXT数据类型，参考图3-30表结构`

# 221页 调整格式

shell# **cp wechat-alert-master/wechat_linux_amd64** `增加一个空格` /etc/zabbix/alertscripts/wehchat

# 222页 调整格式

shell#`mkdir  -p  /etc/zabbix/alertscripts/` 字体加粗

shell#`cat  /etc/zabbix/alertscripts/zabbix_sendmail.py ` 字体加粗

# 224页 调整格式

shell# **chmod  700  /etc/zabbix/alertscripts/zabbix_sendmail.py**

shell#  `多一个空格`**chown  zabbix.zabbix  /etc/zabbix/alertscripts/zabbix_sendmail.py**

# 345页 调整格式

shell# **vi** `加粗`/etc/logrotate.d/zabbix_traps 

# 385、387、388 拼写错误

将”OBDC”修改为”ODBC”

# 436页 路径错误
`vim /etc/zabbix/scripts/nginx_status` 修改为 `vim /etc/zabbix/scripts/check_nginx_status.sh`
# 439页 路径错误
`vim /etc/zabbix/scripts/monitor_phpfpm_status` 修改为 `vim /etc/zabbix/scripts/check_phpfpm.sh`

# 471页 示例错误

修改后的内容如下  

------

将代码和配置文件放到相应的目录下：

\#代码和配置见https://github.com/zabbix-book/Pyora/

shell# **cp** **pyora.py** **/etc/zabbix/scripts/pyora.py**

shell# **cp py_oracle.conf /etc/zabbix/zabbix_agentd.conf.d/oracle.conf**

 

# 538页 调整格式

```python
38. #认证和获取sessionid
39. try: 
40.     result = urllib2.urlopen(request) #此处增加4个空格
41. #对认证出错的处理
42. except HTTPError, e:
43.     print 'The server couldn\'t fulfill the request, Error code: ', e.code
44. except URLError, e:
45.     print 'We failed to reach a server.Reason: ', e.reason
46. else: 
```



# 579页 增加链接

增加链接`完整脚本请参考https://github.com/zabbix-book/zabbix-HA`

```
shell# cat /etc/keepalived/chk_zabbix_server.sh 
#!/bin/bash
# 完整脚本请参考https://github.com/zabbix-book/zabbix-HA
# 
status1=$(ps aux|grep "/usr/sbin/zabbix_server" | grep -v grep | grep -v bash | wc -l)

if [ "${status1}" = "0" ]; then

        /etc/init.d/zabbix-server start
        sleep 3

        status2=$(ps aux|grep zabbix_server | grep -v grep | grep -v bash |wc -l)
        if [ "${status2}" = "0"  ]; then
                /etc/init.d/keepalived stop
        fi
fi
```



 
