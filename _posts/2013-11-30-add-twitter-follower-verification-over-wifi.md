---
layout: post
title: "在路由器上实现Twitter好友认证"
category: 日志
tags: openwrt twitter python gfw
---
上一周发了一条tweet[我家wi-fi无密码，但是我给加上了twitter认证，期待邻居变推友](https://twitter.com/wushaozheng/status/404053137897447424/photo/1),引起了很多推友的兴趣，很多人询问方案，很抱歉不能一一作出回复，因为我也是个小白，经常在twitter上@大侠请教问题，期待回复。其实这个想法是我在用[fqrouter](https://s3-ap-southeast-1.amazonaws.com/fqrouter/fqrouter-latest.apk)这款android神器的时候想到的，并给作者发了邮件和[v2ex.com](http://v2ex.com/t/78718#reply20)上讨论了下，目前我的水平还不能给作者贡献代码，所以买了个刷好openwrt的路由器玩了下 
###限制客户端
最初的想法是将所有连接强制重定向到认证页面，除了twitter的相关页面，也就是除了twitter其他页面都跳转到认证页面，后来想想twitter的IP不止一个，做这么精细的控制反而麻烦，因为twitter的所有网站均是https的，所以我只将所有的http的80端口重定向到认证页面，其他端口不管，twitter，facebook，gmail的https 443端口自然是可以访问的，这样就限制了90%以上的网站，另外腾讯QQ客户端是可以上的哟，QQ是走的腾讯的私有协议，不是http，但是你上不了baidu和youku
{% highlight sh %}
iptables -t nat -I PREROUTING -s 192.168.1.237 -p tcp --dport 80 -j DNAT  --to-destination 192.168.1.1:8888  #限制
iptables -t nat -D PREROUTING -s 192.168.1.237 -p tcp --dport 80 -j DNAT  --to-destination 192.168.1.1:8888  #解除限制
{% endhighlight %}
其实懂点网络的朋友可以使用代理或者VPN跳过限制，根本不需要登录验证，但是直接认证是非常简单的，还帮你翻了墙
###认证推友
使用twitter的api[https://dev.twitter.com/docs/api/1.1/get/friendships/lookup](https://dev.twitter.com/docs/api/1.1/get/friendships/lookup)来查询两个twitter用户的好友关系，关于怎么使用twitter api，我使用了[这篇blog](thomassileo.com/blog/2013/01/25/using-twitter-rest-api-v1-dot-1-with-python/)中的代码，通过用户输入的twitter用户名来匹配twitter api返回的twitter friendship，如果是friend，解除重定向限制，如果不是，给出提示，用户可以从认证页面点击[**立即关注**](https://twitter.com/wushaozheng)跳转到你的twitter页面，从而用户可以完成注册，登录，关注等。所以你必须先配置好路由器使客户端零配置翻墙，可以参考我的上一篇[在openwrt路由器上部署代理，客户端零配置FQ](http://scola.github.io/deploy-proxy-on-openwrt--client-need-not-to-set/),如果客户端访问不了twitter页面，所有的这一切都是毫无意义的，另外api.twitter.com也是被墙的，所以将其加入hosts防止dns污染，还可加快解析速度
{% highlight sh %}
root@OpenWrt:/etc# cat hosts 
127.0.0.1 localhost
199.59.149.232 api.twitter.com
199.59.150.41 api.twitter.com
199.59.148.87 api.twitter.com
{% endhighlight %}
另外还需要将api.twitter.com的ip加入shadowsocks的代理转发规则
{% highlight sh %}
iptables -t nat -A OUTPUT -p tcp -d 199.59.0.0/16 --dport 443 -j SHADOWSOCKS
{% endhighlight %}
###认证页面和扫描客户端
这部分算是最简单的了，可以直接参考我的代码[https://github.com/scola/twittrouter](https://github.com/scola/twittrouter),前端代码我参考了[这里](https://www.ibm.com/developerworks/community/blogs/bobleah/entry/code_example_of_responsive_web_design_using_css3_media_queries13?lang=en),使用一个线程循环扫描客户端，不知道有新的客户端连接，openwrt会不会收到通知，在[论坛](https://forum.openwrt.org/viewtopic.php?id=47326)问了下，也没有人回复，所以只能循环扫描了，代码中有很多hard code，不具通用性，仍需完善，接下来改善代码，增加配置文件通用化


