---
layout: post
title: 'Ubuntu14.04配置nginx开机自启动项'
description: 'Linux系统配置'
category: linux
tags: nginx
---

这里需要特别说明的是，Ubuntu系统下没有RedHat系统下的chkconfig命令。
但Ubuntu有一个类似的命令: `sysv-rc-conf`。

通过apt-get命令完成`sysv-rc-conf`软件的安装。

##背景
Linux系统的运行级别有7个，分别对应的:

* 0: 关机
* 1: 单用户（维护）
* 2~5: 多用户
* 6: 重启

可以通过`runlevel`命令来查看当前系统的运行等级:

```
wds@wds-VirtualBox:~$ runlevel
N 2
```

其中第一个表示上一次的运行等级，N表示没有上一次运行等级的记录；第二个表示**当前运行等级**，这里为2.

Linux中所有开机自启动项目运行脚本都放在`/etc/init.d/`目录下;同时在`/etc/`目录下有rc?.d目录，分别对应了7中不同的运行级别：

```
wds@wds-VirtualBox:/$ ls  /etc/ | grep ^rc
rc0.d
rc1.d
rc2.d
rc3.d
rc4.d
rc5.d
rc6.d
rc.local
rcS.d
```
这里rc2.d目录就对应了我们系统当前的运行等级。

其中里面的一些文件其实都是`/etc/init.d/`目录下文件的软链接:

```
wds@wds-VirtualBox:/etc/rc2.d$ ls -ltr
total 4
-rw-r--r-- 1 root root 677  3月 13  2014 README
lrwxrwxrwx 1 root root  18 12月  8 19:49 S99rc.local -> ../init.d/rc.local
lrwxrwxrwx 1 root root  18 12月  8 19:49 S99ondemand -> ../init.d/ondemand
lrwxrwxrwx 1 root root  18 12月  8 19:49 S70pppd-dns -> ../init.d/pppd-dns
lrwxrwxrwx 1 root root  19 12月  8 19:49 S70dns-clean -> ../init.d/dns-clean
lrwxrwxrwx 1 root root  15 12月  8 19:49 S50saned -> ../init.d/saned
lrwxrwxrwx 1 root root  27 12月  8 19:49 S20speech-dispatcher -> ../init.d/speech-dispatcher
lrwxrwxrwx 1 root root  15 12月  8 19:49 S20rsync -> ../init.d/rsync
lrwxrwxrwx 1 root root  20 12月  8 19:49 S20kerneloops -> ../init.d/kerneloops
lrwxrwxrwx 1 root root  21 12月  9 17:25 S99grub-common -> ../init.d/grub-common
lrwxrwxrwx 1 root root  15 12月  9 17:45 S20nginx -> ../init.d/nginx
lrwxrwxrwx 1 root root  17 12月  9 17:47 S20php-fpm -> ../init.d/php-fpm
```
整个开机自启动项的流程如下：

1. 开机后，系统获得当前的运行等级（例如这里的等级为2）；
2. 运行`/etc/rc?.d`目录下的所有可执行文件（这里运行`/etc/rc2.d/`目录下所有的软链接。这些软链接的源文件都保存在`/etc/init.d/`目录下）。

因此我们只需要在/etc/init.d/完成启动`nginx`进程的脚本，然后在`/etc/rc2.d/`做对应的软链接即可。



## 配置nginx自启动文件

[创建`/etc/init.d/nginx`文件](http://www.linuxidc.com/Linux/2011-10/45735.htm)

```
#! /bin/sh
# Author: rui ding
# Modified: Geoffrey Grosenbach http://www.linuxidc.com
# Modified: Clement NEDELCU
# Reproduced with express authorization from its contributors
set -e
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DESC="nginx daemon"
NAME=nginx
DAEMON=/usr/local/nginx/sbin/$NAME
SCRIPTNAME=/etc/init.d/$NAME


# If the daemon file is not found, terminate the script.
test -x $DAEMON || exit 0

d_start() {
        $DAEMON || echo -n " already running"
}

d_stop() {
        $DAEMON –s quit || echo -n " not running"
}

d_reload() {
        $DAEMON –s reload || echo -n " could not reload"
}

case "$1" in
	start)
echo -n "Starting $DESC: $NAME"
d_start
echo "."
;;
stop)
echo -n "Stopping $DESC: $NAME"
d_stop
echo "."
;;
reload)
echo -n "Reloading $DESC configuration..."
d_reload
echo "reloaded."
;;
restart)
echo -n "Restarting $DESC: $NAME"
d_stop
# Sleep for two seconds before starting again, this should give the
# Nginx daemon some time to perform a graceful stop.
sleep 2
d_start
echo "."
;;
*)
echo "Usage: $SCRIPTNAME {start|stop|restart|reload}" >&2
exit 3
;;
esac
exit 0
```

然后利用sysv-rc-conf命令将其在对应rc?.d目录下建立一个软链接：

```
root@wds-VirtualBox:~# sysv-rc-conf nginx on
```

该命令会在rc2.d ~ rc5.d目录下都建立了一个nginx的软链接。
