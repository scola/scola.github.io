---
layout: post
title: "在openwrt-tplink703n路由器上成功部署GoAgent"
category: 日志
tags: goagent openwrt router linux
---
之前学习使用[goagent](https://code.google.com/p/goagent/)的使用的时候，看过[goagnet的wiki](https://code.google.com/p/goagent/w/list),发现可以将goagent部署在路由器上，可以让连接路由器的所有设备自动凸墙，无需在每个设备上安装代理软件，所以还是值得折腾一下的。

最近为了实现一个想法，需要一款开源系统的路由器，由于我也是一个linux小白用户，所以尽量选取大众一点且资料丰富的路由器系统和设备。从小虾的这篇[在路由器上写CMCC自动登录验证脚本](http://xiaoxia.org/2012/04/28/mini-router-auto-login-cmcc/)了解到TP-Link 703n这款路由器，在他推荐的那家淘宝店买来玩了下。店家已经刷好了openwrt系统，也不用自己刷机担心成砖了，动手安装

###挂载U盘

因为703n自身的flash只有4M，我买的那个被店家改装过了，增加到了8M.想直接安装python到flash上，存储肯定是不够的，我也尝试了下，的确提示内存不足，而且goagent本身也好几M。挂载U盘是必须的。首先找一台linux电脑将U盘格式化成ext4格式，然后创建分区  
{% highlight sh  %}
mkfs.ext4 /dev/sda
fdisk /dev/sda
{% endhighlight %}
将U盘插在路由器上，开始挂载U盘
{% highlight sh  %}
mkdir /mnt/sdc1
mount /mnt/sdc1 /dev/sda1
{% endhighlight %}
运行df -h就会发现U盘已经挂载成功了
{% highlight sh  %}
root@OpenWrt:~# df -h
Filesystem                Size      Used Available Use% Mounted on
rootfs                    1.9M      1.0M    876.0K  54% /
/dev/root                 5.3M      5.3M         0 100% /rom
tmpfs                    30.3M    196.0K     30.1M   1% /tmp
tmpfs                   512.0K         0    512.0K   0% /dev
/dev/mtdblock3            1.9M      1.0M    876.0K  54% /overlay
overlayfs:/overlay        1.9M      1.0M    876.0K  54% /
/dev/sda1                 1.0G    123.8M    856.7M  13% /mnt/sdc1 
{% endhighlight %}
修改配置执行开机自动挂载
{% highlight sh %}
root@OpenWrt:/etc/config# cat fstab 
config global automount
    option from_fstab 1
    option anon_mount 1
    
config global autoswap
    option from_fstab 1
    option anon_swap 0
    
config mount
    option target    /mnt/sdc1
    option device    /dev/sda1
    option fstype    ext4
    option options    rw,sync
    option enabled    1
    option enabled_fsck 0

config swap
    option device    /dev/sda2
    option enabled    1
{% endhighlight %}
调整启动顺序,关于linux的启动顺序，可以参考[Linux 的启动流程](http://www.ruanyifeng.com/blog/2013/08/linux_boot_process.html)
{% highlight sh %}
root@OpenWrt:/etc/rc.d# mv S39usb S19usb
{% endhighlight %}
###安装python到U盘

首先改变opkg的安装目录
{% highlight sh  %}
mkdir /mnt/sdc1/opt
ln -sf /mnt/sdc1/opt /opt
{% endhighlight %}
修改/etc/opkg.conf
{% highlight sh  %}
dest root /opt
{% endhighlight %}
在我的电脑上执行opkg update，然后opkg install python会出现md5sum mismatch 错误
>* opkg_install_pkg: Package ddns-scripts md5sum mismatch. Either the opkg or the package index are corrupt. Try 'opkg update'.

这个问题可能是因为linux内核版本和下载源不匹配造成的，一般是下载源比较新，系统比较老，还没有更新,尝试更改下载源地址http://downloads.openwrt.org/snapshots/trunk/ar71xx/packages/，变为http://downloads.openwrt.org/kamikaze/8.09.2/ar7/packages/，然后opkg update，此时/var/opkg-lists/snapshots会更新md5，然后从官方源中手动下载python的ipk文件，依赖什么就下载安装什么，虽然有些麻烦。
{% highlight sh %}
opkg install python_2.7.3-2_ar71xx.ipk
{% endhighlight %}
然后安装ssl库文件pyopenssl，python-openssl，libopenssl，可以参考goagent的wiki。记得将python的路径添加到/etc/profile
{% highlight sh %}
root@OpenWrt:~# python --version
Python 2.7.3
{% endhighlight %}

###运行goagent
将goagent源码下载到U盘,修改_proxy.ini,ip = 0.0.0.0_，执行python proxy.py,可能会出现缺少ssl模块，说明ssl库没有装全。现在goagent是不就能正常运行了呢,设置电脑的浏览器代理地址。看看是不是能访问YouTube了。如果goagent运行终止且报错
>OpenWrt user.info sysinit: python: md_rand.c: 322: ssleay_rand_add: Assertion `md_c[1] == md_count[1]' failed.'

看看这里[OpenWRT路由里面运行一段时间后自动退出了](https://code.google.com/p/goagent/issues/detail?id=2781#makechanges),#3楼的方案绝对可行，我也遇到了这个错误。使用时建议不要覆盖旧的库文件，使用 LD_LIBRARY_PATH 环境变量加载此版本的库。
{% highlight sh %}
root@OpenWrt:/etc# cat profile 
#!/bin/sh
[ -f /etc/banner  ] && cat /etc/banner

export PATH=/bin:/sbin:/usr/bin:/usr/sbin:/mnt/sdc1/opt/usr/bin
export LD_LIBRARY_PATH=/mnt/sdc1/opt/usr/lib/libssl-thread-safe
{% endhighlight %}
现在goagent是不是可以稳定运行了，我这运行了一整天一切正常。



