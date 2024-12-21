---
title: Nginx报错
description: Nginx报错
date: 2024-03-01
slug: Nginx报错
image: 202412212222986.png
categories:
    - Nginx
---

# Nginx
> 参考[博客](https://www.cnblogs.com/dotnet261010/p/12596185.html)
前端正常显示，但是请求404
比如`http://192.168.56.10/api/user/list`报404，后端请求`http://192.168.56.10:8080/user/list`正常显示
**出现问题的nginx配置如下**
```conf
 
events { 
    worker_connections  1024; 
} 
 
 
http { 
    include       /etc/nginx/mime.types; 
    default_type  application/octet-stream; 
 
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" ' 
                      '$status $body_bytes_sent "$http_referer" ' 
                      '"$http_user_agent" "$http_x_forwarded_for"'; 
 
    access_log  /var/log/nginx/access.log  main; 
 
    sendfile        on; 
    #tcp_nopush     on; 
 
    keepalive_timeout  65; 
    #gzip  on;
    include /etc/nginx/conf.d/*.conf;
    server {
        listen       80;
        server_name  localhost;
        location ^~/api/{
            proxy_pass http://192.168.56.10:8080/;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
`include /etc/nginx/conf.d/*.conf;`表示使用的是conf.d下的`default.conf`文件
将`/etc/nginx/nginx.conf`文件改成
```conf
 
events { 
    worker_connections  1024; 
} 
 
 
http { 
    include       /etc/nginx/mime.types; 
    default_type  application/octet-stream; 
 
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" ' 
                      '$status $body_bytes_sent "$http_referer" ' 
                      '"$http_user_agent" "$http_x_forwarded_for"'; 
 
    access_log  /var/log/nginx/access.log  main; 
 
    sendfile        on; 
    #tcp_nopush     on; 
 
    keepalive_timeout  65; 
    #gzip  on;
    include /etc/nginx/conf.d/*.conf;
}
```
将`/etc/nginx/conf.d/default.conf`文件改成
```conf
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    location ^~/api/{
        proxy_pass http://192.168.56.10:8080/;
    }
    #error_page  404              /404.html;
    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}
    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```
主要是在`/etc/nginx/conf.d/default.conf`文件里添加这一段代码
```conf
    location ^~/api/{
        proxy_pass http://192.168.56.10:8080/;
    }
```
**扩展**
```
    location ^~/api/{
        proxy_pass http://192.168.56.10:8080;
    }
```
`http://192.168.56.10/api/user/list`结果为`http://192.168.56.10:8080/api/user/list`
```
    location ^~/api/{
        proxy_pass http://192.168.56.10:8080/;
    }
```
`http://192.168.56.10/api/user/list`结果为`http://192.168.56.10:8080/user/list`
```
    location ^~/api/{
        proxy_pass http://192.168.56.10:8080/test;
    }
```
`http://192.168.56.10/api/user/list`结果为`http://192.168.56.10:8080/test/user/list`
