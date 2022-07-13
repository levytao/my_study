# yum使用

* [1. 只下载不安装](#1.%20%E5%8F%AA%E4%B8%8B%E8%BD%BD%E4%B8%8D%E5%AE%89%E8%A3%85)

---

## 1. 只下载不安装

系统尚未安装此包：

```
[root@localhost~]# yum install --downloadonly --downloaddir=/usr/local vsftpd
```

系统已安装此包：

```
[root@localhost~]# yum reinstall --downloadonly --downloaddir=/usr/local vsftpd
```