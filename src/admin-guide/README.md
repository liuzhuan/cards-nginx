# 管理员指南

## 特性相关的配置文件

为了让配置文件易于维护，推荐将配置文件按功能特性拆分为不同文件，放入 `/etc/nginx/conf.d` 目录下，然后在主配置文件 `nginx.conf` 中，通过 `include` 指令引入。

```
include conf.d/http;
include conf.d/stream;
include conf.d/exchange-enhanced;
```

## 上下文

常见的上下文包括如下：

- `events` - 普通的连接处理
- `http` - HTTP 流量
- `mail` - 邮件流量
- `stream` - TCP 和 UDP 流量

位于这些上下文之外的指令，就位于主上下文（*main* context）之中。

### 继承

通常来讲，子域上下文会继承父级的配置参数。如果在多个层级有不同的参数，子域参数会覆盖父级参数。

## Web 服务器

```
http {
    server {
        listen      127.0.0.1:8080 default_server;
        server_name example.org www.example.org;
    }
}
```

## REF

- [Creating NGINX Plus and NGINX Configuration Files][config-files]
- [Configuring NGINX Plus as a Web Server][web-server]

[config-files]: https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/
[web-server]: https://docs.nginx.com/nginx/admin-guide/web-server/web-server/