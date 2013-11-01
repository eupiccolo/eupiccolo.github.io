---
layout : post
category : mac os X
tags : [mac os X, 翻墙]
comments : no
---
本文写于2013-11-01，记录一下，免得每次都查。。。

[官方教程点这里](https://code.google.com/p/goagent/wiki/InstallGuide)

首先，从chrome 下载 [这货](https://chrome.google.com/webstore/detail/proxy-switchysharp/dpplabbmogkhghncfbfdeeokoefdjegm)

安装完毕，chrome右上角出现一个小地球图标

![icon]({{site.url}}/media/image/goagent/4.jpg)

进 [GAE](https://appengine.google.com/)登陆google账户，出现如下页面

![icon]({{site.url}}/media/image/goagent/1.jpg)

点Create Application，进入短信验证账户页面

![icon]({{site.url}}/media/image/goagent/2.jpg)

输验证码

![icon]({{site.url}}/media/image/goagent/3.jpg)

填写 Application Identifier 和 Application Title
其余默认，记得同意协议

![icon]({{site.url}}/media/image/goagent/5.jpg)

这样就注册成功了，出现如下页面

![icon]({{site.url}}/media/image/goagent/6.jpg)

点击左上角的 Google app engine 图标，查看自己的Application列表

![icon]({{site.url}}/media/image/goagent/7.jpg)

下载[goagent](https://code.google.com/p/goagent/)，如下图

![icon]({{site.url}}/media/image/goagent/8.jpg)

解压到一个自己觉得舒服的位置。。。。

![icon]({{site.url}}/media/image/goagent/9.jpg)

用终端进入goagent-XXXXXX/server 目录下， 执行命令 python uploader.zip

出现如下字样，提示你输入相关信息

![icon]({{site.url}}/media/image/goagent/10.jpg)

验证成功后，goagent服务端会自动部署到GAE上。

![icon]({{site.url}}/media/image/goagent/11.jpg)

下面我们修改一下 goagent-XXXXXX/local/proxy.ini ,把appid改成自己的

![icon]({{site.url}}/media/image/goagent/12.jpg)

然后设置SwitchySharp（就是一开始安装的那个小地球插件），点选项，然后点击 导入/导出 TAB，出现如下页面

![icon]({{site.url}}/media/image/goagent/13.jpg)

点击 从文件恢复 ，选择local文件夹下的 SwitchyOptions.bak  

![icon]({{site.url}}/media/image/goagent/14.jpg)

SwitchySharp 配置完毕

然后 用终端进入 goagent的 local文件夹，输入命令sudo python proxy.py，启动goagent客户端如图

![icon]({{site.url}}/media/image/goagent/15.jpg)

在浏览器中点击小地球图标，再点 Goagent，小地球变蓝后，就可以浏览被墙的网站了。。

![icon]({{site.url}}/media/image/goagent/16.jpg)


