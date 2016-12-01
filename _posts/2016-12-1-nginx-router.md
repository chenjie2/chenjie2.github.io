---
layout: post
title: 基于nginx的路由转发
categories: [nginx, kubernetes]
description: 基于nginx的路由转发
keywords: nginx, kubernetes, dns
---

基于nginx的路由转发

## Install
nginx install
./configure --prefix=/usr/local/nginx --with-debug --with-http_addition_module --with-http_dav_module --with-http_flv_module --with-http_gzip_static_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_stub_status_module --with-http_ssl_module --with-http_sub_module --with-http_xslt_module --with-ipv6 --add-module=../ngx_devel_kit-0.2.19 --add-module=../echo-nginx-module-0.53 --add-module=../lua-nginx-module-0.9.7 --add-module=../redis2-nginx-module-0.11 --add-module=../set-misc-nginx-module-0.24

## Config

```
#user  nobody;
#daemon off;
worker_processes  auto;
worker_rlimit_nofile 100000; 
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
    use epoll;
}


http {
    underscores_in_headers on;
    include       mime.types;
    default_type  application/octet-stream;
    include upstream.conf;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  0;
    #keepalive_timeout  65;

    #gzip  on;
    gzip on;
    gzip_min_length 1k;
    gzip_comp_level 2;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";

    server {
        listen       80;
        server_name  soma.yonyou.com 172.16.50.233 ~^(?<subdomain>.+)\.yysoma\.cn$ ~^(?<subdomain>.+)\.yysoma\.com$;
	rewrite ^(.*) https://$server_name$1 permanent;
     }

    server {
        listen       443;
        keepalive_timeout 0;
        client_max_body_size 2048m;
        server_name  soma.yonyou.com 172.16.50.233 ~^(?<subdomain>.+)\.yysoma\.cn$ ~^(?<subdomain>.+)\.yysoma\.com$;
    	ssl on;
    	ssl_certificate /usr/local/nginx/conf/server.crt;
    	ssl_certificate_key /usr/local/nginx/conf/server.key;

        location ~ .*/.(css|js|swf|php|htm|html)$ {
                add_header Cache-Control no-store;
        }
        location ^~ /app/kibana {
                proxy_set_header Host $host;
                proxy_set_header X-Real-Ip $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
                proxy_pass http://172.16.50.233:5601;
		proxy_redirect     off;
        }
        location ^~ /bundles {
                proxy_set_header Host $host;
                proxy_set_header X-Real-Ip $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
                proxy_pass http://172.16.50.233:5601;
		proxy_redirect     off;
        }
        location ^~ /elasticsearch {
                proxy_set_header Host $host;
                proxy_set_header X-Real-Ip $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
                proxy_pass http://172.16.50.233:5601;
		proxy_redirect     off;
        }
        location / {
#                proxy_set_header Host $host;
#                proxy_set_header X-Real-Ip $remote_addr;
#                proxy_set_header X-Forwarded-For $remote_addr;
                content_by_lua '
                        if ngx.var.subdomain == nil or ngx.var.subdomain == "soma" or  ngx.var.subdomain == "www" then
                                local uri = string.match(ngx.var.uri, "^/soma.*")
                                if uri == nil then
                                        ngx.exec("@root")
                                else
                                        ngx.exec("@console")
                                end
                        else
                                ngx.exec("@app")
                        end
                ';
                #proxy_pass http://$subdomain;
        }

        location @console {
                keepalive_timeout 0;
                proxy_set_header Host $host;
                proxy_set_header X-Real-Ip $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
		proxy_set_header X-Forwarded-Proto https;
                proxy_pass http://172.16.50.233:7681;
		#proxy_redirect     off;
		#proxy_redirect ~^[^:]+:// $scheme://;
		 proxy_redirect ~^[^:]+://[^/]+/(.*) $scheme://$host/$1;
                proxy_http_version 1.1;
#                proxy_set_header Connection "";
        }

        location @root {
                proxy_set_header Host $host;
                proxy_set_header X-Real-Ip $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
		proxy_set_header X-Forwarded-Proto https;
                proxy_pass http://20.10.93.220:8080;
		proxy_redirect     off;
        }

        location @app {
                proxy_set_header Host $host;
                proxy_set_header X-Real-Ip $remote_addr;
                proxy_set_header X-Forwarded-For $remote_addr;
		proxy_set_header X-Forwarded-Proto https;
                proxy_pass http://$subdomain;
		#proxy_redirect     off;
		#proxy_redirect ~^[^:]+://[^/]+/(.*) $scheme://$host$request_uri;
		proxy_redirect ~^[^:]+://[^/]+/(.*) $scheme://$host/$1;
        }
        location /echo {
                default_type 'text/plain';
                echo_before_body $subdomain;
                proxy_pass http://127.0.0.1:8081/upstream/$subdomain;
        }
        location /test {
                default_type 'text/plain';
                content_by_lua '
                        local uri = string.match(ngx.var.uri, "^/soma.*")
                        if uri == nil then
                                ngx.say(uri)
                        end
                ';
        }
     }

    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    server {
        listen 8081;
        location / {
            dyups_interface;
        }
    }
    server {
        listen       8000;
    #    listen       somename:8080;
        server_name  localhost;

        location / {
           default_type 'text/plain';
            echo "8000";
        }
    }
    server {
        listen       8001;
    #    listen       somename:8080;
        server_name  localhost;

        location / {
            default_type 'text/plain';
            echo "8001";
        }
    }

}
```
