# FTP安装手册

* [注：使用场景说明](#1.%20%E4%B8%8B%E8%BD%BDvsftpd%E5%92%8Cftp%E5%AE%89%E8%A3%85%E5%8C%85)
* [1. 下载vsftpd和ftp安装包](#2.%20%E5%AE%89%E8%A3%85vsftpd%E5%92%8Cftp)
* [2. 安装vsftpd和ftp](#%E5%AE%89%E8%A3%85vsftpd)
* [3. 修改配置文件](#3.%20%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
  * [3.1 禁用SELinux](#3.1%20%E7%A6%81%E7%94%A8SELinux)
  * [3.2 开启匿名用户功能](#3.2%20%E5%BC%80%E5%90%AF%E5%8C%BF%E5%90%8D%E7%94%A8%E6%88%B7%E5%8A%9F%E8%83%BD)
  * [3.3 开启本地用户登录](#3.3%20%E5%BC%80%E5%90%AF%E6%9C%AC%E5%9C%B0%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95)
  * [3.4 开启非本地用户登录](#3.4%20%E5%BC%80%E5%90%AF%E9%9D%9E%E6%9C%AC%E5%9C%B0%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95)
  * [3.5 限制用户访问范围](#3.5%20%E9%99%90%E5%88%B6%E7%94%A8%E6%88%B7%E8%AE%BF%E9%97%AE%E8%8C%83%E5%9B%B4)

---

## 使用场景说明

系统使用版本：CentOS 7.9.2009 for x86_64

---

## 1. 下载vsftpd和ftp安装包

下载链接：

* vsftpd: <http://rpmfind.net/linux/rpm2html/search.php?query=vsftpd%28x86-64%29>

* ftp: <http://rpmfind.net/linux/rpm2html/search.php?query=ftp&submit=Search+...&system=&arch=>

---

## 2. 安装vsftpd和ftp

将下载好的安装包放到服务器上

执行安装命令

```linux
[root@localhost~]# rpm -ivh vsftpd-3.0.2-28.el7.x86_64.rpm
```

```linux
[root@localhost~]# rpm -ivh ftp-0.17-67.el7.x86_64.rpm
```

---

## 3. 修改配置文件

vsftpd服务程序的主配置文件（/etc/vsftpd/vsftpd.conf）

默认文件存储位置 (/var/ftp)

### 3.1 禁用SELinux

查看SELinux状态

```linux
[root@localhost~]# /usr/sbin/sestatus -v
SELinux status:                 disabled
```

如果状态不是disabled,需要修改其配置文件

```linux
[root@localhost~]# vi /etc/selinux/config
```

注释掉SELINUX=*****添上一行，添加上

```linux
SELINUX=disabled
```

设置开机自启

```linux
[root@localhost~]# systemctl enable vsftpd
```

重启服务器

### 3.2 开启匿名用户功能

```linux
[root@localhost~]# vi /etc/vsftpd/vsftpd.conf
```

在底部添加三行：

```linux
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
```

此时无法上传文件和新建文件夹，因为文件的权限被限制了

### 3.3 开启本地用户登录

以root身份登录

修改以下两个vsftpd禁用表，注释root一行

```linux
[root@localhost~]# vim /etc/vsftpd/user_list
```

```linux
[root@localhost~]# vim /etc/vsftpd/ftpusers
```

设置root密码

```linux
[root@localhost~]# password root
```

登录

```linux
[root@localhost~]# ftp 192.168.33.55
```

以root进入可看整个系统的文件

### 3.4 开启非本地用户登录

采用非本地用户模式登录，需创建FTP组“FTP”用户

与采用本地用户模式的区别是：采用非本地用户模式可以指定FTP文件夹。非本地用户也不能shell连接服务器

新增ftp组的用户，并把shell设置为nologin：

```linux
[root@localhost~]# useradd -g ftp -s /sbin/nologin ftpuser
```

设置用户密码(192.168.33.55上设置为wenjie)

```linux
[root@localhost~]# passwd ftpuser
```

创建用户登录的家目录地址

```linux
[root@localhost~]# mkdir /var/ftp/ftpuser
```

改变文件夹所属用户

```linux
[root@localhost~]# chown -R ftpuser ftpuser
```

改变用户的登录家目录

```linux
[root@localhost~]# usermod -d /var/ftp/ftpuser ftpuser
```

vsftpd 将shell设置成为nologin后不能登录解决办法:

禁止vsftpd通过pam认证,并将check_shell配置为NO

```linux
[root@localhost~]# vim /etc/pam.d/vsftpd
```

注释一行：

```linux
#auth  required    pam_shells.so
```

将check_shell配置为NO

```
[root@localhost~]# vim /etc/vsftpd/vsftpd.conf
```

在底部加上一行：

```
check_shell=NO
```

重启服务

```
[root@localhost~]# systemctl restart vsftpd
```

测试

```
连接
[root@localhost~]# ftp 192.168.33.55

上传文件
ftp> put /web/test.php ./test.php

下载文件
ftp> get ./test.php /web/test111.php

新建文件夹
ftp> mkdir hello
```

此时的ftpuser及root用户是可以访问系统所有文件的，下一步操作将用户锁定在家目录

### 3.5 限制用户访问范围

* chroot_local_user : 是否将所有用户限制在主目录

* chroot_list_enable : 是否启动限制用户的名单

* /etc/vsftpd/chroot_list : 不限制用户的白名单文件

修改配置文件

```
[root@localhost~]# vim /etc/vsftpd/vsftpd.conf
```

```
## 取消注释
chroot_local_user=YES

## 底部新加一行
allow_writeable_chroot=YES
```

修改白名单文件，不限制root用户

```
[root@localhost~]# vi /etc/vsftpd/chroot_list
```

```
## 添加一行
root
```

重启服务

此时，使用ftpuser登录会限制在家目录，而root可以访问系统所有文件

