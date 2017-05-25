# 配置Nginx SSL Node.js环境详细指南

Nginx是一款高性能的HTTP服务器，同时也是一款高效的反向代理服务器。不像传统的服务器，Nginx是基于事件驱动的异步架构，内存占用少但是性能很好。如果你的Web应用是基于Node.js的，那么建议你考虑使用Nginx来做反向代理，因为Nginx可以非常高效地提供静态文件服务。本文的主要内容是在不同的操作系统下配置Nginx和SSL，并且搭建一个Node.js运行环境。

## 安装Nginx

假设你已经在服务器上安装了Node.js，下面我们来安装Nginx。

**在Mac系统上安装Nginx**

利用`chown`命令来获取访问`/usr/local`文件夹的权限，命令代码如下：

> `sudo chown -R ‘username here’ /usr/local`

接下来的两行命令就可以安装Nginx了：

> `brew link pcre`
>
> `brew install nginx`



