---
title: 从硬件开始搭建自己的服务器（二）
date: 2017-09-24 14:34:44
tags: 从硬件开始搭建自己的服务器
---
<!-- 标签别名 -->
{% cq %} 伤心桥下春波绿，曾是惊鸿照影来。<br/><br/><strong>—— 陆游《沈园二首》</strong> {% endcq %}
<!-- more -->
# 编写启动脚本 #
问题：每次打开服务器都要先插一下TTL，用命令行启动frp，然后才能用ssh连上服务器。能不能开机的时候就自启？并且检测反向代理网络连接是否正常，如果不正常，重新连接，如果连不上及时通知管理者。
接下来我们就一个个实现。
## 实现frp开机自启 ##
在Debian（Ubuntu）环境下，

到开机脚本目录下 ，

```sh
cd /etc/init.d
```

新建一个脚本，名字随意，

```sh
vim frpc
```

在 ``frpc`` 脚本中输入下面这段命令。
```sh
#!/bin/bash
### BEGIN INIT INFO
# Provides:         frpc
# Required-Start:   $remote_fs 
# Required-Stop:    $remote_fs
# Default-Start:    2 3 4 5 
# Default-Stop:     0 1 6 
# Short-Description: frpc
# Description:       frpc
### END INIT INFO

FRPC=frpc
FRPC_CONFIG=frpc.ini
# 你自己的frp目录
FRPC_PATH=/***/**/*
LOG=$FRPC_PATH/$FRPC.log
PIDFILE=/var/run/$FRPC.pid
[ -x "$FRPC_PATH/$FRPC" ] || exit 0

case "$1" in
     start)
         echo "==========分割线===========" >> $LOG
         if [ -e $FRPC_PATH ];then
           ps -ef | grep -v grep | grep $FRPC_PATH/$FRPC_CONFIG
           if [ $? -ne 0 ];then
             echo "$FRPC is start..." >> $LOG
             /sbin/start-stop-daemon -S -m -p $PIDFILE -b  -x \
                    $FRPC_PATH/startFrp.sh \
                         -o -q 
             echo "**********$FRPC  [PID] is  $(cat $PIDFILE)*************" >> $LOG 
             echo "$FRPC is success start"
           else
             echo -n "`date '+%Y/%m/%d %H:%M:%S'` [start] [warning] " >> $LOG
             echo "$FRPC is running" >> $LOG
             echo "$FRPC is running"
           fi
         fi
         ;;
     stop)
         echo "==========分割线===========" >> $LOG
         if [ -f $PIDFILE ];then
           #kill -9 $(cat $PIDFILE)
           /sbin/start-stop-daemon -K -p $PIDFILE -s TERM -o -q || return 2
           if [ $? -eq 0 ];then
             echo -n "`date '+%Y/%m/%d %H:%M:%S'` [stop] [success]" >> $LOG
             echo "$FRPC is stop success" >> $LOG
             echo "$FRPC is stop success"
           else
             echo -n "`date '+%Y/%m/%d %H:%M:%S'` [stop] [*error*]" >> $LOG
             echo "$FRPC is stop error" >> $LOG
             echo "$FRPC is stop error"
           fi
           rm -rf $PIDFILE
         else
           echo -n "`date '+%Y/%m/%d %H:%M:%S'` [stop] [*error*] " >> $LOG
           echo "$FRPC is not running" >> $LOG
           echo "$FRPC is not running"
         fi
         ;;
     restart)
        $0 stop && sleep 2 && $0 start
        ;;
     *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
        ;;
esac
exit 0

```
<font color="red">注意：</font> ``start-stop-daemon`` 要写全路径，不然计划任务 ``crontab`` 执行时，会出现无法找到命令的错误，下面是查找 ``start-stop-daemon`` 全路径的指令。

```sh
which start-stop-daemon
```

设置 ``frpc`` 脚本权限。

```sh
chmod 755 frpc
```
这里有一张图 来说明 755 代表什么意思。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vxj4iaoj30iz0a8jvp.jpg)

