---
layout: post
title: Git、Github 笔记
category: 技术
author: 沈伯伟
tags: Git Github
---

网上关于git的教程有很多，这里主要参考的有两个：

- [廖雪峰-Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000 "跳转")

- [git官方文档](https://git-scm.com/docs "跳转")

第一个是入门非常好的教程，对`git`的相关概念和操作有比较详细的介绍。第二个是`git`的官方文档，在里面可以查询到`git`所有命令和相关参数，可以在以后使用中当作工具书。
由于在[廖雪峰-Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000 "跳转")中已经对`git`有了非常详细的介绍，
下面主要以实践为主，外加记录一些实用的技巧。

## Git 安装配置

### 安装

```
[shebnowei@bogon ~]$ sudo yum install git

Determining fastest mirrors
 * base: mirrors.btte.net
 * epel: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.btte.net
 * updates: mirrors.btte.net
正在解决依赖关系
--> 正在检查事务
---> 软件包 git.x86_64.0.1.8.3.1-6.el7_2.1 将被 安装
--> 正在处理依赖关系 perl-Git = 1.8.3.1-6.el7_2.1，它被软件包 git-1.8.3.1-6.el7_2.1.x86_64 需要
--> 正在处理依赖关系 perl(Term::ReadKey)，它被软件包 git-1.8.3.1-6.el7_2.1.x86_64 需要
--> 正在处理依赖关系 perl(Git)，它被软件包 git-1.8.3.1-6.el7_2.1.x86_64 需要
--> 正在处理依赖关系 perl(Error)，它被软件包 git-1.8.3.1-6.el7_2.1.x86_64 需要
--> 正在检查事务
---> 软件包 perl-Error.noarch.1.0.17020-2.el7 将被 安装
---> 软件包 perl-Git.noarch.0.1.8.3.1-6.el7_2.1 将被 安装
---> 软件包 perl-TermReadKey.x86_64.0.2.30-20.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package               架构        版本                      源            大小
================================================================================
正在安装:
 git                   x86_64      1.8.3.1-6.el7_2.1         updates      4.4 M
为依赖而安装:
 perl-Error            noarch      1:0.17020-2.el7           base          32 k
 perl-Git              noarch      1.8.3.1-6.el7_2.1         updates       53 k
 perl-TermReadKey      x86_64      2.30-20.el7               base          31 k

事务概要
================================================================================
安装  1 软件包 (+3 依赖软件包)

总下载量：4.5 M
安装大小：22 M
```

如果上述安装过程没有问题，则可以查看到安装的`git`版本：

```
[shebnowei@bogon ~]$ git --version
git version 1.8.3.1
```

### 配置

`git`的配置命令为`git config`，后边有很多参数、选项，可以访问上边提到的官方文档链接来查询详细的参数和选项使用([git config命令](https://git-scm.com/docs/git-config "跳转"))。
在使用`git`的时候`user.name`和`user.email`这两个选项是用来表明客户身份的信息，必须要配置下。

`git`的配置文件分为两种：

- 全局配置文件
  针对当前用户起作用，需要通过添加`--global`参数来配置。
  所在路径：`~/.gitconfig`
  作用范围：该用户的所有`git`项目。

- 单个项目仓库的配置文件
  针对当前项目仓库起作用，需要在项目仓库中配置，不添加参数`--global`。
  所在路径：`项目路径/.git/config`
  作用范围：只对当前项目起作用，会覆盖全局的配置。
  
下面配置全局的用户信息：

```
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

配置完成后可以通过命令查看:

```
$ git config --global --list
```

类似地，针对单个项目仓库，我们可以在项目所在路径通过如下三条命令配置和查看用户信息：

```
$ git config user.name "Your Name"
$ git config user.email "email@example.com"

$ git config --list
```

## Github with Git

`GitHub`作为免费的远程仓库是我们托管、分享、学习代码的好去处。可以说正是因为`Github`，才让更多的开源项目得到了发展。下面记录如何使用`Github`来管理我们的项目。

### 托管自己的项目

我们可以分享自己的项目到`Github`上，既能达到管理代码的效果，又能得到和他人交流的目的。
步骤很简单，总结如下：

1. 在[Github官网](https://github.com "跳转")注册账号，创建一个新的仓库(Create a new repository)。

2. 创建完后，`Github`会告诉我们本地需要执行的一系列命令：

		echo "# test" >> README.md
		git init
		git add README.md
		git commit -m "first commit"

		git remote add origin https://github.com/"your name"/"your repo name".git
		git push -u origin master


	按照上边的命令一次执行，便可以推送一个`README.md`文件到你的仓库了（对于已经有完整`git`仓库的人，只需执行最后两条命令添加远程仓库和推送即可）。
	这里，我们是通过https协议方式(`https://github.com/"your name"/"your repo name".git`)链接远程仓库的，
	在执行完最后一条推送命令`git push -u origin master`后，会提示让你输入`Github`仓库所属用户的用户名和密码。

	如果不想每次推送都输入用户名和密码，可以通过`SSH`的方式连接：
	
	1. `ssh-keygen -t rsa -C "your email"`创建SSH Key，在`~/.ssh`目录下会生成两个文件：`id_rsa`（私钥，不能泄露出去）和`id_rsa.pub`（公钥，可以放心地告诉任何人）。
	
	2. 将公钥`id_rsa.pub`中的内容添加到`Github`设置中的`SSH keys`中即可。
	
	3. `git remote rm origin`命令，先删掉原来`https`协议的origin。
	
	4. `git remote add origin git@github.com:"your name"/"your repo name".git`，再重新添加`ssh`支持的`git`协议的origin。
	
	5. `git push origin master`进行推送。
	

3. 之后提交只需执行命令`git push origin master`。其他操作请参考相应指令即可。

### 贡献代码到他人开源项目

当我们在`Github`上看到了自己感兴趣的项目，并且想向该项目贡献自己的代码时，可以通过以下步骤实现：

1. 首先在该项目下点击`Fork`，这样在自己的仓库中就复刻了一份该项目的复本。

2. 在本地对应的项目目录下`git clone https://github.com/"your name"/"forked repo name".git`。这样在本地就能够看到该项目的完整仓库了（只是Fork时的版本）。

3. 此时，自己Fork的仓库和原仓库相对是独立的，那么如果原仓库更新了代码，我们的仓库如何保持和原仓库的同步是一个非常重要的问题，也是发起`Pull request`前需要保证的一步。
	我们需要把原项目的地址作为我们本项目的远程上游地址（upstream是我们起的名字，可以改为其他）：
		git remote add upstream https://github.com/"原项目作者名字"/"项目名称".git
		
		[shebnowei@localhost MyGitTest]$ git remote -v
		origin	https://github.com/"你自己的名字"/"项目名称".git (fetch)
		origin	https://github.com/"你自己的名字"/"项目名称".git (push)
		upstream	https://github.com/"原项目作者名字"/"项目名称".git (fetch)
		upstream	https://github.com/"原项目作者名字"/"项目名称".git (push)

4. `git pull upstream master`这个命令需要在发起`Pull request`之前执行，保证你现在的代码和当前原项目代码同步，这样才能保证你的提交可以正常工作在原项目（因此需要保证在最新的原项目下测试你的提交）。
	之后往往还需要执行`git push origin master`来保证你的远程仓库和原仓库一致。	

5. 这里需要注意两点：

	- 本地和`origin`的master分支最好和`upstream`的master分支保持一致，即我们的改动不建议直接在master分支。master分支需要作为原项目的参考。
	
	- 本地建立新的分支`feature/... 或 bug/...`，在该分支上进行修改，并提交到`origin`的对应分支。
	
	将自己的修改的分支push到自己Fork的仓库中对应分支（**注意：**在push前需要先执行`git pull upstream master`和`git rebase master`，将原始仓库的最新代码pull过来并合并到修改前，从而让我们的修改的commit排到了最后）。
	但是此时你的修改只是作用在你自己的项目中，对原项目是没有影响的。如果你想将你的修改告诉原项目的作者，需要发起`Pull request`，具体步骤为：
	
	1. 在自己Fork的项目中的`Branch`旁（需要提前选好修改的分支）有个`New pull request`按钮，点击进入提交页。
	
	2. 在该页面会显示目前你自己的项目和原项目是否可以merge，如果可以自动合并的话，直接点击`create pull request`按钮。（一般需要提前把最新的原项目pull下来进行合并，这样就可以避免出现问题）
	
	3. 之后填写标题和内容，告诉原作者你修复了什么bug，增加了什么功能，原作者会决定是否接受你的代码。

总结流程如下：

```
1. git clone https://github.com/"your name"/"forked repo name".git //clone fork的代码到本地

2. git remote add upstream https://github.com/"原项目作者名字"/"项目名称".git 		
	//添加上游地址，可以通过git remote -v 查看

3. git pull upstream master //更新原始项目的最新代码

4. git checkout -b "分支名称" //创建修改代码的分支

5. git add . //修改添加到暂存区

6. git commit -m "注释" //提交更改到本地分支

7. git checkout master //切换到master分支下准备再次同步

8. git pull upstream master //再次更新原始项目的最新代码

9. git checkout "分支名称" //切换回分支

10. git rebase master //将原仓库的最新commit放到我们修改后提交的commit之前

11. git push origin "分支名称" //推送到我们自己的远程仓库

```

	
### 接收他人提交的代码到自己的项目

与前面对应，如果自己的项目被别人`Fork`了，当别人发起`Pull request`后，可以通过以下步骤接收他人的代码：
（这里不是通过网页端操作，而是通过命令行）

1. 首先避免出错，需要先在本地建立分支检查他人提交的代码：`git checkout -b "某人"/"分支"`

2. `git pull https//github.com/某人/项目名称.git "分支名称"`，拉取他修改后的代码进行检查

3. `git checkout master`如果检查没有问题，切换到主分支后合并：`git merge "某人"/"分支"`

4. `git push origin master`,提交到远程库，此时该`Pull request`会自动关闭。


## Github 小技巧

下面为`git`相关使用指令的快捷列表，便于快捷查询使用：

[github-git-cheat-sheet](/assets/files/github-git-cheat-sheet.pdf "跳转")

> ## 参考文献
>
>[廖雪峰-Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000 "跳转")
>
>[git官方文档](https://git-scm.com/docs "跳转")
>
>[Pro Git（中文版）](http://git.oschina.net/progit/ "跳转")








