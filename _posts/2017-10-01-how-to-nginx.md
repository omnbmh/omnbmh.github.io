---
layout: post
title: Nginx的安装、使用和调优
date: 2017-10-01 10:10:10
tags: [HowTo nginx]
categories: [test]
---

### 安装

+ 源码安装

```
# 下载源码
curl -o nginx-1.16.0.tar.gz http://nginx.org/download/nginx-1.16.0.tar.gz -v

# 解压
tar zxvf nginx-1.16.0.tar.gz

# 编译
> 未完待续
```

+ 包安装
```
# CentOS
yum install nginx
```

### 使用

#### 默认配置文件查找顺序


#### 命令参数解析

```
-c </path/to/config> 为 Nginx 指定一个配置文件，来代替缺省的。
-t 不运行，而仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件。
-v 显示 nginx 的版本。
-V 显示 nginx 的版本，编译器版本和配置参数。
```

#### 启动
```
nginx -c /etc/nginx.conf
```

#### 停止
```
nginx -s stop
```

### 调优

### 扩展 Nginx集成Lua

Nginx默认不支持Lua语言,需要重新编译

- 下载要用到的文件

```
curl http://luajit.org/download/LuaJIT-2.0.5.tar.gz -o LuaJIT-2.0.5.tar.gz -v
make install PREFIX=/usr/local/luagit

```
- 准备完成 编译
```
cd nginx-1.16.0/
./configure --prefix=/usr/local/nginx-1.16.0 --with-http_realip_module --with-http_stub_status_module --with-http_ssl_module \
--with-http_flv_module --with-http_gzip_static_module --with-cc-opt=-O3 --with-stream --add-module=../ngx_devel_kit-0.3.1rc1 --add-module=../lua-nginx-module-0.10.9rc7 --with-openssl=../openssl-1.0.2s \
--with-zlib=../zlib-1.2.11
```

- configure 问题
```

./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.

缺少 pcre 包
yum install pcre pcre-devel

./configure: error: SSL modules require the OpenSSL library.
You can either do not enable the modules, or install the OpenSSL library
into the system, or build the OpenSSL library statically from the source
with nginx by using --with-openssl=<path> option.

缺少 openssl 包
yum install openssl openssl-devel


```

- make 问题

```
../lua-nginx-module-0.10.8/src/ngx_http_lua_headers.c: In function ‘ngx_http_lua_ngx_req_raw_header’:
../lua-nginx-module-0.10.8/src/ngx_http_lua_headers.c:151: error: incompatible types when assigning to type ‘struct ngx_buf_t *’ from type ‘ngx_chain_t’
../lua-nginx-module-0.10.8/src/ngx_http_lua_headers.c:227: error: incompatible types when assigning to type ‘struct ngx_buf_t *’ from type ‘ngx_chain_t’
make[1]: *** [objs/addon/src/ngx_http_lua_headers.o] Error 1

版本兼容问题 lua-nginx-module

```

- 启动错误

```
./sbin/nginx -t
./sbin/nginx: error while loading shared libraries: libluajit-5.1.so.2: cannot open shared object file: No such file or directory

# 就这么干吧
ln -s /usr/local/lib/libluajit-5.1.so.2 /lib64/libluajit-5.1.so.2
```
