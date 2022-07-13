# GIT安装手册

* [注：使用场景说明](#1.%20%E4%B8%8B%E8%BD%BDvsftpd%E5%92%8Cftp%E5%AE%89%E8%A3%85%E5%8C%85)
* [1. 下载git安装包](#1.%20%E4%B8%8B%E8%BD%BDgit%E5%AE%89%E8%A3%85%E5%8C%85)
* [2. 安装依赖包](#2.%20%E5%AE%89%E8%A3%85%E4%BE%9D%E8%B5%96%E5%8C%85)
* [3. 安装git](#3.%20%E5%AE%89%E8%A3%85git)
* [4. 配置环境变量](#4.%20%E9%85%8D%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)
* [5. 检测安装是否成功](#5.%20%E6%A3%80%E6%B5%8B%E5%AE%89%E8%A3%85%E6%98%AF%E5%90%A6%E6%88%90%E5%8A%9F)

---

## 使用场景说明

系统使用版本：CentOS 7.9.2009 for x86_64

---

## 1. 下载git安装包

下载链接：

* git : <https://git-scm.com/download/linux>

---

## 2. 安装依赖包

如果已经安装过依赖包，卸载原有包或更新对应版本

下载链接:

* rpm包 : <https://rpmfind.net/linux/rpm2html/>

(1) zlib-devel、zlib

检查是否安装了zlib，发现1.2.7-18版本

```linux
[root@localhost~]# rpm -qa | grep zlib
zlib-1.2.7-18.el7.x86_64
```

更新zlib-1.2.7-20版本并自动删除旧版本

```linux
[root@localhost~]# rpm -Uvh zlib-1.2.7-20.el7_9.x86_64.rpm --nodeps --force
Preparing...                          ################################# [100%]
Updating / installing...
   1:zlib-1.2.7-20.el7_9              ################################# [ 50%]
Cleaning up / removing...
   2:zlib-1.2.7-18.el7                ################################# [100%]
```

安装zlib-devel

```linux
[root@localhost~]# rpm -ivh zlib-devel-1.2.7-20.el7_9.x86_64.rpm
```

(2) 安装curl

检查是否安装了curl

```linux
[root@localhost~]# rpm -qa | grep curl
curl-7.29.0-51.el7.x86_64
libcurl-7.29.0-51.el7.x86_64
```

覆盖安装curl-7.29.0-59版本及其依赖

```linux
[root@localhost~]# rpm -Uvh libcurl-7.29.0-59.el7_9.1.x86_64.rpm --nodeps --force
Preparing...                          ################################# [100%]
Updating / installing...
   1:libcurl-7.29.0-59.el7_9.1        ################################# [ 50%]
Cleaning up / removing...
   2:libcurl-7.29.0-51.el7            ################################# [100%]

[root@localhost~]# rpm -Uvh libssh2-1.8.0-4.el7.x86_64.rpm --nodeps --force
Preparing...                          ################################# [100%]
Updating / installing...
   1:libssh2-1.8.0-4.el7              ################################# [ 50%]
Cleaning up / removing...
   2:libssh2-1.4.3-12.el7_6.2         ################################# [100%]

[root@localhost~]# rpm -Uvh curl-7.29.0-59.el7_9.1.x86_64.rpm --nodeps --force
Preparing...                          ################################# [100%]
Updating / installing...
   1:curl-7.29.0-59.el7_9.1           ################################# [ 50%]
Cleaning up / removing...
   2:curl-7.29.0-51.el7               ################################# [100%]
```

安装curl-devel

```linux
[root@localhost~]# rpm -ivh libcurl-devel-7.29.0-59.el7_9.1.x86_64.rpm
```

(3) 安装expat，expat-devel

检测是否安装expat

```linux
[root@localhost~]# rpm -qa | grep expat
expat-2.1.0-10.el7_3.x86_64
```

覆盖安装并删除旧版

```linux
[root@localhost~]# rpm -Uvh expat-2.1.0-14.el7_9.x86_64.rpm --nodeps --force
```

安装expat-devel

```linux
[root@localhost~]# rpm -ivh expat-devel-2.1.0-14.el7_9.x86_64.rpm
```

(4) 安装gettext

```linux
[root@localhost~]# rpm -Uvh gettext-libs-0.19.8.1-3.el7.x86_64.rpm --nodeps --force

[root@localhost~]# rpm -Uvh gettext-0.19.8.1-3.el7.x86_64.rpm --nodeps --force
```

(5) 安装openssl

```linux
[root@localhost~]# rpm -Uvh openssl-libs-1.0.2k-25.el7_9.x86_64.rpm --nodeps --force

[root@localhost~]# rpm -Uvh openssl-1.0.2k-25.el7_9.x86_64.rpm --nodeps --force
```

---

## 3. 安装git

解压到/usr/local目录下

```linux
[root@localhost~]# tar -zxvf git-2.36.1.tar.gz -C /usr/local/
```

进入解压后的文件夹

```linux
[root@localhost~]# cd /usr/local/git-2.36.1/
```

执行configure命令

```linux
[root@localhost git-2.36.1]# ./configure --prefix /usr/local/git
```

执行make命令

```linux
[root@localhost git-2.36.1]# make prefix=/usr/local/git
```

执行make install命令

```linux
[root@localhost git-2.36.1]# make prefix=/usr/local/git install
```

---

## 4. 配置环境变量

```linux
[root@localhost~]# vi /etc/profile
```

在底部添加一行

```linux
export PATH=$PATH:/usr/local/git/bin
```

使文件生效

```linux
[root@localhost~]# source /etc/profile
```

---

## 5. 检测安装是否成功

```linux
[root@localhost~]# git version
git version 2.36.1
```
