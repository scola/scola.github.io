---
layout: post
title: "在openwrt路由器上部署代理，客户端零配置FQ"
category: 日志
tags: openwrt goagent shadowsocks dnsproxy chnroute gfw
---
之前在openwrt的路由器上安装了goagent，但是仍然需要浏览器设置代理，使用iptables来进行端口转发好像只能代理http的流量，对于https好像不行，我这个小白也不懂原理，不知道最新版是否可以，有空试试，于是想到用另一个神器[shadowsocks](http://shadowsocks.org/)试试，于是在[这里](http://buildbot.sinaapp.com)下载了已经编译好的ipk安装在路由器上，配置好shadowsocks的配置文件，所以要玩这个必须要知道怎么使用shadowsocks来凸墙，需要有自己的VPS，如果没有的话，[v2ex.com](https://v2ex.com/go/shadowsocks)有很多人共享自己的shadowsocks给网友用，具体怎么配置请参考[SHADOWSOCKS的wiki](https://github.com/clowwindy/shadowsocks/wiki-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E),怎么配置端口转发，请参考[https://github.com/madeye/shadowsocks-libev#advanced-usage](https://github.com/madeye/shadowsocks-libev#advanced-usage),我也会在本文末尾给出我所有的配置文件，我建议所有的这些最好先在linux系统的电脑上试验，然后在照搬到路由器上 
{% highlight sh %}
root@OpenWrt:/mnt/sdc1/Tools/shadowsocks# cat shadowsocks-goagent.sh 
#!/bin/sh
iptables -t nat -N SHADOWSOCKS
iptables -t nat -A SHADOWSOCKS -d YOUR-VPS-IP -j RETURN
iptables -t nat -A SHADOWSOCKS -d 0.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 10.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 127.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 169.254.0.0/16 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 172.16.0.0/12 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 192.168.0.0/16 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 224.0.0.0/4 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 240.0.0.0/4 -j RETURN
iptables -t nat -A SHADOWSOCKS -p tcp --dport 80 -j DNAT  --to-destination 192.168.1.1:8087
iptables -t nat -A SHADOWSOCKS -p tcp -j REDIRECT --to-ports 8089
iptables -t nat -A PREROUTING -p tcp -j SHADOWSOCKS
iptables -t nat -I PREROUTING -p udp  --dport 53 -j DNAT --to-destination 192.168.1.1:5353
iptables -t nat -A OUTPUT -p tcp -d 199.59.0.0/16 --dport 443 -j SHADOWSOCKS
ss-redir -c /etc/config/shadowsocks.json -f /var/run/shadowsocks.pid -v
{% endhighlight %}
为了防止dns污染，需要使用一个[dnsproxy](https://github.com/phuslu/dnsproxy),goagent作者的另一大作,将端口改成5353，代码中的53端口被openwrt的dnsmasq占用，然后将所有的dns请求转发到5353端口，这个东东还有dns缓存功能，相当好用，第一次dns解析需要100ms多，比直接解析稍慢一些，为了凸墙，我认了
{% highlight sh %}
python dnsproxy.py
iptables -t nat -I PREROUTING -p udp  --dport 53 -j DNAT --to-destination 192.168.1.1:5353
{% endhighlight %}
从上面的配置可以看出，我使用goagent代理所有的80端口的http请求，这样做是为了省流量，GAE的流量是免费了，随便用，尤其是看[YouTube](http://www.youtube.com/)的视频，非常费流量，部署shadowsocks的VPS可是花钱的,所以VPS的流量还是省着用  
###国内网站无需代理
如果将所有的网站都走代理，一是非常耗流量，同时走代理速度慢，二是youku这些网站限制国外IP，搞得我老婆看不了youku上的电视剧，我的麻烦就大了，所以必须跳过国内路由，我使用的是[bestroutetb](https://github.com/ashi009/bestroutetb/wiki/%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E),这个是作者优化的路由表，我只用了其中的700多条，选择国内直连,本文结尾会附上配置文件
{% highlight sh %}
iptables -t nat -A SHADOWSOCKS -d chn-ip  -j RETURN
{% endhighlight %}
###小结
最后就是将所有的配置进行开机执行,怎么设置开机执行某程序，请参考我上一篇博客[在openwrt-tplink703n路由器上成功部署GoAgent](http://scola.github.io/deploy-goagent-on-openwrt-tplink-703n/)，对连上这个wifi的人来说是傻瓜式翻墙，或者根本感觉不到墙的存在，安装和配置的过程中可能遇到很多问题，基本都可以用google帮忙解决，搜中文找不到答案就试试英文，实在不行就上论坛或者stackoverflow上求大神帮帮忙,最后谢谢我用到的开源项目代码的作者和一些blogger
###分享我的配置文件
使用goagent和shadowsocks代理国外的流量的配置文件[https://www.dropbox.com/s/mj3cc6ucv4cxkwq/openwrt-fuckgfw.sh](https://www.dropbox.com/s/mj3cc6ucv4cxkwq/openwrt-fuckgfw.sh)
