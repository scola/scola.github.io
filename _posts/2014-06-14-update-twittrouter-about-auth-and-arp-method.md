---
layout: post
title: "更新twittrouter，增加授权模式和去除对net-tools-arp的依赖"
category: 日志
tags: openwrt twittrouter gfw shadowsocks
---
关于[twittrouter](https://github.com/scola/twittrouter)这个小应用，只需看一下我之前发的的一条[twitter状态](https://twitter.com/wushaozheng/status/404053137897447424/photo/1),就会明白基本功能，但那时这个小玩具是用python实现的，在openwrt的路由器上安装python解释器比较占存储，4M flash的路由器根本装不下，8M的都有点悬，我当时在路由器装python完全是为了跑goagent，前几个月用C 重新实现了这个功能，编译后仅30K，实用性提高不少，上周进一步优化，增加了授权模式和去除了对net-tools-arp的依赖
###下载安装
[点此下载](http://crater.herokuapp.com/uploads/twittrouter_0.1.2-1_ar71xx.ipk),上传路由器即可安装
{% highlight sh %}
opkg update
opkg install http://crater.herokuapp.com/uploads/twittrouter_0.1.2-1_ar71xx.ipk
{% endhighlight %}
###使用方法
{% highlight sh %}
twittrouter -h   #查看使用方法 
{% endhighlight %}
因为twitter api被墙，需要使用shadowsocks或者VPN等网络工具，以实现无论在路由器内部还是连接路由器的设备无需任何设置即可翻墙，可以使用这个自动配置脚本[pdnsd-ss-iptables.sh](https://github.com/scola/twittrouter/blob/master/config/pdnsd-ss-iptables.sh)，此脚本已打包在twittrouter里，你可以在/etc/config/下找到,如果你碰巧使用TP-Link 703n这款路由器，也可使用[编译一个tplink-703n自动凸墙的openwrt固件](http://scola.github.io/build-openwrt-firmware-within-shadowsocks-and-twittrouter/),配置好网络和简单测试是否OK
{% highlight sh %}
twittrouter -u kfc    #测试网络及授权
{% endhighlight %}
如果测试OK的话
{% highlight sh %}
twittrouter -a   #授权自己twitter帐号
{% endhighlight %}
授权很简单，按提示操作即可，也可以跳过授权快速体验此应用，因为已经默认配置好了[@twitrouter](https://twitter.com/twitrouter)的授权
{% highlight sh %}
twittrouter    #运行此应用
/etc/init.d/twittrouter enable    #加入开机启动
{% endhighlight %}
###android版本
[这里](http://crater.herokuapp.com/uploads/twittrouter.apk)有一个同样功能的android应用，同样是个玩具，没啥实用性，[代码](https://github.com/scola/twittrouter-android)是从fqrouter fork过来的，需要root权限才能运行
