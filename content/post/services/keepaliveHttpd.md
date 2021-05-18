---
title: "KeepaliveHttpd"
date: 2021-04-25T11:08:54+08:00
lastmod: 2021-04-25T11:08:54+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

# ha httpd

```
# ha1 conf
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.8.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}


# vrrp_script
vrrp_script check_httpd {
        script "killall -0 nginx"
        # script "if [ -f /var/run/nginx/nginx.pid ]; then exit 0; else exit 1;fi"
        # script "、etc/keepalived/check_mysql.sh" 脚本在下边
        #kill -0 pid 不发送任何信号，但是系统会进行错误检查。
        # killall -0 0表示不关闭某个程序， 而表示对程序(进程) 的运行状态进行监控， 如果发现进程有异常则返回1
        #所以经常用来检查一个进程是否存在，存在返回0；不存在返回1
        interval 2
        # fall 2 失败两次才判断为失败
        # rise 1 成功一次即为成功
}


vrrp_instance HA_1 {
    state MASTER
    interface ens33
    virtual_router_id 80
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    notify_master "/etc/keepalived/master.sh"
    notify_backup "/etc/keepalived/backup.sh"
    notify_fault "/etc/keepalived/fault.sh"

    track_script {
        check_httpd
    }
    virtual_ipaddress {
        192.168.8.30
    }
}
```

```
# ha2 conf
将 state MASTER 改为 state BACKUP
将 priority 100 改为 priority 80
```

```
#master.sh
LOGFILE=/var/log/keepalived-mysql-state.log
echo "[Master]" >> $LOGFILE
date >> $LOGFILE

#backup.sh
LOGFILE=/var/log/keepalived-mysql-state.log
echo "[Backup]" >> $LOGFILE
date >> $LOGFILE

#fault.sh
LOGFILE=/var/log/keepalived-mysql-state.log
echo "[Fault]" >> $LOGFILE
date >> $LOGFILE
```

```
#!/bin/bash
# check_mysql.sh
MYSQL=/usr/bin/mysql
MYSQL_HOST=localhost
MYSQL_USER=root
MYSQL_PASSWORD='123'

$MYSQL -h $MYSQL_HOST -u $MYSQL_USER -p$MYSQL_PASSWORD -e "show status;" > /dev/null 2> &1

if [ $? == 0 ];then
    MYSQL_STATUS=0
else
    MYSQL_STATUS=1
fi

exit $MYSQL_STATUS
```

非抢占模式， 只需加入 nopreempty
