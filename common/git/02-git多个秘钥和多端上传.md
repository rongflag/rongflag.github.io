# 02-git多端上传多个秘钥

## 1.多端上传

https://blog.csdn.net/htp_411/article/details/99687343

1. 在Gitee和GitHub上创建一个项目
   户名为admin,仓库名为demo
2. 克隆项目到本地
   直接下载
   在本地使用命名
   //从gitee 获取
   $ git clone https://gitee.com/admin/demo.git

//从github获取
$ git clone https://github.com/admin/demo.git

3. 自定义关联远程库
删除GIt默认远程库名称
//git默认远程库名称为origin
$ git remote rm origin

分别关联Gitee和GitHub并设置名称
//关联gitee并设置别名为gitee
$ git remote add gitee @git/gitee.com:admin/demo.git



//关联github并设置别名为github
$ git remote add githob @git/github.com:admin/demo.git

4.推送到远程仓库
推送到Gitee
$ git push gitee master
推送到GitHub
$ git push github master
5.可能出现的错误
这里提示拒绝更新，提示先从远程pull再尝试
$ git push github master
To github.com:admin/demo.git
! [rejected]     master -> master (fetch first)
...

解决方法
从github上pull

$ git pull github master
1 推送到github远程仓库

$ git push github master
1可能提示push失败这里可以尝试用强制push

$ git push github master -f
/*	
由于是初始化项目，并从远程仓库pull，使用强制推送不会对项目造成影响
一般不推荐强制push

## 多个git秘钥

使用终端 ssh命令 生成 rsa秘钥

ssh-keygen -t rsa -C "邮箱地址1" -f ~/.ssh/id_rsa_github

ssh-keygen -t rsa -C "邮箱地址1" -f ~/.ssh/id_rsa_gitee

注意名称

-f 为秘钥存放地址。默认为当前路径 。一直回车下去。

### 配置多个秘钥

我们重复步骤 生成秘钥、将公钥告诉git服务器 生成并配置新的秘钥后。 在 .ssh 目录下面新建 config文件，文件内容如下： 配置以下内容：

```
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_github
    user 名称

Host gitee.com
    HostName gitee.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_gitee
    user 名称
```

HostName 指定秘钥使用域名，就能区分不同域名之间使用秘钥不同

保存后，测试 ssh -T git@github.com、ssh -T git@gitee.com


如果配置 config 文件后，未能生效。本地 git 软件的配置存在缓存，可使用以下命令判断秘钥是否生效：

$ cd ~/.ssh

$ eval $(ssh-agent)

Agent pid 3593



$ exec ssh-agent bash



$ ssh-add ~/.ssh/id_rsa_github

Identity added: /c/Users/Administrator/.ssh/id_rsa_github



$ ssh -T git@github.com

Hi coderdao! You've successfully authenticated, but GitHub does not provide shell access.