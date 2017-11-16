# nginx+uwsgi+django的Https通信

**写在前面：**

```
由于苹果商店（App Store），从2017年开始，要求APP的HTTP通信，必须转为HTTPS，所以，我们需要把原来的架构（nginx+uwsgi+django），变为走https的。
```

**方法：**

**■ 方法一（通过Nginx 实现**）：

**可以去腾讯云申请免费SSL证书。请参考我另一篇文章：**

[**《腾讯云服务器申请SSL证书, 配置Nginx, 实现HTTPS 》**](http://blog.csdn.net/chenggong2dm/article/details/61925938)

**    
**

**下面是如何自己创建私有SSL证书（不太推荐，仅用于测试）    
**

1，创建一个目录，保存证书和私钥。（也可以放到其他目录下）

mkdir /home/key\_dir

进入这个目录：

cd /home/key\_dir

2，创建服务器私钥: 长度1024位, des3加密算法的. （之后输入一个口令，需要记住）

openssl genrsa -des3 -out server.key 1024

3，创建签名请求的证书CSR。（之后会要求你输入国别，比如CN，省份，城市，公司等等一系列信息）

openssl req -new -key server.key -out server.csr

4，在加载SSL支持的[Nginx](http://www.centos.bz/category/web-server/nginx/)并使用上述私钥时除去必须的口令：

cp server.key server.key.org

openssl rsa -in server.key.org -out server.key

5，标记证书使用上述私钥和CSR

openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

6，修改nginx.conf （其实就是在原配置的基础上，把监听的80端口改为监听443，然后加入密钥相关配置），例如：

```
server {
        listen       443;
        server_name  xbreak;

        charset utf-8;

        ssl on;
        ssl_certificate /home/key_dir/server.crt;
        ssl_certificate_key /home/key_dir/server.key;
        #ssl_session_timeout 5m;

        location / {
            include uwsgi_params;
            uwsgi_pass 127.0.0.1:19808;
        }

        # error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

重复一遍，需要更改的就是：

```
    listen       443;  
    ssl on;  
    ssl\_certificate /home/key\_dir/server.crt;  
    ssl\_certificate\_key /home/key\_dir/server.key;
```

7，重启 Nginx  （由于uwsgi 没有任何更改，不需要重启！）

8，我们来使用https进行访问，发现是可以的！



**  
■ 方法二（可以去掉Nginx，直接用uwsgi解决**）：

具体怎么做，uwsgi官网已经给出了。就是uwsgi 的https  


  


  


■ 本文参考：

nginx使用ssl模块配置HTTPS支持  


http://www.cnblogs.com/yanghuahui/archive/2012/06/25/2561568.htm

