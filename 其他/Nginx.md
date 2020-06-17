# Nginx
## nginx在应用程序中的作用
* 解决跨域
* 请求过滤
* 配置gzip
* 负载均衡
* 静态资源服务器
* …

### 正向代理
**正向代理**是为我们服务的，即为客户端服务的，客户端可以根据正向代理访问到它本身无法访问到的服务器资源。
**正向代理**对我们是透明的，对服务端是非透明的，即服务端并不知道自己收到的是来自代理的访问还是来自真实客户端的访问

### 反向代理

**反向代理**是为服务端服务的，反向代理可以帮助服务器接收来自客户端的请求，帮助服务器做请求转发，负载均衡等。
**反向代理**对服务端是透明的，对我们是非透明的，即我们并不知道自己访问的是代理服务器，而服务器知道反向代理在为他服务。


## 基本配置
![](https://user-gold-cdn.xitu.io/2019/3/11/1696a118b4910728?imageView2/0/w/1280/h/960/ignore-error/1)

```
events { 

}

http 
{
    server
    { 
        location path
        {
            ...
        }
        location path
        {
            ...
        }
     }

    server
    {
        ...
    }

}


```

main:nginx的全局配置，对全局生效。
Events:配置影响nginx服务器或与用户的网络连接。
http：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。
Server：配置虚拟主机的相关参数，一个http中可以有多个server。
Location：配置请求的路由，以及各种页面的处理情况。
Upstream：配置后端服务器具体地址，负载均衡配置不可或缺的部分。

## nginx如何实现负载均衡
```
upstream balanceServer {
    server 10.1.22.33:12345;
    server 10.1.22.34:12345;
    server 10.1.22.35:12345;
}

    server {
        server_name  fe.server.com;
        listen 80;
        location /api {
            proxy_pass http://balanceServer;
        }
    }



```


## nginx解决跨域的原理
```
server {
        listen       80;
        server_name  fe.server.com;
        location / {
                proxy_pass dev.server.com;
        }
}

```

## 静态资源服务器
```
location ~* \.(png|gif|jpg|jpeg)$ {
    root    /root/static/;  
    autoindex on;
    access_log  off;
    expires     10h;# 设置过期时间为10小时          
}

```


## 参考
 * [前端开发者必备的Nginx知识 - 掘金](https://juejin.im/post/5c85a64d6fb9a04a0e2e038c#heading-11)