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

Nginx安装完毕后，你可以用下面的命令来启动Nginx

> `sudo nginx`

最后你可以在目录`/usr/local/etc/nginx/nginx.conf`下看到Nginx的配置文件。

**在Ubuntu上安装Nginx**

如果你使用Ubuntu，那么可以用以下方式安装Nginx：

> `sudo apt-get update`
>
> `sudo apt-get install nginx`

Nginx安装完成后便可自动启动。

**在Windows下安装Nginx**

windows版本的nginx可以在这里下载，接下来将安装包解压放到指定目录下，在cmd命令工具下运行以下代码：

> `unzip nginx-1.3.13.zip`
>
> `cd nginx-1.3.13`
>
> `start nginx`

同样，`start nginx`命令会让nginx启动完成。

现在我们已经安装完Ngnix，接下来该配置服务器了。

## 配置Node.js服务器

首先我们来创建一个简单的Node.js服务器，你可以在这里下载Express版本的Node.js。下载源代码后，将其解压至demoApp文件夹下，并且输入以下命令让服务器在3000端口上启动。

> `npm install`

> `node bin/www`

## 配置nginx

在MAC上，可以直接使用nano进行安装

> `nano /usr/local/etc/nginx/nginx.conf`



