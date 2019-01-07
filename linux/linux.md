---
title: Linux服务器基本配置
author: 大大白
date: 2015-01-06 18:30:00
categories:
- Linux
- 服务器基本配置
tags: 
- Linux
---

Linux服务器标配的基本环境。

<!-- more -->

# 安装Nginx
1. 需要先安装Nginx的依赖。`-y`是可选参数，意思是在安装的过程中遇到`Is this OK[y/d/N]`的时候，自动选择`y`。这个其实就是和Windows环境下的默认安装是同一个道理。
```
yum [-y] install gcc
yum [-y] install pcre-devel
yum [-y] install zlib zlib-devel
yum [-y] install openssl openssl-devel
```
2. 下载Nginx的安装包。
```
wget http://nginx.org/download/nginx-1.13.7.tar.gz
```
3. 解压安装包。
```
tar -xvf nginx-1.13.7.tar.gz
```
4. 运行configure。参数`--with-http_ssl_module`顾名思义，就是准备等下安装Nginx的时候带上ssl模块。
```
cd nginx-1.13.7/
./configure --with-http_ssl_module
```
5. 编译。
make
6. 安装。
make install

# 安装rar解压工具。
