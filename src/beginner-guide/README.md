# 新手入门

nginx 有一个 master 进程和多个 worker 进程。master 进程主要用于读取和解析配置文档和维护 worker 进程。worker 进程进行实际的网络请求处理。

nginx 的配置文件位于 `/usr/local/nginx/conf`，`/etc/nginx` 或 `/usr/local/etc/nginx`（mac 系统）。

## 启动、停止和重载配置

nginx 启动后，就可以使用以下命令控制：

```sh
nginx -s signal
```

其中的 `signal` 可以是以下参数之一：

- `stop`   快速关闭
- `quit`   优雅关闭
- `reload` 重新加载配置文件
- `reopen` 重新打开日志文件

比如，假如要在 worker 进程处理完当前请求之后，关闭 nginx 进程，可以使用：

```sh
nginx -s quit
```

对配置文件的改动，经过重新读取才能生效：

```sh
nginx -s reload
```

nginx master 进程的 ID 默认写入 `/usr/local/var/run/nginx/`（通过 Homebrew 安装）, `/usr/local/nginx/logs/` 或 `/var/run/` 的 `nginx.pid`。可以通过 Unix 实用工具 `kill` 优雅关闭它：

```sh
# 假如 
kill -s QUIT 1628
```

要获取所有正在运行的 nginx 进程，可以使用 `ps` 实用工具：

```
# -a 显示自己和他人的进程
# -x 显示没有控制终端的进程
$ ps -ax | grep nginx
```

## 配置文件的结构

nginx 由模块组成，模块行为受配置文件的指令控制。指令分为简单指令和块指令。简单指令由名称和值构成，中间空格分开，结尾使用分号 `;`。块指令结尾是一个 `{}`，里面包含一些指令。如果块指令可以在括号内包含其他指令，它就称为 `context` 上下文。

在所有 `context` 之外的指令，位于 `main` context。`events` 和 `http` 就存在于 `main` context，`server` 在 `http` 中，`location` 在 `server` 中。

`#` 开始的行是注释行。

## 托管静态内容

首先，创建 `/data/www` 目录，并放置一个 `index.html` 文件；创建 `/data/image` 目录，防止一些图片。

然后，打开配置文件，增加一个新的 `server` 块：

```
http {
    server {
        location / {
            root /data/www;
        }

        location /images/ {
            root /data;
        }
    }
}
```

通常，配置文件可以包含多个 `server` 块，通过它们监听端口和 `server names` 作区分。

## 设立一个简单代理服务器

假如我们要设定一个简单代理服务器，图像从本机读取，其他请求代理到其他服务器。那么，可以新增一个 `server` 块：

```
server {
    listen 8080;
    root /data/up1;

    location / {
    }
}
```

注意，上面的 `root` 位于 `server` 内。当匹配的 `location` 中不包含 `root` 指令时，会使用该 `root` 值。之后，`http://localhost:8080` 就可以作为上游服务器（即被代理的服务器）。

然后，使用上一个板块的服务器设置，将其修改为代理服务器。在第一个 `location` 中，增加 `proxy_pass` 指令：

```
server {
    location / {
        proxy_pass http://localhost:8080;
    }

    location /images/ {
        root /data;
    }
}
```

我们接下来修改第二个 `location` 块，让它只接受特定的图像类型：

```
location ~ \.(gif|jpg|png)$ {
    root /data/images;
}
```

这个参数是一个正则表达式，会匹配所有以 `.gif`, `.jpg` 或 `.png` 结尾的 URI 地址。正则表达式需要以 `~` 开始。对应的请求会映射到 `/data/images` 目录。

最终的代理服务器配置如下所示：

```
server {
    location / {
        proxy_pass http://localhost:8080/;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

## 设定 FastCGI 代理

nginx 可以把请求路由转发到 FastCGI 服务器。

最简单的 nginx 设置可以使用 `fastcgi_pass` 指令代替 `proxy_pass`。使用 `fastcgi_param` 指令设定参数传递到 FastCGI 服务器。

（未完待续。。。）

## REF

- [Beginner's Guide][guide]
- [How nginx processes a request?][request_processing]
- [Server names][server_names]

[guide]: http://nginx.org/en/docs/beginners_guide.html
[request_processing]: http://nginx.org/en/docs/http/request_processing.html
[server_names]: http://nginx.org/en/docs/http/server_names.html