在 ``start-stop-daemon`` 运行的那行会发现 ``startFrp.sh`` 脚本，这是由于frp的启动命令直接写在 ``start-stop-daemon`` 里无法运行，只好另外写个 ``startFrp.sh`` 脚本执行。下面贴上 ``startFrp.sh``脚本，我将它放置在frp目录下。
```sh
#!/bin/bash

FRPC=frpc
FRPC_CONFIG=frpc.ini
CRON=cronParameter
FRPC_CRON=/home/frp
FRPC_PATH=/home/frp/frp_0.13.0_linux_arm
LOG=$FRPC_PATH/$FRPC.log
PIDFILE=/var/run/$FRPC.pid
FLAG=0

#判断网络是否已经连上
for i in `seq 3`
do
  ping -c 1 www.baidu.com > /dev/null 2>&1
  if [ $? -ne 0 ]
  then
    sleep 10
  else
    FLAG=1
    break
  fi
done

if [ $FLAG -eq 1 ]
then
  #末尾要加& 否则$！取得是 脚本的PID
  $FRPC_PATH/$FRPC -c $FRPC_PATH/$FRPC_CONFIG >> $LOG 2>> $LOG & 
  echo $! > $PIDFILE
  sed -i "/network=/c network=normal" $FRPC_CRON/$CRON
else
  sed -i "/network=/c network=error" $FRPC_CRON/$CRON
fi


```
设置脚本权限。
```sh
chmod 711 startFrp.sh
```


接着试下脚本运行是否正常。

```sh
./frpc start
```


如果正常运行，则将脚本加入开机启动项中。

```sh
update-rc.d frpc defaults 90
```

如果没有什么意外，开机就能启动frp了。

## 发送mail ##
安装mail。

```sh
apt-get install heirloom-mailx
```

修改配置文件。

```sh
vim /etc/s-nail.rc 
```

<font color="red">注意：</font> 不同的版本配置文件命名不一样，有可能是 ``nail.rc`` 或 ``mail.rc``
在配置文件末尾添加上这几句。

```sh
set from=9********3@qq.com(cg)
set smtp=smtp.qq.com
set smtp-auth-user=9********3@qq.com
set smtp-auth-password=g*************b
set smtp-auth=login
set smtp-use-starttls
set ssl-verify=ignore
set nss-config-dir=/etc/pki/nssdb/ 

```
注意： 这里的密码不是填QQ的登录密码，要填开通SMTP服务时给的密码。
QQ邮箱开通SMTP服务： 设置 -> 账户

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vxx99xpj30li02j74b.jpg)

发送邮件。

```sh
echo "test（内容）" | mail -s "firstTest（主题）" 9*****@qq.com
```

可以看到QQ邮箱收到的邮件。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vy9qrzzj30k007hdg9.jpg)

