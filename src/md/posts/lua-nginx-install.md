<!--
author: wngn123
date: 2016-08-24
title: nginx lua 环境搭建
tags: lua
category: Lua
status: publish
summary: nginx lua 环境搭建
-->

## 目录结构 ##

```
drwxr-xr-x. 6 wang wang     93 May 15  2015 LuaJIT-2.0.4
-rw-r--r--. 1 root root 847615 Jun  6 21:16 LuaJIT-2.0.4.tar.gz
drwxrwxr-x. 9 root root   4096 May 26 10:41 lua-nginx-module-0.10.5
-rw-r--r--. 1 root root 579793 Jun  6 21:35 lua-nginx-module-0.10.5.tar.gz
drwxr-xr-x. 9 work work   4096 Jun  6 21:25 nginx-1.10.1
-rw-r--r--. 1 root root 909077 Jun  6 21:17 nginx-1.10.1.tar.gz
drwxrwxr-x. 9 root root   4096 May 10 05:28 ngx_devel_kit-0.3.0
-rw-r--r--. 1 root root  66455 Jun  6 21:34 ngx_devel_kit-0.3.0.tar.gz
```

## 安装依赖 ##

```shell
#使用root用户登录
yum -y install gcc gcc-c++ autoconf automake
yum -y install zlib zlib-devel openssl openssl-devel pcre-devel
```

## 下载解压安装文件 ##

```shell
cd /usr/local/src
wget http://nginx.org/download/nginx-1.10.1.tar.gz
wget http://luajit.org/download/LuaJIT-2.0.4.tar.gz
wget https://github.com/chaoslawful/lua-nginx-module/archive/v1.10.5.tar.gz
wget https://github.com/simpl/ngx_devel_kit/releases/tag/v0.3.0
tar -zxvf LuaJIT-2.0.4.tar.gz
tar -zxvf nginx-1.10.1.tar.gz

tar -zxvf lua-nginx-module-0.10.5.tar.gz
tar -zxvf ngx_devel_kit-0.3.0.tar.gz
tar -zxvf echo-nginx-module-0.59

tar -zxvf v0.59
tar -zxvf v1.10.5.tar.gz
tar -zxvf v0.3.0.tar.gz
```

## 安装LuaJIT ##

```shell
cd /usr/local/src/LuaJIT-2.0.4
make
make install
export LUAJIT_LIB=/usr/local/lib
export LUAJIT_INC=/usr/local/include/luajit-2.0
ln -s /usr/local/lib/libluajit-5.1.so.2 /lib64/libluajit-5.1.so.2
```

## 安装NGINX ##

```shell
cd /usr/local/src/nginx-1.10.1
./configure --add-module=/usr/local/src/lua-nginx-module-0.10.5 --add-module=/usr/local/src/ngx_devel_kit-0.3.0
./configure --add-module=/usr/local/src/lua-nginx-module-0.10.5 --add-module=/usr/local/src/ngx_devel_kit-0.3.0 --add-module=/usr/local/src/echo-nginx-module-0.59 
make -j2
make install
```

## 配置NGINX ##

```shell
cp /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf.bak
vi /usr/local/nginx/conf/nginx.conf

location ~* ^/2328(/.*) {
    default_type 'text/plain';
    content_by_lua 'ngx.say("hello, ttlsa lua")';
}
```

## 启动NGINX ##

```shell
/usr/local/nginx/sbin/nginx -v
/usr/local/nginx/sbin/nginx
```

## 测试NGINX ##

```shell
curl http://localhost/2328/
```

## 注 ##
1. echo命令只能放在url请求中，如果放在url请求外，会报错 如果报[emerg]: "echo" directive is not allowed here in  ，请检查echo放置的位置
2. 一次url请求，echo 只能打印一行，如果有逻辑判断，且判断成功，则echo会执行判断成功里边的echo，否则执行最后一句echo（此处不一定正确，在测试中发现是此现象）
3. 如果echo后边有配置return 或者配置 proxy_pass，则echo的输出会被覆盖，即浏览器无法看到echo的内容
4. echo的内容不是写在nginx的配置文件中，而是输出到浏览器中，所以echo的打印字符的查看请在浏览器中查看


## 参考 ##
[http://www.ttlsa.com/nginx/nginx-modules-ngx_lua](http://www.ttlsa.com/nginx/nginx-modules-ngx_lua/)
