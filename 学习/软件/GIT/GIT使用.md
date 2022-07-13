# GIT使用

* [1. 建立远程仓库](#1.%20%E5%BB%BA%E7%AB%8B%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93)
  * [1.1 注册github账号](#1.1%20%E6%B3%A8%E5%86%8Cgithub%E8%B4%A6%E5%8F%B7)
  * [1.2 创建仓库](#1.2%20%E5%88%9B%E5%BB%BA%E4%BB%93%E5%BA%93)
  * [1.3 提交代码到远程仓库](#1.3%20%E6%8F%90%E4%BA%A4%E4%BB%A3%E7%A0%81%E5%88%B0%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93)

---

## 1. 建立远程仓库

### 1.1 注册github账号

* 官网地址 : <https://github.com/>

### 1.2 创建仓库

* 登录账号

* 找到 NEW按钮 点击create repository，选择public（private付费），输入仓库名，创建

* 此时以建立了仓库，复制仓库地址后续使用

* 修改密码认证方式

新版本放弃了password，使用token

首先在GitHub的个人设置页面，找到setting，选择开发者设置Developer setting

选择个人访问令牌Personal access tokens

然后选中生成令牌Generate new token

要使用token从命令行访问仓库，请选择repo

要使用token从命令行删除仓库，请选择delete_repo

此处要把workflow也勾选上

保存token，因为页面刷新就不再显示该token

```linux
// git push时的密码
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
```

### 1.3 提交代码到远程仓库

* 进入需要提交的文件夹

```linux
[root@localhost~]# cd /web/my_study/
```

* 初始化本地仓库

```linux
[root@localhost my_study]# git init
```

* 修改当前分支的名称

```linux
[root@localhost my_study]# git branch -M study
```

* 执行git add .

```linux
[root@localhost my_study]# git add .
```

* 执行git commit 提交到本地仓库

```linux
[root@localhost my_study]# git commit -m '部署文档'
```

* 添加远程仓库地址, my_study为主机名

```linux
[root@localhost my_study]# git remote add my_study https://github.com/levytao/my_study.git
```

* 查看远程仓库地址

```linux
[root@localhost my_study]# git remote -v
my_study        https://github.com/levytao/my_study.git (fetch)
my_study        https://github.com/levytao/my_study.git (push)
```

* 修改远程地址（会修改所有）

```linux
[root@localhost my_study]# git remote set-url my_study https://github.com/levytao/my_study.git
```

* 将代码推送到远程仓库

git push <远程主机名> <本地分支名>:<远程分支名>

若远程分支名不存在，则会自动创建

```linux
[root@localhost my_study]# git push my_study study:study
```

