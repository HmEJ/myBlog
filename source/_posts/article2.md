---
title: nginx实现正向代理
date: 2024-06-25 14:41:02
categories:
  - 运维
tags:
  - nginx
  - 正向代理
---

本文介绍如何通过nignx实现正向代理。

<!-- more -->

nginx正向代理需要借助一个模块： `ngx_http_proxy_connect_module`

该模块的[源码地址](https://github.com/chobits/ngx_http_proxy_connect_module)

我们需要先下载这个源码，然后通过编译的方式将其安装到nginx里。nginx安装方式有很多，比如通过apt直接安装，或者使用docker等。本文使用的nginx安装方式是源码编译安装。

## 步骤

1. 下载ngx_http_proxy_connect_module模块。

   直接clone其github仓库即可。

   ```sh
   git clone https://github.com/chobits/ngx_http_proxy_connect_module.git
   ```

2. 下载nginx （v1.24.0）

   ```sh
   wget https://nginx.org/download/nginx-1.24.0.tar.gz
   # 解压
   tar -zxvf nginx-1.24.0.tar.gz
   ```

   > 安装nginx前，你可能需要一些前置的准备：
   >
   > - 安装gcc 
   >
   >   ```sh
   >   apt install build-essential
   >   ```
   >
   > - 安装pcre （正则库）
   >
   >   ```sh
   >   apt-get install libpcre3 libpcre3-dev
   >   ```
   >
   > - openssl
   >
   >   ```sh
   >   apt-get install openssl libssl-dev
   >   ```

3. 打补丁

   ```sh
   cd /path/to/nginx-1.24.0 #进入源码根目录
   # 打补丁
   patch -p1 < /path/to/ngx_http_proxy_connect_module/patch/proxy_connect_rewrite_102101.patch
   ```

   > 这里我使用nginx1.24.0，需要的补丁是 proxy_connect_rewrite_102101.patch。
   >
   > 如果你使用其他nginx版本，可以在ngx_http_proxy_connect_module模块的github仓库[查看对应的补丁](https://github.com/chobits/ngx_http_proxy_connect_module?tab=readme-ov-file#select-patch)。只需要将上述 ‘proxy_connect_rewrite_102101.patch’ 替换为对应版本的补丁即可。

4. 编译nginx源码，安装nginx

   ```sh
   # 进入源码目录
   cd /path/to/nginx-1.24.0
   # 编译配置项
   ./configure --add-module=../ngx_http_proxy_connect_module --with-http_ssl_module
   # 编译
   make
   # 安装
   sudo make install
   ```

   > 通过编译安装的nginx主目录位置在： /usr/local/nginx 下

5. 编辑配置文件 ，启用正向代理

   ```nginx
   worker_processes  auto;
   
   error_log  /usr/local/nginx/logs/error.log warn;
   pid        /usr/local/nginx/logs/nginx.pid;
   
   events {
       worker_connections  1024;
   }
   # 在 http 节中添加配置
   http {
       include       mime.types;
       default_type  application/octet-stream;
   
       # 配置日志
       log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
   
       access_log /usr/local/nginx/logs/access.log main;
   
       # 配置代理缓存
       proxy_cache_path /usr/local/nginx/cache levels=1:2 keys_zone=cache_zone:10m max_size=1g inactive=60m use_temp_path=off;
   
       # 配置正向代理
       server {
           listen       8888; # 你需要在对应的云主机ISP防火墙开放这个端口
           resolver 223.5.5.5; 
   
           # 允许 CONNECT 方法
           proxy_connect;
           proxy_connect_allow            all;
           proxy_connect_connect_timeout  10s;
           proxy_connect_read_timeout     10s;
           proxy_connect_send_timeout     10s;
   
           location / {
               proxy_pass http://$host;
               proxy_set_header Host $host;
               proxy_set_header X-Real-IP $remote_addr;
               proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header X-Forwarded-Proto $scheme;
           }
       }
   
   }
   ```

6. 在win上配置代理，填入云主机的ip地址和端口即可。

## 已经安装了nginx？

上面是从头开始安装nginx的例子。 如果你的云主机上已经存在了nginx，但是不想重装，可以通过平滑升级方式实现。

### 步骤

1. 首先查看你当前安装的nginx包含哪些配置项

   ```sh
   /usr/local/nginx/sbin/nginx -V
   ```

   记住`configure arguments:` 后面的内容， 我们需要保留这些 配置项，这样才不会影响你原来的nginx

2. 进入新下载的nginx源码目录， 通过configure配置新的配置项，这个配置项会覆盖掉原来的配置。因此把原来的 配置加上，再添加你想要的 其他模块。

   ```sh
   # 进入源码目录
   cd /path/to/nginx-1.24.0
   # 编译配置项   
   # xxxxxxxxxxx 指的是之前让你记下的，你原来nginx的配置项，在此基础上添加新的依赖。
   ./configure xxxxxxxxxxx --add-module=../ngx_http_proxy_connect_module
   # 编译
   make
   ```

3. 将 objs 目录下，编译好的nginx二进制文件复制到nginx的安装目录下的/sbin里。

   ```sh
   cd objs
   cp nginx /usr/local/nginx/sbin
   ```

4. 回到源码的目录，执行make upgrade指令

   ```sh
   cd /path/to/nginx-1.24.0
   make  upgrade
   ```

> 如果在执行make upgrade指令时报错： make: *** [Makefile:22：upgrade] 错误 1
>
> 尝试先kill掉原来的nginx进程，然后使用绝对路径来启动新的nginx：
>
> ```sh
> # 直接执行
> /usr/local/nginx/sbin/nginx
> ```
>
> 之后再执行 make upgrade 应该不会有问题了。

5. 安装完ngx_http_proxy_connect_module模块后，剩下的就是配置nginx.conf了，和上面一样，不再赘述。
