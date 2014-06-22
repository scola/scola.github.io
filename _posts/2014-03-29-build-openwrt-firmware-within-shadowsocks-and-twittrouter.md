---
layout: post
title: "编译一个tplink-703n自动凸墙的openwrt固件"
category: 日志
tags: openwrt shadowsocks twittrouter gfw
---
编译了一个只需修改一下宽带帐号就可以自动翻墙的openwrt固件，客户端零配置，固件中集成了[shadowsocks-libev](https://github.com/madeye/shadowsocks-libev)和一个我自用的shadowsocks帐号，还有pdnsd可防dns污染和一个自动配置脚本[pdnsd-ss-iptables.sh](https://github.com/scola/twittrouter/blob/master/config/pdnsd-ss-iptables.sh)，所有的一切都配置OK,并且开机自启动
###使用方法
{% highlight sh %}
cp /etc/config/network /etc/config/network.bak    #备份一下network
cp /etc/config/network-pppoe /etc/config/network
{% endhighlight %}
然后修改一下你的宽带帐号，你也可以按[network-pppoe](https://github.com/scola/twittrouter/blob/master/config/network-pppoe)直接修改network配置拨号连接，修改前最好也备份下，以备出错还原
###集成twitter好友认证
固件中集成了[twittrouter](https://github.com/scola/twittrouter)这个应用，默认是不会开机自启动的，设置开机自启动,去掉/etc/init.d/twittrouter注释的"#",即可开机自启动
{% highlight sh %}
#/usr/bin/twittrouter -f /var/run/twittrouter.pid 
{% endhighlight %}
之前发过一个[帖子](https://www.v2ex.com/t/104237#reply18)详细介绍过这个功能，真心想体验这个功能的朋友在确认自己联网和凸墙OK后可以先测试一下
{% highlight sh %}
twittrouter -u kfc
{% endhighlight %}
Congratulations! Verify success,kfc is your twitter friend  

/etc/config/twittrouter.json 已经配置好了我自己的一个新注册的twitter帐号，按[README](https://github.com/scola/twittrouter/blob/master/README.md)配置成你自己的，另外这个项目仍不完善，诚邀懂web前端的朋友帮助，有热心的朋友可通过博客的关于页面的信息联系我
###对比官方固件
官方固件的wifi开关默认关闭的，而且无web配置页面，我这个wifi开关已打开，而且web配置页面也是最新的bootstrap的主题，这个固件原厂的tp-link 703n即可刷，刷后还剩200多K的可用空间
###免责声明
此固件不作恶，所有代码都是开源的，集成的shadowsocks某一天到期后会导致你无法连接国外网站，所以请及时更换自己的帐号，使用此固件出现任何问题可更新官方的固件解决，本人不负担责任
###下载地址
[https://www.dropbox.com/s/yiayc01wufu6c4x/tplink-703-cross-GFW.zip](https://www.dropbox.com/s/yiayc01wufu6c4x/tplink-703-cross-GFW.zip),或者[http://crater.herokuapp.com/uploads/tplink-703-cross-GFW.zip](http://crater.herokuapp.com/uploads/tplink-703-cross-GFW.zip)
刷机前请做好校验
{% highlight sh %}
md5sum -c md5sums 2> /dev/null | grep OK
{% endhighlight %}
另外最好不要在failsafe模式下更新固件，我不确定是否可以，但是我在failsafe模式更新固件变过砖，还寄到[这家taobao店http://jjwifi.taobao.com](http://jjwifi.taobao.com/)让店家给修砖，在此给他做个广告，新玩openwrt的话建议买他家升级后的路由，原厂的703n存储太小，玩起来捉襟见肘
