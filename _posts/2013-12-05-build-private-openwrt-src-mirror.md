---
layout: post
title: "搭建一个openwrt的本地镜像"
category: 日志
tags: openwrt python flask
---
玩过openwrt的朋友估计都碰到过
>opkg_install_pkg: Package ddns-scripts md5sum mismatch. Either the opkg or the package index are corrupt. Try 'opkg update'.

这样的问题，一般使用trunk上的软件包安装可能会出现这个问题，trunk上的内核和软件更新最快，我们一般刷的固件都不是最新的，我买路由的时候店家已经刷好了openwrt的系统，好像是3.2.5的内核，我使用trunk上的源[http://downloads.openwrt.org/snapshots/trunk/ar71xx/packages/](http://downloads.openwrt.org/snapshots/trunk/ar71xx/packages/)安装软件一直出现上面的问题，走了不少弯路。后来把路由器弄坏了，逼着我有刷了最新trunk上的固件，终于不再出现上面内核与软件包不匹配的问题了，但是没过两天trunk上软件源又更新了，又出现不匹配的问题，搞得我又得刷最新的固件，都是没有经验啊，如果一直坚持使用[http://downloads.openwrt.org/backfire/10.03.1/ar71xx/packages/](http://downloads.openwrt.org/backfire/10.03.1/ar71xx/packages/)的固件和源就不会出现不匹配的问题了，因为这上面的源是稳定版，不会像trunk那样三天两头的更新。现在我已经刷了trunk上的固件，为了避免trunk上的源马上更新，造成上面的问题，我必须搭建一个本地镜像，这样还能提高安装软件的速度
###抓取所有ipk文件
手动下载所有的ipk文件肯定不可取，所以必须使用爬虫抓取，我在twitter上关注的大侠[@shell909090](https://twitter.com/shell909090)分享了一个抓取网页所有图片的脚本[推出一个自动下载图片的工具](https://twitter.com/shell909090/status/338976826217086977),非常强大，只需小改一下就可以用来抓所有的ipk文件，只是我家的网速太慢，一直抓不全，VPS上两分钟全部抓取，在通过sftp下载到本地，网速好的话可以直接本地抓取，我在公司电脑上试了下，也很快就全部抓取了

使用python可以很方便造一个一样的镜像，这里我使用了flask web框架
{% highlight python %}
from flask import Flask,send_from_directory
import os

UPLOAD_FOLDER = os.getcwd()

application = app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER


@app.route('/snapshots/trunk/ar71xx/packages/')
def packages():
    with open('packages.htm', 'r') as f:
        return f.read()

@app.route('/snapshots/trunk/ar71xx/packages/<filename>')
def uploaded_file(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'],
                               filename)


if __name__ == '__main__':
    app.run()
{% endhighlight %}
然后使用uwsgi部署一下
{% highlight sh %}
uwsgi --http :8080 --chdir /home/scola/openwrt/ --wsgi-file wsgi.py -p 4 --enable-threads
{% endhighlight %}
在本地打开[http://192.168.1.104:8080/snapshots/trunk/ar71xx/packages/](http://192.168.1.104:8080/snapshots/trunk/ar71xx/packages/)试试
![packages](/assets/openwrt.png)
修改openwrt的/etc/opkg.conf
{% highlight sh %}
root@OpenWrt:/etc# cat opkg.conf 
src/gz snapshot http://192.168.1.104:8080/snapshots/trunk/ar71xx/packages
{% endhighlight %}
这样就不再出现不匹配的错误了，还方便我以后重刷固件后安装软件，其实也可以搭建在VPS上分享给网友用