## 定时检查网络连接情况，并mail通知 ##
用 ``crontab`` 定时计划命令，在机器重启后仍可以运行，而且在运行时间准确，易于控制。这边有篇详尽介绍crontab命令的博文（点[链接](http://www.cnblogs.com/peida/archive/2013/01/08/2850483.html)）
这里贴上我写的脚本。
 ``cronFrp.con`` 定时计划，一小时执行一次。
```sh
01 * * * * /***/**/cronFrp.sh >>/***/**/frpc.log 2>>/***/**/frpc.log
```
 ``cronFrp.sh``判断frp服务及网络是否正常，并发送mail。
```sh
#!/bin/bash

FRPC=frpc
FRPC_CONFIG=frpc.ini
CRON=cronParameter
FRPC_CRON=/***/**/frp
FRPC_PATH=/***/**/*
LOG=$FRPC_PATH/$FRPC.log

# 获取参数
error=$(grep 'error=' $FRPC_CRON/$CRON | awk -F '=' '{print $2}')
sendDate=$(grep 'sendDate=' $FRPC_CRON/$CRON | awk -F '=' '{print $2}')
network=$(grep 'network=' $FRPC_CRON/$CRON | awk -F '=' '{print $2}')
network_restart=$(grep 'network_restart=' $FRPC_CRON/$CRON | awk -F '=' '{print $2}')
# 标志位 0为网络异常
FLAG=0

# 检测网络状态
ping -c 1 www.baidu.com > /dev/null 2>&1
isTruePing=$?

# 获取日志最后一行
sed -n '$p' $LOG | grep 'reconnect to server'
if [ $? -eq 0 ]
then
  # 停止 frp 进程
  /etc/init.d/frpc stop
  if [ $isTruePing -eq 0 ]
  then 
    FLAG=1
  else
    FLAG=0
  fi
else
  if [ $isTruePing -eq 0 ]
  then
    FLAG=1
  else
    FLAG=0
  fi
fi

if [ $FLAG -eq 1 ]
then
  sed -i "/network=/c network=normal" $FRPC_CRON/$CRON
else
  sed -i "/network=/c network=error" $FRPC_CRON/$CRON

fi

# 如果本机网络异常
if [ "$network" != "error" ]
then
   # 复位 重启标志位
   sed -i "/network_restart=/c network_restart=0" $FRPC_CRON/$CRON
   # 判断 进程是否存在
   ps -ef | grep -v grep | grep $FRPC_PATH/$FRPC_CONFIG >> /dev/null 2>&1
   if [ $? -ne 0 ]
   then
     if [ $error -lt 3 ]
     then
        /etc/init.d/frpc start
        error=$(($error + 1))
        # 将值 存入 参数文件中
        sed -i "/error=/c error=$error" $FRPC_CRON/$CRON
     else
        echo "`date '+%Y/%m/%d %H:%M:%S'` [service] [*error*] frpService is error, can't start the frpc"
        sed -i "/error=/c error=0" $FRPC_CRON/$CRON
        nowTime=$(date +%s)
        # 得到时间差
        time=$(( $nowTime-$sendDate ))
        # 发送邮件的时间差 大于一天
        if [ $time -ge $((60*60*24)) ]
        then
           echo "frp服务错误，请检查frps是否正常启动。" | mail -s "frp服务错误" 9*****@qq.com
           sed -i "/sendDate=/c sendDate=$nowTime" $FRPC_CRON/$CRON
        fi
     fi
   else
     echo "`date '+%Y/%m/%d %H:%M:%S'` [service] [success] frp  is normal"
     if [ $error -ne 0 ]
     then
       sed -i "/error=/c error=0" $FRPC_CRON/$CRON
     fi
   fi
else
   if [ $network_restart -eq 0 ]
   then
     sed -i "/network_restart=/c network_restart=1" $FRPC_CRON/$CRON
     echo "`date '+%Y/%m/%d %H:%M:%S'` [network] [warning] service is restart, wait...."
     #重启 服务器
     /sbin/reboot
   else
     echo "`date '+%Y/%m/%d %H:%M:%S'` [network] [*error*] network is error , can't connect the internet!"
   fi
fi

```
脚本检查frp服务及网络是否正常，如果是frp服务问题，则停止frpc进程，三次无法启动frpc服务，则发送mail，mail发送时间间隔为一天。如果为网络问题，则重启服务器。
## 日志清理 ##
清除三天前的记录。
日志记录格式：
```sh
2017/10/05 12:01:02 [I] [control.go:318] [00e1c17********] new proxy [web] success
==========分割线===========
2017/10/05 12:04:33 [stop] [success]frps is stop success
==========分割线===========
frps is start...
**********frps  [PID] is  19141*************
2017/10/05 12:18:47 [I] [service.go:83] frps tcp listen on 0.0.0.0:7000
2017/10/05 12:18:47 [I] [service.go:108] http service listen on 0.0.0.0:8080
2017/10/05 12:18:47 [I] [main.go:112] Start frps success
2017/10/05 12:18:47 [I] [main.go:114] PrivilegeMode is enabled, you should pay more attention to security issues
2017/10/05 12:19:02 [I] [service.go:230] client login info: ip [112.*.*.*:39603] version [0.13.0] hostname [] os [linux] arch [arm]

```
清除记录脚本 ``clearLog.sh``：
```sh
#!/bin/bash

FRPC=frpc
FRPC_CONFIG=frpc.ini
CRON=cronParameter
FRPC_CRON=/***/frp
FRPC_PATH=/***/**/*
LOG=$FRPC_PATH/$FRPC.log

endTime=`date -d '-2 day' '+%Y\\/%m\\/%d'`

# 从第一行开始 删除到 前三天的记录
sed -i "1,/$endTime/d" $LOG
if [ $? -eq 0 ]
then
  echo "`date '+%Y/%m/%d %H:%M:%S'` [clear] [success] frpcLog is success clear"
else
   echo "`date '+%Y/%m/%d %H:%M:%S'` [clear] [*error*] frpcLog clear is fail"
fi

```
日志清理每周日凌晨0点清理：
```sh
59 23 * * 7 /***/**/clearLog.sh >>/***/**/frpc.log 2>>/***/**/frpc.log
```
做到这里，差不多就能把服务器丢在家里不用管。