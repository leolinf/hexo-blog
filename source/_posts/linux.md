---
title: libssl.so.10:cannot open shared object file:No such file or directory
date: 2019-11-01 10:30:23
tags: linux
categories: 技术
comment: true
---

### 1.docker from python:3.7-alpine 相关的问题记录

在使用 python最基础镜像的时候，如果要使用动态链接库，就必须自己手动安装 openssl

不然就会报下面这个错误

```
error while loading shared libraries: libssl.so.10: cannot open shared object file: No  such file or directory
```

安装方法在基础镜像的基础之上追加安装相应的库，得到最新的镜像

```
RUN apk add --no-cache openssl-dev

RUN ln -s /lib/libssl.so.1.0.0 /lib/libssl.so.10

RUN ln -s /lib/libcrypto.so.1.0.0 /lib/libcrypto.so.10
```

自此也就修复了 docker 当中会出现这个问题的情况

linux 系统出现这种情况的时候，ubuntu如下修复

Lets make sure that you have your SSL installed and updated: 

```
sudo apt-get update

sudo apt-get install libssl1.0.0 libssl-dev
```

Now lets fix the naming of the file by creating a link: 

```
cd /lib/x86_64-linux-gnu

sudo ln -s libssl.so.1.0.0 libssl.so.10

sudo ln -s libcrypto.so.1.0.0 libcrypto.so.10
```

And finally, lets inform the developer about this flaw so he can fix it
