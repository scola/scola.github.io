---
layout: post 
title: "以android为跳板使用移动网络登陆VPS"
category: 日志
tags: android SSH VPS
---
之前尝试在公司使用ssh登陆我的vps，发现公司在网关上封锁了ssh协议，所以不可能使用公司的网络来连接vps了,但是有点时候我的确需要登陆一下vps，比如重启某个程序。于是考虑使用手机的移动网络来连接vps，我使用的是android手机，于是找到了一个android应用[connectbot](http://code.google.com/p/connectbot/)，非常好用的android ssh客户端。但是我的手机键盘没有shift，esc这些键，导致操作非常不方便。于是找了下有没有键盘的android应用呢，找了一款[Hacker's Keyboard](https://play.google.com/store/apps/details?id=org.pocketworkstation.pckeyboard&hl=en)，很给力，完全模拟了电脑的键盘。vps的基本操作在手机上无障碍了。

但是想在手机上进行更加复杂的操作就不行了，比如文本编辑，毕竟手机的屏幕太小，触屏也很容易点错。于是想到用电脑ssh到手机，再ssh到vps，这样就可以通过电脑使用移动网络登陆vps了，万能的google帮我找到了[QuickSSHd](https://play.google.com/store/apps/details?id=com.teslacoilsw.quicksshd&hl=en)这个应用,使用方法在[这里](http://sumoudou.org/How%20to%20ssh%20into%20an%20Android%20phone%20over%20USB.html)

使用这个应用很容易就通过putty登陆到手机了，另外这个应用不需要root就可以使用哟，我使用的是amazon家的免费一年[EC2](http://aws.amazon.com/cn/ec2/)，需要使用一个.pem鉴权登陆，但是QuickSSHd这个应用不支持这种秘钥，于是需要秘钥转换，比如我之前使用putty登陆到这家的vps就将.pem转换成.ppk，但我折腾了很长时间也没有秘钥转换成dropbear支持的。

也许转换秘钥是可行的，却是不必要的，为什么不考虑密码登陆呢，于是在vps上配置了下，设置了用户和root的密码登陆。终于搞定了在电脑上以android为跳板登陆vps了，这篇blog就是用这种方式登陆vps写的，感叹一下，我的工作能力没咋提升，打酱油的功力更进一步。


