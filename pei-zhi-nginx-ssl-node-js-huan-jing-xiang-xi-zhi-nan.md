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
>
> `node bin/www`

## 配置nginx

在MAC上，可以直接使用nano进行安装

> `nano /usr/local/etc/nginx/nginx.conf`

如果需要，你也可以直接到文件夹下用你最喜欢的

文本编辑器

打开config文件，你可以在文件中找到`server`配置节点，类似下面的代码：

```
server {
listen       8080;
server_name  localhost;
 
….
more config goes here
}
```

接下来我们将对

`server`

 节点作一些符合自己需求的配置，我们需要把网站中的静态文件请求交给Nginx处理，其他的文件请求则交给Node.js后端服务器。我们将用下面的代码替换上面的`server`配置节点：

```
server {
listen       8080;
server_name  localhost;
 
location / {
proxy_pass http://localhost:3000;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection ‘upgrade’;
proxy_set_header Host $host;
proxy_cache_bypass $http_upgrade;
}
 
location /public {
root /usr/local/var/www;
}
 
}

```

这样我们就让Nginx在`http://localhost:8080`. 上监听了。`location /` 配置节点将告诉Nginx接收任意的请求，`location` 配置节点中我们用`proxy_pass` 指定了Node.js的后端服务器为`http://localhost:3000` 。

现在我们要用另外一个配置节点`location /public` 来告诉Nginx处理静态文件请求，其中内部的`location` 节点设置了根目录为`/usr/local/var/www`. 当然你也可以换成其他的目录。如此一来，当有类似这样的请求`http://localhost:8080/public/somepath/file.html` ，Nginx都会从`/usr/local/var/www/public/somepath/file.html`读取静态文件。

修改完配置文件后，你需要用下面的代码来重启Nginx：

**Mac**

```
sudo nginx -s stop &amp;&amp; sudo nginx
```

**Ubuntu：**

```
sudo service nginx restart
```

或者

```
sudo /etc/init.d/nginx restart
```

**Windows：**

```
nginx -s reload
```

接下来我们来用Nginx来代替Node.js提供CSS样式文件，Node.js模板用的是

`/public/stylesheets/style.css`下面的文件。在`/usr/local/var/www/public/stylesheets`文件夹下创建一个名为

`style.css`的文件，Nginx将会正确地解析到它。比如你可以在CSS文件中写入以下代码：

```
body {
padding: 50px;
font: 14px “Lucida Grande”, Helvetica, Arial, sans-serif;
}
 
a {
color: #00B7FF;
}
```

然后你可以登录`http://localhost:8080`下来看自己的web应用，你会发现尽管是访问Nginx服务器，但是请求都是通过真实的Node.js后端服务器处理的，只有CSS静态文件由Nginx处理。

## 创建SSL

网站产品做多了，你会发现需要创建SSL来保护敏感的信息。可能你第一反应会想到从证书颁发机构申请网站证书，但是你也可以创建签名证书。唯一的问题就是浏览器端会提示“该证书不可信”的警告信息，但是作为本地测试，这也就足够了。这里有一篇教程讲解了如何自己创建签名SSL证书，可以看看。

当你有了自己的证书，你就可以在Nginx上安装SSL了，修改后的配置文件，代码如下：

```
server {
listen       8080;
listen       443 ssl;
server_name  localhost;
 
ssl_certificate  /etc/nginx/ssl/server.crt
ssl_certificate_key /etc/nginx/ssl/server.key
 
location / {
proxy_pass http://localhost:3000;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection ‘upgrade’;
proxy_set_header Host $host;
proxy_cache_bypass $http_upgrade;
}
 
location /public {
root /usr/local/var/www;
}
 
}
```

完成了！这样当你访问`https://localhost:8080` 的时候SSL就可以开始工作了。这里我们默认将证书保存在`/etc/nginx/ssl/server.crt` 目录下。将私钥保存在`/etc/nginx/ssl/server.key` 目录下，你也可以改变保存的目录。

## 总结

本文中我们学到了如何用Nginx为Node.js做反向代理，并且配置SSL。由Nginx在前端处理静态文件请求，这可以为Node.js后端服务器大大减轻压力。自己尝试一下吧，有什么问题可以在评论中交流。





