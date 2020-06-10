---
layout: post
title:  "Nginx 缓存配置一例"
date:   2020-06-10
categories: ['Nginx']
---

大家一般会在应用前面放台**Nginx**, 通过反向代理实现动静分离, 以此来减轻**应用**的压力.

在此记录下使用配置**Nginx**缓存碰到的问题.

一开始的时候想尽可能多的缓存, 所以就开了全部缓存, 
这里只展示规则部分(完整配置在[文末](#附)):

```
proxy_no_cache 0;
```


过了一段时间, 用户开始抱怨某些**api**访问总是获取不了最新的数据, 分析下来发现是应用返回的**header**中没有**cache-control: no-store**, 这样导致用户的api请求一直返回缓存的数据. 

最好的办法就是让开发把header修复下, 但是开发改程序也不是分分钟的事情.
作为运维, 可以利用灵活的配置来迅速解决问题, 所以就有了下面的缓兵之计:

添加缓存例外, 当**url**中含有**api**就不缓存:

```
set $nocache 0;
if ($request_uri ~ api) { set $nocache 1; }
proxy_no_cache $nocache;
```

这样的好处就是还是尽可能多的缓存, 但是随着用户的反馈, 一直在添加例外的路上... 这肯定不是办法.


看起来只有用笨方法了

使用下面命令分析了日志中访问最多的文件类型
```bash
awk '($9 ~ /200/)' access.log* | awk '{print $7}'|awk -F '/' '{print $NF}'|grep -v '^$'|grep -oP '\.([a-z]+)$'|sort -n|uniq -c|sort -k 1 -r|head -10
```
就有了下面的最终版本, 然后就可以慢慢等开发修复问题了.

```
set $nocache "1";
# 缓存指定文件类型
if ($request_uri ~* \.(exe|html)$ ) { set $nocache 0; }
# 缓存包含2个关键词的url
set $flag 0;
if ($request_uri ~* foo ) { set $flag "${flag}1"; }
if ($request_uri ~* bar ) {set $flag "${flag}2";}
if ($flag = "012") { set $nocache 0; }
proxy_no_cache $nocache;
}
```

#### 附

Nginx缓存完整配置

```
http {
    ***
    proxy_cache_path  /var/cache/nginx/proxy_cache/  levels=1:2    keys_zone=STATIC:100m inactive=168h  max_size=5g;
    add_header X-Cache-Status $upstream_cache_status;
    log_format rt_cache '$remote_addr - $upstream_cache_status [$time_local] '
                    '"$request" $status $body_bytes_sent $request_time '
                    '"$http_referer" "$http_user_agent"';
    access_log   /var/log/nginx/access.log rt_cache;
    ***
}

server {
    ***
    location / {
        ***
        set $nocache "1";
        # 缓存指定文件类型
        if ($request_uri ~* \.(exe|html)$ ) { set $nocache 0; }
        # 缓存包含2个关键词的url
        set $flag 0;
        if ($request_uri ~* foo ) { set $flag "${flag}1"; }
        if ($request_uri ~* bar ) {set $flag "${flag}2";}
        if ($flag = "012") { set $nocache 0; }
        proxy_no_cache $nocache;
        proxy_cache            STATIC;
        proxy_cache_valid      200  7d;
        proxy_cache_use_stale  error timeout invalid_header updating http_500 http_502 http_503 http_504;
        ***
    }
}
```
验证缓存
```
$ curl -I https://example.com
***
X-Cache-Status: HIT
***
```

#### 有用的**link**
- https://www.nginx.com/blog/nginx-caching-guide/
- https://www.maxcdn.com/one/tutorial/prevent-caching-specific-paths/