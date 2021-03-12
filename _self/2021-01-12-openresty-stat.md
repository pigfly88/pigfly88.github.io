---
layout: post
category: "mix"
title:  "使用OpenResty实现网站统计分析"
---

OpenResty 打包了 Nginx 核心，基于它开发人员可以使用 Lua 编程语言对 Nginx 核心以及现有的各种 Nginx C 模块进行脚本编程，构建出可以处理一万以上并发请求的极端高性能的 Web 应用。

OpenResty 致力于将你的服务器端应用完全运行于 Nginx 服务器中，充分利用 Nginx 的事件模型来进行非阻塞 I/O 通信。不仅仅是和 HTTP 客户端间的网络通信是非阻塞的，与MySQL、PostgreSQL、Memcached 以及 Redis 等众多远方后端之间的网络通信也是非阻塞的。

Lua 的设计目的是为了嵌入到应用程序中，从而为应用程序提供灵活的扩展和定制功能。Lua 脚本可以很容易的被 C/C++ 代码调用，也可以反过来调用 C/C++ 的函数。、

Nginx ("engine x") 是一个高性能的 HTTP 和反向代理服务器，由于 Nginx 使用基于事件驱动的架构，能够并发处理百万级别的 TCP 连接。

openresty nginx 配置：
```shell
http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    log_format  pv  '$remote_addr,$u_cookie_token,$u_page_url,$u_channel,$u_fmt_localtime,$u_from_url';
    access_log  logs/access.log  main;

    sendfile        on;
    keepalive_timeout  65;


    server {
        listen       80;
	    server_name  stat.dev;

        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

	location = /lua {
	    default_type 'text/html';
	    lua_code_cache off;
	    content_by_lua_file lua/test.lua;
	    add_header 'Access-Control-Allow-Origin' $http_origin;
	    add_header 'Access-Control-Allow-Credentials' 'true';
	    add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT,OPTIONS';
	    add_header Expires 'Fri, 01 Jan 1980 00:00:00 GMT';
	    add_header Pragma 'no-cache';
	    add_header Cache-Control 'no-cache, max-age=0, must-revalidate';
	    }

	location /i-log {
		internal;
			
		set_unescape_uri $u_page_url $arg_page_url;
		set_unescape_uri $u_cookie_token $arg_cookie_token;
		set_unescape_uri $u_channel $arg_channel;
		set_unescape_uri $u_fmt_localtime $arg_fmt_localtime ;
	    set_unescape_uri $u_from_url $arg_from_url;		
		log_subrequest on;
		access_log /usr/local/openresty/nginx/logs/pv.log pv;
			
		echo '';
	}
    }
}
```
		
### pv.log

```shell
[root@test11-bi-openresty ~]# tail /usr/local/openresty/nginx/logs/pv.log
10.5.2.18,b1aeccde4bc105809cfc182f784a6885,\x22https://c2b-test11.brightoilonline.com/bdh5std/oilcard.html^/oilrecharge?activityCode=c2b56q49h8emual58702\x22,c2b56q49h8emual58702,1610434214.16,\x22https://c2b-test11.brightoilonline.com/bdh5std/oilcard.html^/mine?backToUrl=/bdh5std/oilcard.html#/oilrecharge?activityCode=c2b56q49h8emual58702\x22
```

lua:

local socket = require "socket"
local t0 = socket.gettime()

-- check and set cookie
local cookie_token = ngx.var.cookie_cookie_token
--if( cookie_token) then
        --ngx.say("cookie_token:"..cookie_token)
--end
if( not cookie_token) then
        cookie_token = ngx.md5(ngx.now() .. ngx.var.remote_addr .. ngx.var.http_user_agent)
        --ngx.say("new cookie_token:"..cookie_token)
        ngx.header["Set-Cookie"] =  {'cookie_token=' .. cookie_token .. '; path=/;max-age=' .. 31536000}
end

local page_url = ngx.var.arg_page_url
--local fmt_localtime = ngx.localtime()
local fmt_localtime = socket.gettime()
ngx.location.capture('/i-log?' .. string.gsub(ngx.var.args,",","@") .. '&cookie_token=' .. cookie_token .. '&fmt_localtime=' .. fmt_localtime )



filebeat:

filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /usr/local/openresty/nginx/logs/pv.log
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false
setup.template.settings:
  index.number_of_shards: 3
setup.kibana:
setup.template.enabled: false
setup.template.name: "sdb"
setup.template.pattern: "sdb-*"
output.elasticsearch:
     hosts: ["10.5.11.21:9200"]
     index: "sdb"
	 
	 
	
	
	
es查询：

[root@test11-bi-openresty ~]# curl -X GET "10.5.11.21:9200/sdb/_search?pretty"
{
  "took" : 65,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 580595,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "sdb",
        "_type" : "doc",
        "_id" : "XQERe3MBY-U7e7ST3Slf",
        "_score" : 1.0,
        "_source" : {
          "@timestamp" : "2020-07-23T09:47:14.148Z",
          "beat" : {
            "name" : "bi-openresty",
            "hostname" : "bi-openresty",
            "version" : "6.4.0"
          },
          "message" : "10.10.1.2,50ba8ee43a6a758cc6a6e89200717241,\\x22https://c2b-test1.brightoilonline.com/bdh5std/oilcard.html^/mine?backToUrl=/bdh5std/gasstation.html#/gaslist?activityCode=c2b37i383304e1aa9180\\x22,c2b37i383304e1aa9180,1595497633.5877,",
          "source" : "/usr/local/openresty/nginx/logs/pv.log",
          "offset" : 18281,
          "prospector" : {
            "type" : "log"
          },
          "input" : {
            "type" : "log"
          },
          "host" : {
            "name" : "bi-openresty"
          }
        }
      },
	  ...
	  ...
	]
  }
}