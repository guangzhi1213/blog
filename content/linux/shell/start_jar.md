---
title: "Start_jar"
date: 2021-04-25T11:26:21+08:00
lastmod: 2021-04-25T11:26:21+08:00
draft: false
keywords: []
description: ""
tags: ["linux","sre","ops"]
categories: ["linux"]
author: "王清"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

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

---

### 编写jar启动脚本

```shell
#!/bin/bash
#
# 主要是根据PORT和prog启动和停止jar程序
#sudo -u nobody nohup java -Dfile.encoding=UTF-8 -Xms384m -Xmx768m -jar DiscoveryPay-wechat.jar &>> /logs/nohup/wechat.log &

prog=ldyspay
JARFILE=DiscoveryPay-ldyspay.jar
EXEC='java -Dfile.encoding=UTF-8 -Xms384m -Xmx768m -jar'
LOGFILE=/logs/nohup/${prog}.log

if [ ! -f $LOGFILE ]; then
    touch $LOGFILE
fi
#sudo -u nobody nohup $EXEC $JARFILE &>> $LOGFILE &


PORT=40030
HOME='/yst/DP'
pid=$(ss -tnlp | grep :${PORT}[[:space:]] | cut -d, -f 2)

start() {
    echo pid:$pid
    if [ ${pid}X != X ]; then
        echo "service ${prog} is already up"
        return 0
    fi
    cd ${HOME}/${prog}
    chown -R nobody:nobody ${JARFILE} ${LOGFILE}
    nohup ${EXEC} ${JARFILE} &>> ${LOGFILE} &
    echo "the program $prog need 2 minutes to started"
    pid=$(ss -tnlp | grep :80[[:space:]] | cut -d, -f 2)
    if [ ${pid}X != X ]; then
        echo "service ${prog} started"
    else
        echo "unknown failed"
        return 2
    fi
}
stop() {
    if [ ${pid}X == X ]; then
        echo "not found prog for ${prog}"
        return 0
    fi
    kill -9 ${pid}
    echo "kill program use signal 2,pid: ${pid}"
}
status() {

    if [ ${pid}X == X ]; then
        echo "not found program for ${prog}"
    else
        echo "program is running."
    fi
}
log1() {
    tailf ${LOGFILE}
}
case $1 in
    "start")
    start
;;
    "stop")
    stop
;;
    "status")
    status
;;
esac
```
