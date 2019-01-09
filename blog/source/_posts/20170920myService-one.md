---
title: 从硬件开始搭建自己的服务器（一）
date: 2017-09-20 19:19:09
tags: 从硬件开始搭建自己的服务器
---
<!-- 标签别名 -->
{% cq %} 洛阳城东西，长作经时别。昔去雪如花，今来花似雪。<br/><br/><strong>—— 范云《别诗二首·其一》</strong> {% endcq %}
<!-- more -->
## 硬件选择 ##
直接买一台电脑主机？有这么多钱就不用下面这么折腾了...
网上有很多小型主机板，出名的有树莓派、香蕉派等等，这些都是不错的选择，可以考虑买二手的板子，对于硬件来说，二手和一手其实相差不是特别大。我在闲鱼上100块淘了一个二手的Cubieboard2，处理器是全志A20，双核cpu，内存1G，4G的Nand Flash。具体的硬件信息点  [链接](http://www.waveshare.net/shop/Cubieboard2-Package-C.htm "链接")，这个配置，作为个人的服务器差不多够用了。
## 系统烧录 ##
有些买的Cubieboard板Nand Flash里直接就有Linux系统，懒得折腾就直接用。
Cubieboard2启动顺序是先检测TF卡，然后再启动Nand Flash里的系统，因此如果想要用自己编译的Linux就需要一张TF卡，2G以上，将编译好的Linux烧录到TF卡中，插到板上启动。
自己编译的Linux...嗯...这里有一篇很给力的文档（[点击下载](http://pan.baidu.com/s/1skZjRdF)），直接照着文档里的步骤跟着走就好了..
虽然这个方法折腾了三天，最后败在最后一步，没有成功，但是还是有经验教训可以传授...跟文档里作者的编译环境一定要一样，不然会出现各种奇怪的错误，如果是用虚拟机跑Ubuntu，TF卡识别可能会慢一点，多试几次，在TF卡分区后，系统有可能识别不出来分区后的空间，``reboot`` 后敲  ``partprobe`` ，在 ``/dev`` 目录中如果可以找到分区空间的设备名就说明成功识别了分区，如果还是没有，就多来几次或者删掉分区在来一次。就像文档中摘要说的一样
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vu90hw9j30k204mdh0.jpg)
最后我选择了一种不折腾的方法。先不说啥方法，丢一个链接再说（[点这里](http://linux-sunxi.org/Bootable_OS_images#Ubuntu)）。这个网址里面有很多牛人已经编译好了的系统镜像，可供下载直接运行。这么简单粗暴，干嘛不用？最后我选择了Armbian的Debian服务版，到[Armbian官网](https://www.armbian.com/)什么都变得按部就班，分分钟开启启动。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vuiinw1j30i408c75d.jpg)

这里是用TTL连接电脑，如果买板的时候没有配TTL的线，只要去淘宝PL2303USB转TTL的线即可。和硬件连接的时候，黑线接GND，绿线接RX，白线接TX，一定要<font color="#FF0000">注意红色的线不能插！！</font>，其实我也没试过，不过官网写着会烧板啥的很危险。这里提供一个PL2303的[Win10驱动](http://pan.baidu.com/s/1qXIWSuo)。
连接上电脑后，``ifconfig`` 获取到板子的IP后，就可以把板子丢一边了，用ssh连接。
## 配置Java环境 ##
在下载JDK时，选择Linux-arm。
Linux下配置JDK有两种方法，一个是在 ``/etc/profile`` 下，这个配置对所有的用户生效，更推荐的是第二个方法 ``~/.bashrc`` ，只对当前用户生效，更安全。
```sh
export JAVA_HOME=/*/**/jdk1.8.0_144
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
tomcat运行前需要赋予 ``.sh``文件权限
```sh
chmod u+x *.sh
```
## ngrok ##
ngrok 是一个反向代理，通过在公共的端点和本地运行的 Web 服务器之间建立一个安全的通道。ngrok 可捕获和分析所有通道上的流量，便于后期分析和重放。
简单的说就是别人想通过外网地址访问你内网的网站就可以用到这个东西，花生壳，nat123等的也都是做这种内网穿透，但是对Linux-arm的支持不太友好，ngrok免费，对各个系统都有支持的版本，因此就是它了。
这里提供一个[自己搭建ngrok的教程](http://www.jianshu.com/p/91f01e30a9b0)，有时间再折腾，但是看一遍也差不多理解ngrok 的一个工作流程是啥子。网上有很多大神搭好了ngrok环境，对外开放。我们选择[sunny-ngrok](https://www.ngrok.cc/)，官网有详细教程。
开通两个隧道：

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vv0ji38j30cf036wef.jpg)

tcp隧道用于 ssh 连接，http用于映射 tomcat 端口，服务器上启动隧道的时候两个都要启。
输入你开通隧道时自定义的域名，就可以成功访问到tomcat的主页啦~
## frp ##
frp类似ngrok，也是内网穿透，但是相对ngrok来说，更加稳定可靠，而且，配置文件配置起来简直不要太爽..
点进作者的 [Github](https://github.com/fatedier/frp/blob/master/README_zh.md) ，下面有详细文档及配置信息。
下面贴上我写的frps服务端的启动脚本
```sh
#!/bin/bash

FRPS=frps
FRPSCONFIG=frps.ini
FRPS_PATH=/***/**/*
LOG=/***/**/*/frps.log
PIDFILE=/var/run/$FRPS.pid
# 判断 是否 有权限执行
[ -x "$FRPS_PATH/$FRPS" ] || exit 0

case "$1" in
     start)
         echo "==========分割线===========" >> $LOG
         if [ -e $FRPS_PATH ];then
           ps -ef | grep -v grep | grep $FRPS_PATH/$FRPSCONFIG
           if [ $? -ne 0 ];then
             echo "$FRPS is start..." >> $LOG
             nohup $FRPS_PATH/$FRPS -c $FRPS_PATH/$FRPSCONFIG >> $LOG &
             echo $! > $PIDFILE
             echo "**********$FRPS  [PID] is  $(cat $PIDFILE)*************" >> $LOG
             echo "$FRPS is success start"
           else
             echo -n "`date '+%Y/%m/%d %H:%M:%S'` [start] [warning] " >> $LOG
             echo "$FRPS is running" >> $LOG
             echo "$FRPS is running"
           fi
         fi
         ;;
     stop)
         echo "==========分割线===========" >> $LOG
         if [ -f $PIDFILE ];then
           kill -9 $(cat $PIDFILE)
           if [ $? -eq 0 ];then
             echo -n "`date '+%Y/%m/%d %H:%M:%S'` [stop] [success]" >> $LOG
             echo "$FRPS is stop success" >> $LOG
             echo "$FRPS is stop success"
           else
             echo -n "`date '+%Y/%m/%d %H:%M:%S'` [stop] [*error*]" >> $LOG
             echo "$FRPS is stop error" >> $LOG
             echo "$FRPS is stop error"
           fi
           rm -rf $PIDFILE
         else
           echo -n "`date '+%Y/%m/%d %H:%M:%S'` [stop] [*error*] " >> $LOG
           echo "$FRPS is not running" >> $LOG
           echo "$FRPS is not running"
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
给脚本加上执行权限 ``chmod +x frps.sh`` 后，输入  ``./frps.sh start`` 即可启动frp服务。客户端的写法和服务端的类似。