---
title: "Git"
type: "docs"
weight: 1
---

Git 是分布式版本控制系统，每个用户端都会拥有本地仓库（.git 目录），可以在没有网络的情况下 commit、等网络恢复再 push 到服务端。

![git](git.png)

## 仓库

```shell
# 进入仓库目录、初始化空仓库
cd project && git init

# 初始化裸仓库。
# 无工作区、暂存区，只有仓库，节省空间，适用于服务端。
git init --bare

# 新建project.git目录，并将其初始化为裸仓库。
# 推荐服务端仓库文件夹以.git后缀命名。
git init --bare project.git

# 克隆仓库到本地，http方式
git clone https://git.test/username/project.git

# 克隆仓库到本地，ssh方式
git clone git@git.test:username/project.git

# 查看仓库设置
git config --list

# 设置全局git用户名
git config --global user.name "name"

# 设置全局git用户邮箱
git config --global user.email "test@test.com"`

# 修改远程仓库地址
git remote set-url origin https://git.test/username/project2.git

# 修改远程仓库地址、以SSH方式访问
git remote set-url origin git@git.test:username/project2.git

# 修改远程仓库地址，带用户名密码、后续操作无需再输入密码
git remote set-url origin https://username:password@git.test/username/project2.git
```

### 多仓库配置 ssh

```shell
# 配置ssh，内容在下方
vi ~/.ssh/config

# 克隆github仓库到本地
git clone github:username/project.git

# 克隆test仓库到本地
git clone test:username/project.git

# 修改远程仓库地址，通过私钥访问Git服务
git remote set-url origin github:username/project2.git
```

```text
# Github
Host github    # 简称
  HostName github.com    # 服务器地址
  User git    #ssh用户
  IdentityFile ~/.ssh/id_rsa_github    # 私钥

# My Server
Host test
 HostName 127.0.0.1
 User git
 IdentityFile ~/.ssh/id_rsa_lynnclub
```

## 分支

```shell
# 查看本地的分支
# 建议养成经常查看当前分支的习惯，以免操作错了分支、惹来麻烦。
git branch

# 查看本地分支与远程分支
git branch -a

# 检出本地分支
# 本地不此存在、远程存在，将检出远程分支到本地
git checkout feature_test

# 基于当前分支新建分支
git branch feature_test

# 新建分支并且切换到新分支
git checkout -b feature_test

# 将新分支推送到远程服务端
git push --set-upstream origin feature_test

# 修改分支的名称
git branch -m feature_test feature_rename

# 将feature_test合并到当前分支
git merge feature_test

# 取消刚刚的合并操作，如果已提交则无效
git merge --abort

# 删除本地分支
# 必须checkout到其它分支，才能删除该分支
git branch -d feature_test

# 删除远程分支
git push origin --delete feature_test
# 或者
git push origin :feature_test
```

## 文件操作

```shell
# 显示修改文件清单
git status

# 查看文件的修改差异
git diff a.php
# 查看所有文件的修改差异
git diff .

# 丢弃工作区的修改
# 支付通配符（例如 \*.php）
# 危险操作，checkout之后无法找回。建议多使用stash临时储存，这样误操作checkout之后，还可以找回
git checkout a.php

# 移动文件或目录
git mv a.php b.php

# 删除文件
git rm b.php

# 添加文件或目录到暂存区，支持通配符
git add a.php

# 提交暂存区到本地仓库
# 必须填写-m 说明
# 建议每次提交，有且仅有一个完整的功能。频繁的零星提交，不利于查看记录、或者回滚操作。
git commit -m 'test'

# 推送到本地分支对应的远程仓库分支
git push

# 查看提交记录
git log
```

### stash

```shell
# 临时存储修改，将修改从工作区移入堆栈
# 适合保存不便于加入暂存区的修改
git stash

# 临时存储修改并备注
git stash save "test"

# 查看临时存储列表
git stash list

# 弹出临时储存内容，将修改恢复到工作区
git stash pop

# 弹出堆栈中指定的临时储存，1表示弹出第二个
git stash pop stash@{1}
```

## tag

```shell
# 新建标签
git tag 1.0

# 新建带备注的标签
git tag -a 1.0 -m "my tag"

# 查看tag详细信息
git show 1.0

# 将tag推送到远程
git push origin 1.0

# 删除本地仓库tag
git tag -d 1.0

# 删除远程仓库tag
git push origin :refs/tags/1.0
```

## 回滚、重置、找回

```shell
# 回滚某次commit
git revert 326fc9f70d022afdd31b0072dbbae003783d77ed

# 强制重置HEAD到指定版本
# 危险操作，请谨慎
git reset 326fc9f70d022afdd31b0072dbbae003783d77ed

# 回退HEAD的位置，可用于回滚
# HEAD指向的是使用中分支的最后一次更新
# 2个^表示回退两个commit
git reset HEAD^^

# 100表示回退一百个commit
git reset HEAD~100

# 回退HEAD版本库
git reset --soft HEAD^^

# 回退HEAD版本库、暂存区，mixed为默认参数
git reset --mixed HEAD^^

# 回退HEAD版本库、暂存区、工作区
# 使用需要谨慎
git reset --hard HEAD^^

# 把暂存区文件拉回工作区
git reset HEAD -- <file>

# 改变分支的节点，不常用
git rebase master

# 记录了每一次操作，即使reset掉的commit，reflog都会有记录
git reflog

# 从数据库查找丢失数据，可恢复stash弹出后、误操作checkout丢失的数据
git fsck --lost-found
```
