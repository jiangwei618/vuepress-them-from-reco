---
title: git基本原理
date: 2019-09-03T17:27:09.000Z
description: git基本原理
categories:
    - 工具
    - git
tags:
    - git
---  
  
  
##  1. 概念
  
  
###  1.1. 四个工作区域
  
  
Git 本地有四个工作区域：工作目录（Working Directory）、暂存区(Stage/Index)、资源库(Repository 或 Git Directory)、git 仓库(Remote Directory)。文件在这四个区域之间的转换关系如下：
  
![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/git基本原理.md-2019-09-03-18-06-12.png )
  
- Workspace： 工作区，就是你平时存放项目代码的地方
  
- Index / Stage： 暂存区，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息
  
- Repository： 仓库区（或版本库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中 HEAD 指向最新放入仓库的版本
  
- Remote： 远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换
  
###  1.2. 工作流程
  
  
git 的工作流程一般是这样的：
  
１、在工作目录中添加、修改文件；
  
２、将需要进行版本管理的文件放入暂存区域；
  
３、将暂存区域的文件提交到 git 仓库。
  
因此，git 管理的文件有三种状态：已修改（modified）,已暂存（staged）,已提交(committed)
  
###  1.3. 文件的四种状态
  
  
版本控制就是对文件的版本控制，要对文件进行修改、提交等操作，首先要知道文件当前在什么状态，不然可能会提交了现在还不想提交的文件，或者要提交的文件没提交上。
  
GIT 不关心文件两个版本之间的具体差别，而是关心文件的整体是否有改变，若文件被改变，在添加提交时就生成文件新版本的快照，而判断文件整体是否改变的方法就是用 SHA-1 算法计算文件的校验和。
  
![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/git基本原理.md-2019-09-03-18-07-18.png )
  
- Untracked: 未跟踪, 此文件在文件夹中, 但并没有加入到 git 库, 不参与版本控制. 通过 git add 状态变为 Staged.
  
- Unmodify: 文件已经入库, 未修改, 即版本库中的文件快照内容与文件夹中完全一致. 这种类型的文件有两种去处, 如果它被修改, 而变为 Modified.如果使用 git rm 移出版本库, 则成为 Untracked 文件
  
- Modified: 文件已修改, 仅仅是修改, 并没有进行其他的操作. 这个文件也有两个去处, 通过 git add 可进入暂存 staged 状态, 使用 git checkout 则丢弃修改过,返回到 unmodify 状态, 这个 git checkout 即从库中取出文件, 覆盖当前修改
  
- Staged: 暂存状态. 执行 git commit 则将修改同步到库中, 这时库中的文件和本地文件又变为一致, 文件为 Unmodify 状态. 执行 git reset HEAD filename 取消暂存,文件状态为 Modified
  
下面的图很好的解释了这四种状态的转变：
  
![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/git基本原理.md-2019-09-03-17-30-33.png )
  
新建文件--->Untracked
  
使用 add 命令将新建的文件加入到暂存区--->Staged
  
使用 commit 命令将暂存区的文件提交到本地仓库--->Unmodified
  
如果对 Unmodified 状态的文件进行修改---> modified
  
如果对 Unmodified 状态的文件进行 remove 操作--->Untracked
  