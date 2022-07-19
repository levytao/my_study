# GIT错误

* [1. 新建分支报Not a valid object name](#1.%20%E6%96%B0%E5%BB%BA%E5%88%86%E6%94%AF%E6%8A%A5Not%20a%20valid%20object%20name)
* [2. 报错unsafe repository](#2.%20%E6%8A%A5%E9%94%99unsafe%20repository)
* [3. fatal: unable to access](#3.%20fatal%3A%20unable%20to%20access)

---

## 1. 新建分支报Not a valid object name

* 错误情况

```linux
[root@localhost my_study]# git branch main
fatal: not a valid object name: 'main'
```

* 产生原因

原因是没有提交一个对象

* 解决方案

要先commit之后才会真正建立master分支，此时才可以建立其它分支

```linux
[root@localhost my_study]# git commit -m '部署文档'

[root@localhost my_study]# git branch
* main

[root@localhost my_study]# git branch study

[root@localhost my_study]# git branch
* main
  study

[root@localhost my_study]# git checkout study
Switched to branch 'study'

[root@etcd3 my_study]# git branch
  main
* study
```

---

## 2. 报错unsafe repository

* 产生原因

vagrant与window共享文件夹在linux虚拟机中所属用户和组都为vagrant且不能修改

```linux
[root@localhost /]# ll | grep web
drwxrwxrwx    1 vagrant vagrant       4096 Jul 13 01:36 web
```

* 解决方案

```linux
[root@localhost my_study]# git config --global --add safe.directory "*"
```

---

## 3. fatal: unable to access

* 错误情况

```linux
[root@localhost my_study]# git push my_study study:study
fatal: unable to access 'https://github.com/levytao/my_study.git/': TCP connection reset by peer
```

* 产生原因

发生这种情况是git设置了代理这种情况是git设置了代理

* 解决方案

先设置全局代理，再取消全局代理

```linux
[root@localhost my_study]# git config --global http.proxy http://127.0.0.1:1080

[root@localhost my_study]# git config --global https.proxy http://127.0.0.1:1080

[root@localhost my_study]# git config --global --unset http.proxy

[root@localhost my_study]# git config --global --unset https.proxy
```
