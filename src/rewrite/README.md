# 改写地址

设置 nginx 配置文件。比如，将 `www.example.com/project/index/1234-abcd` 改写为 `www.example.com/project/index.html?id=1234-abcd`。

```
server {
    server_name: www.example.com;
    location /project/index {
        try_files $uri @rewrite;
    }
    
    location @rewrite {
        rewrite ^/project/index/(.*) /index.html?id=$1;
    }
}
```