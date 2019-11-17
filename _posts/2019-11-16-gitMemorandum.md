---
layout: post
title: Git操作备忘
date: 2019-11-16
tags: 工具
---
# Git操作备忘

Git一般只添加数据。你执行的 Git 操作，几乎只往 Git 数据库中增加数据。很难让 Git 执行任何不可逆操作，或者让它以任何方式清除数据。同别的 VCS 一样，未提交更新时有可能丢失或弄乱修改的内容；但是一旦你提交快照到 Git 中，就难以再丢失数据，特别是如果你定期的推送数据库到其它仓库的话。所以，一旦commit过了，那就大胆的玩吧。

## 本地与推送

![20191118094635.png](https://i.loli.net/2019/11/18/nmzA64vsPYFM3GS.png)

以上是常用的Git命令图。和远程的交互基本只是push，操作基本都在本地最后push到远程即可。

git bash下键入命令```gitk --all```可以打开log可视化界面，或者输入```git gui```可以调出gui界面。

## 指针与节点
**强烈推荐一个在线可视化git实战网址：[Learn Git Branch](https://learngitbranching.js.org/)，跟着这个网址实战，就明白了Git指针的灵活与强大**

要知道，git中有一个特殊的指针HEAD，表示的是你当前正位于哪里。此外，git中的每个branch分支，都可以看作一个指针。如```git branch dev-tmp```命令可以在当前HEAD指针处新建一个分支(指针)dev-tmp

**Git的魅力在于指针的操作，以下介绍一些指针移动的操作**

## commit
最简单的就是commit了，每commit一次，HEAD指针向前移动到新的提交，并且当前的分支也向前移动到新的提交。
+ ```git commit -m "提交信息"```：提交暂存区的修改到仓库中
+ ```git commit --amend```：打开vim你可以修改上一次的提交信息，同时提交暂存区的修改到仓库中，不会生成新建节点。

## merge
Git会使用两个分支的末端所指的快照以及这两个分支的工作祖先，做一个简单的三方合并，生成一个节点，并指向该节点。
+ ```git merge dev```：将dev分支合并到当前分支，假设当前分支是master，那其操作就是，找到dev和master的最新节点，找到dev、master两者的共同祖先节点，将这三个节点合并。

## rebase
rebase是一个强大的命令，他的原理是对比两个节点的差异，并将差异在第三节点上重放，即所谓的“变基”。基于这个原理，他能做到对历史节点进行编辑拆分、删除、复制、粘贴。
+ ```git rebase master```：假设当前处于dev分支，该命令会找到dev和master最新节点的共同祖先节点，然后对比dev和该祖先节点的历史提交，提取相应的修改并存为临时文件，然后将当前分支指向目标基底 master, 最后以此将之前另存为临时文件的修改依序应用重放。
+ ```git rebase -i head~3```：该命令会打开vim进行交互，可以实现合并节点squash，删除节点drop，修改节点edit，还可以调整节点顺序(调整vim中的顺序即可)。

## checkout
checkout是一个很强大的命令，它的作用是将当前的HEAD指针移向指定的位置，并重置暂存区和工作区。HEAD指针是可以与分支的指针分离的，这时的HEAD指针称为游离指针。
+ ```git checkout <commitId>```：将HEAD指针指向<commitId>处，将<commitId>的文件覆盖到暂存区和工作区，已有的修改会保留。
+ ```git checkout <commitId> file```：检出commitId节点的指定文件，工作区和暂存区已有的修改将被丢弃
+ ```git checkout <branchName>```：同git checkout <commitId>

## reset
不像checkout，reset命令会同时移动HEAD和分支指针，他有```soft, mixed, hard```三种参数。
+ ```git reset HEAD~3 --soft```：仅仅移动指针到指定的节点，工作区和暂存区不会丢失修改
+ ```git reset HEAD~3 --mixed```：这是默认情况，移动指针到指定的节点，并使用节点内容覆盖暂存区，工作区的修改不会丢弃。
+ ```git reset HEAD~3 --hard```：移动指针到指定的节点，并使用节点内容覆盖工作区和暂存区，修改的内容会丢弃。
+ ```git reset file.txt```：这条命令本质上是git reset head file.txt --mixed，其使用了节点的file.txt覆盖了暂存区。

## branch
branch分支命令，也是可以用了移动指针的
+ ```git branch branchname```：该命令建立一个branchname分支，即新建一个branchname指针
+ ```git branch -f dev <commitId>```：该命令强制将dev指针指向了commitId处

## cherry-pick
cherry-pick拣选，这个命令会挑选指定的节点，重放在当前节点上。
+ ```git cherry-pick <commitId>```：拣选commitId处的节点，将其重放在当前节点上。

以上。
