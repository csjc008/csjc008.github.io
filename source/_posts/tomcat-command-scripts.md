title: Tomcat 启动停止等相关操作脚本
date: 2015-01-11
tags:
---

下面贴一些方便的单机多实例部署tomcat的操作脚本，在实际使用中还是很方便的。
<!--more-->
启动脚本：(需要相应设置YOUR_TOMCAT_DIR和YOUR_DEPLOY_DIR)
```bash
#!/bin/bash
source /etc/profile

export CATALINA_HOME=YOUR_TOMCAT_DIR

if echo $1 | grep -q "YOUR_DEPLOY_DIR"
then
    export CATALINA_BASE=${1%/}
else
    export CATALINA_BASE=YOUR_DEPLOY_DIR/${1%/}
fi

instance=`ls YOUR_DEPLOY_DIR | head -1`;
if ! [ -e $CATALINA_BASE/conf/server.xml ]
then
    echo -e " usage: $0 YOUR_DEPLOY_DIR/$instance\n"
    exit 1;
fi


if [ -r "$CATALINA_BASE"/startenv.sh ]; then
  . "$CATALINA_BASE"/startenv.sh
fi

TOMCAT_ID=`ps aux |grep "java"|grep "Dcatalina.base=$CATALINA_BASE"|grep -v "grep"|awk '{ print $2}'`


if [ -n "$TOMCAT_ID" ] ; then
    echo "tomcat(${TOMCAT_ITOMCAT_ID}) still running now , please shutdown it firest";
    exit 2;
fi

TOMCAT_START_LOG=`$CATALINA_HOME/bin/startup.sh`


if [ "$?" = "0" ]; then
    echo "$0 ${1%/} start succeed"
else
    echo "$0 ${1%/} start failed"
    echo $TOMCAT_START_LOG
fi
```

停止脚本：
```bash
#!/bin/bash
source /etc/profile

export CATALINA_HOME=YOUR_TOMCAT_DIR

if echo $1 | grep -q "YOUR_DEPLOY_DIR"
then
        export CATALINA_BASE=${1%/}
else
        export CATALINA_BASE=YOUR_DEPLOY_DIR/${1%/}
fi

instance=`ls YOUR_DEPLOY_DIR | head -1`;
if ! [ -e $CATALINA_BASE/conf/server.xml ]
then
	echo -e " usage: $0 YOUR_DEPLOY_DIR/$instance\n"
        exit 1;
fi


TOMCAT_ID=`ps aux |grep "java"|grep "[D]catalina.base=$CATALINA_BASE "|awk '{ print $2}'`


if [ -n "$TOMCAT_ID" ] ; then
    TOMCAT_STOP_LOG=`$CATALINA_HOME/bin/shutdown.sh`
else
    echo "Tomcat instance not found : ${1%/}"
    exit

fi



for i in {1..10}; do

    TOMCAT_ID=`ps aux |grep "java"|grep "Dcatalina.base=$CATALINA_BASE "|grep -v "grep"|awk '{ print $2}'`

    if [ -n "$TOMCAT_ID" ]; then

        if [ "$i" = "1" ]; then
             echo -n "trying stop ($TOMCAT_ID): $i"
        else
             echo -n -e "\b$i"
        fi


        if [ $i -ge 5 ]; then
             kill "$TOMCAT_ID"
        fi

        sleep 1

    else
        if [ $i -gt 5 ]; then
            echo -e "\n$TOMCAT_BASE was killed($i)"
        else
            echo -e "\n$TOMCAT_BASE was stoped"
        fi

        exit;
    fi

done;

kill -9 "$TOMCAT_ID"

echo "$TOMCAT_BASE was force killed"
```

重启脚本：
```bash
#!/bin/bash

CD_TO/stop_tomcat.sh $1

CD_TO/start_tomcat.sh $1
```