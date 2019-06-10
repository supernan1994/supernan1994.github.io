---
layout: post
title:  "code review工具：gerrit实战"
date:   2019-06-09 00:00:00
categories: management
comments: true
image: 2019-06-09-gerrit-in-action/cover.jpg
tags: [management]
---

> 文章欢迎转载，但转载时请保留本段文字，并置于文章的顶部。  
> 作者：supernan1994，[原文地址](https://supernan1994.github.io/management/gerrit-in-action.html)

----
* any list
{:toc}

----

## 一、意义
code review对于工程团队具有很多工程意义：
1. 提升代码质量
2. 知识共享
3. 找到问题的更优解 
4. 促进团队合作和获取指导资源的渠道

## 二、选型原因
gerrit是google内部的code review工具，选择gerrit的原因有以下几点：
1. 开源系统
2. 兼容git协议
3. 丰富的插件，能和gitlab, jenkins集成
4. 有webhook和丰富的api，可以集成企业微信机器人
5. IntelliJ ide有成熟插件，vs code有插件
6. 完善的权限控制系统
7. andriod, golang, openstack背书

## 三、开发规范
1. 不要把没测试的代码提交到gerrit
2. 不要把多个tapd任务的改动合并为一个change
3. 不要把一个tapd任务的改动分成多个change
4. 多人协作的情况下，起多分支开发
5. review到错误可以-1，这样在修复之前其他人就不需要浪费时间review了
6. branch命名规范
    - feature/v3.18_O1
7. commit命名规范（标准的git commit语义参考：[git commit message](https://gist.github.com/joshbuchea/6f47e86d2510bce28f8e7f42ae84c716)）
    - [feat][v3.18 O1] 增加开放平台v2发消息接口 
    - [fix][v3.17 O2] 创建账号时默认赋予IM权限
    - [refactor][v3.18 O1] 重构开放平台v2同步消息接口
    - [chore][v3.18 Q4] 迁移欢迎语数据

## 四、踩坑指南

gerrit是git的超集，实操过程中会遇到很多问题，解决起来学习曲线比较陡峭。因此本节整理了一下常见问题，给出解决方案。过程中会涉及一些原理。

### 1. 处理gerrit和gitlab不同步
gerrit做code review的原理：当开发者提交一个change时，相当于在gerrit上的**临时分支**提交代码。比如在master修改后提交change等待review，实际上代码提交到了refs/for/master分支上。review完submit的时候，代码才会真正提交到master上。

<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15588593895436.jpg"/></div>

gerrit的gitlab插件会将gitlab作为gerrit的slave，每隔几秒就把最新的**真实分支**的代码提交到gitlab上。当有人绕过gerrit向gitlab提交代码，在插件工作时，会认为两个仓库不同步，从而不能合并。
如果要解决这个问题，我们需要把gitlab最新的改动手动同步到gerrit上。
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15588586460497.jpg"/></div>



在本地执行：
```
在gitlab上查看最新一次提交的分支
▶ git checkout {不同步的分支}
▶ git remote add upstream https://gitlab.com/[laiye-backend-repos|laiye-frontend-repos|laiye-nlp-repos]/{项目名}.git
▶ git fetch upstream
▶ git merge upstream/{不同步的分支}
向gerrit提交变更，review后将自动同步到gitlab
```

###  2. 缺少change id
change id是gerrit change的唯一标识。每一个change id代表一个code review的请求。在向gerrit push代码时，我们经常会遇到missing Change-Id in message footer的报错。主要有两个原因：
#### 2.1. git clone的时候没有选择带git hook的方式
每次git commit的时候git会自动执行.git/hooks/commit-msg脚本，为commit添加change id。这个.git/hooks/commit-msg脚本是在clone仓库的时候一起下载下来的。
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15588556985206.jpg"/></div>

如果没有使用带hook的地址clone代码，push时会报错：
```
▶ git push origin HEAD:refs/for/master
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 268 bytes | 268.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1)
remote: Processing changes: refs: 1, done
remote: ERROR: commit e74dfd3: missing Change-Id in message footer
remote:
remote: Hint: to automatically insert a Change-Id, install the hook:
remote:   gitdir=$(git rev-parse --git-dir); scp -p -P 29418 mengnan@172.17.202.23:hooks/commit-msg ${gitdir}/hooks/
remote: and then amend the commit:
remote:   git commit --amend --no-edit
remote:
To http://172.17.202.23:29419/a/test-webhook2
 ! [remote rejected] HEAD -> refs/for/master (commit e74dfd3: missing Change-Id in message footer)
error: failed to push some refs to 'http://mengnan@172.17.202.23:29419/a/test-webhook2'
```
如果想正确提交代码，需要根据报错提示执行操作：
```
▶ gitdir=$(git rev-parse --git-dir);scp -p -P 29418 {username}@gerrit.laiye.com:hooks/commit-msg${项目名}/hooks/
commit-msg                                                                                                                                                                100% 1521    70.3KB/s   00:00
▶ git commit --amend --no-edit #手动给前一个请求加上change id
[master a5822f8] add one line
 Date: Sun May 26 15:37:56 2019 +0800
 1 file changed, 1 insertion(+)
▶ git push origin HEAD:refs/for/master
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 311 bytes | 311.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1)
remote: Processing changes: refs: 1, new: 1, done
remote:
remote: SUCCESS
remote:
remote: New Changes:
remote:   http://172.17.202.23:29419/c/test-webhook2/+/24 add one line
To http://172.17.202.23:29419/a/test-webhook2
 * [new branch]      HEAD -> refs/for/master
```
#### 2.2. 某些操作不会自动添加change id
比如pull+自动merge的时候git hook加上了change id，然后又自动追加了merge冲突的文件名，这样change id不在最后一行，gerrit识别不出来会报错。
```
commit 7104c75d78562df974d6b60183d2b8d27cbced43 (HEAD)
Merge: e7a3fb5 6d6fa16
Author: mengnan <supernan1994@gmail.com>
Date:   Sun Jun 9 16:15:45 2019 +0800

    Merge remote-tracking branch 'origin/master' into HEAD
    
    Change-Id: I3b8e9ce79bbfd1e7ca698be597316e8e4f78b376
    
    # Conflicts:
    #       main.go
```
具体报错信息如下：
```
▶ git push origin HEAD:refs/for/master                                                                        
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 572 bytes | 572.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1)
remote: Processing changes: refs: 1, done    
remote: ERROR: commit c532445: missing Change-Id in message footer
remote: 
remote: Hint: run
remote:   git commit --amend
remote: and move 'Change-Id: Ixxx..' to the bottom on a separate line
remote: 
To ssh://172.17.202.23:29418/test-webhook
 ! [remote rejected] HEAD -> refs/for/master (commit c532445: missing Change-Id in message footer)
error: failed to push some refs to 'ssh://mengnan@172.17.202.23:29418/test-webhook'
```
如果想正确提交代码，需要根据报错提示执行操作：
```
▶ git commit --amend          
[detached HEAD b266908] Merge remote-tracking branch 'origin/master' into HEAD
 Date: Sun May 26 15:25:59 2019 +0800
▶ git push origin HEAD:refs/for/master
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 555 bytes | 555.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1)
remote: Processing changes: refs: 1, new: 1, done    
remote: 
remote: SUCCESS
remote: 
remote: New Changes:
remote:   http://172.17.202.23:29419/c/test-webhook/+/25 Merge remote-tracking branch 'origin/master' into HEAD
To ssh://172.17.202.23:29418/test-webhook
 * [new branch]      HEAD -> refs/for/master
```
#### 2.3. 批量提交多个commit，中间一个commit没加change id
经常会出现这样的情况：在pull代码之后，没有直接提交，直接基于pull的结果进行了修改。最终向gerrit提交代码时发现中间有一个commit没有change id。
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600842011227.jpg"/></div>

如下图所示，在这个例子中一共有三个commit需要提交，分别是ea24f1d，4f99a0e，37958d1。其中4f99a0e没有change id。
```
▶ git checkout local_branch2
▶ git log --oneline
37958d1 (HEAD -> master) 第三次提交
4f99a0e Merge remote-tracking branch 'origin/master'
7a1edf0 (origin/master, origin/HEAD) 第二次提交
ea24f1d 第一次提交
```

处理这样的情况，一般有两种方式：
##### 2.3.1. 多个commit需要分开提交到多个change
我们需要check到没有change id的commit，添加change id后手动提交。我们先提交ea24f1d和4f99a0e两个commit。
```
▶ git checkout 4f99a0e
# 创建一个新的分支对4f99a0e进行改动
▶ git checkout -b local_branch3
▶ git log --oneline
4f99a0e (HEAD) Merge remote-tracking branch 'origin/master'
7a1edf0 (origin/master, origin/HEAD) 第二次提交
ea24f1d 第一次提交
▶ git commit --amend --no-edit
[detached HEAD 2b9dbee] Merge remote-tracking branch 'origin/master'
 Date: Sun Jun 9 18:40:13 2019 +0800
 
▶ git push origin HEAD:refs/for/master
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 569 bytes | 569.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1)
remote: Processing changes: refs: 1, new: 2, done    
remote: 
remote: SUCCESS
remote: 
remote: New Changes:
remote:   http://172.17.202.23:29419/c/test-webhook/+/64 第一次提交
remote:   http://172.17.202.23:29419/c/test-webhook/+/65 Merge remote-tracking branch 'origin/master'
To ssh://172.17.202.23:29418/test-webhook
 * [new branch]      HEAD -> refs/for/master
```
看一下merge这个change的最后一个commit id，我们发现已经不是原来的4f99a0e，已经变成了2b9dbee。
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600842405895.jpg"/></div>

```
▶ git fetch "ssh://mengnan@172.17.202.23:29418/test-webhook" refs/changes/65/65/1 && git checkout FETCH_HEAD
▶ git log
commit 2b9dbeedda57338bee6799781fa26722247df36b (HEAD)
Merge: ea24f1d 7a1edf0
Author: mengnan <supernan1994@gmail.com>
Date:   Sun Jun 9 18:40:13 2019 +0800

    Merge remote-tracking branch 'origin/master'
    
    Change-Id: I66766de4aa2dcbae836e4bec13b84ce66d7f3efc
```

而第三次提交37958d1还是基于4f99a0e，而不是基于2b9dbee。我们需要基于2b9dbee重做第三次提交。

```
▶ git checkout local_branch2
▶ git rebase local_branch3
First, rewinding head to replay your work on top of it...
Applying: 第三次提交

▶ git push origin HEAD:refs/for/master
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
Delta compression using up to 8 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (8/8), 1.01 KiB | 1.01 MiB/s, done.
Total 8 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2)
remote: Processing changes: refs: 1, new: 1, done    
remote: warning: no changes between prior commit 2b9dbee and new commit f360771
remote: 
remote: SUCCESS
remote: 
remote: New Changes:
remote:   http://172.17.202.23:29419/c/test-webhook/+/66 第三次提交
remote: 
To ssh://172.17.202.23:29418/test-webhook
 * [new branch]      HEAD -> refs/for/master
```

##### 2.3.2. 多个commit提交到一个change
我们可以批量把commit rebase到一个commit。rebase多个commit的操作见3.2节讲解。

### 3. 合并commit
#### 3.1. 修改上一个commit
假设现在完成了一个需求，准备提交code review:

```
▶ git commit -m 'main commit'
▶ git log --oneline
674f8ca (HEAD -> master) main commit
66c6677 (origin/master, origin/HEAD) Merge "等待被修改的change"
```

在push之前突然发现了一个bug。如果正常提交，在gerrit上就会有两个change。reviewer在看第一个change的时候，不知道哪些是第二个已经修复的bug，哪些是没修复的，对比起来很麻烦。
我们提倡同一个需求、相互依赖的两个改动尽量放在一个change里。方便reviewer review，也会减少一些不必要的冲突。
第二个commit我们用--amend参数来修改上一个commit：

```
▶ git commit -m 'main commit and fix' --amend
[master 81598cd] main commit and fix
 Date: Sun May 26 16:51:35 2019 +0800
 1 file changed, 1 insertion(+)
 ▶ git log --oneline
81598cd (HEAD -> master) main commit and fix
66c6677 (origin/master, origin/HEAD) Merge "等待被修改的change"
```

这样就会作为一个change提交了

```
▶ git push origin HEAD:refs/for/master
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 604 bytes | 604.00 KiB/s, done.
Total 4 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1)
remote: Processing changes: refs: 1, new: 1, done    
remote: 
remote: SUCCESS
remote: 
remote: New Changes:
remote:   http://172.17.202.23:29419/c/test-webhook/+/29 main commit and fix
To ssh://172.17.202.23:29418/test-webhook
 * [new branch]      HEAD -> refs/for/master
```

#### 3.2. 合并多个commit
有时当我们git pull/git merge的时候，解决冲突后，会自动生成没有加change id的commit，夹在多个commit中间（见2.3节的情况）；或者commit忘记加--amend的时候，在提交gerrit的时候会变成多个changes。我们可以用rebase操作合并多个commit。

假如现在有三个commit(19e38cb, 4bea9f9, 710b142)我们希望合并为一个commit:

```
▶ git log --oneline
19e38cb (HEAD -> master) 等待被合并的提交3
4bea9f9 等待被合并的提交2
710b142 等待被合并的提交1
66c6677 (origin/master, origin/HEAD) Merge "等待被修改的change"
bfe60ee 等待被修改的change
743bc04 Merge "Merge branch 'master' of ssh://172.17.202.23:29418/test-webhook"
12e2855 Merge changes Ic0e4291c,I178ede01
460190e Merge remote-tracking branch 'origin/master' into HEAD
b266908 Merge remote-tracking branch 'origin/master' into HEAD
0b732d7 Merge branch 'master' of ssh://172.17.202.23:29418/test-webhook
51fde7e Merge "pl"
```

可以用rebase批量合并最近3个commit

```
▶ git rebase -i HEAD~3
pick 710b142 等待被合并的提交1
pick 4bea9f9 等待被合并的提交2
pick 19e38cb 等待被合并的提交3

# Rebase 66c6677..19e38cb onto 66c6677 (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
"~/Works/programs/SaaS/test-webhook/.git/rebase-merge/git-rebase-todo" 29L, 1253C
```

把pick修改为s(squash)

```
pick 710b142 等待被合并的提交1
s 4bea9f9 等待被合并的提交2
s 19e38cb 等待被合并的提交3
```

然后保存。本地会由上至下重做3个commit。

```
# This is a combination of 3 commits.
# This is the 1st commit message:

等待被合并的提交1

Change-Id: Ic34f22bbd03ff1f482fdbb559dce6ad86e9a9524

# This is the commit message #2:

等待被合并的提交2

Change-Id: I1ac2cd32b9894e0a7e21da726e031f96ecfeca2b

# This is the commit message #3:

等待被合并的提交3

"~/Works/programs/SaaS/test-webhook/.git/COMMIT_EDITMSG" 37L, 913C
```

成功合并：

```
▶ git rebase -i HEAD~3             
[detached HEAD 83835ff] 等待被合并的提交1
 Date: Sun May 26 16:05:13 2019 +0800
 1 file changed, 1 deletion(-)
Successfully rebased and updated refs/heads/master.
▶ git log --oneline
83835ff (HEAD -> master) 等待被合并的提交1
66c6677 (origin/master, origin/HEAD) Merge "等待被修改的change"
bfe60ee 等待被修改的change
743bc04 Merge "Merge branch 'master' of ssh://172.17.202.23:29418/test-webhook"
12e2855 Merge changes Ic0e4291c,I178ede01
460190e Merge remote-tracking branch 'origin/master' into HEAD
```

### 4. 修改已提交的change
开发者正常提交了一个change到gerrit，reviewer看过之后觉得有些地方需要修改，这个时候开发者需要修完之后继续提交新的patchset到原来的change里，方便reviewer对比。
主要分三种情况：  
1）修改的change是最后一个change，后面再没有修改了  
2）修改的change后面还有其他change，改完需要把这个修改同步到后面的change上，这种情况比较复杂  
3）修改的change后面还有其他change，改完需要把这个修改同步到后面的change上，但最新的改动和后面的change有冲突，复杂程度MAX  

#### 4.1 修改最新提交的change
假设现在正常提交了一个change：
```
▶ git commit -m '等待被修改的commit'
[master 68e2e71] 等待被修改的commit
 1 file changed, 1 deletion(-)
▶ git push origin HEAD:refs/for/master
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 331 bytes | 331.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1)
remote: Processing changes: refs: 1, new: 1, done    
remote: 
remote: SUCCESS
remote: 
remote: New Changes:
remote:   http://172.17.202.23:29419/c/test-webhook/+/31 等待被修改的commit
To ssh://172.17.202.23:29418/test-webhook
 * [new branch]      HEAD -> refs/for/master
```

在gerrit上能看到这个新的change。我们可以点击右下角update change获取修改change的提示。
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15588624296384.jpg"/></div>

按照提示操作：

```
▶ git fetch "ssh://mengnan@172.17.202.23:29418/test-webhook" refs/changes/31/31/1 && git checkout FETCH_HEAD
From ssh://172.17.202.23:29418/test-webhook
 * branch            refs/changes/31/31/1 -> FETCH_HEAD
Note: checking out 'FETCH_HEAD'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 68e2e71 等待被修改的commit
▶ git add main.go &&   git commit --amend --no-edit
[detached HEAD ca2b767] 等待被修改的commit
 Date: Sun May 26 17:17:08 2019 +0800
 1 file changed, 2 insertions(+)
▶ git push origin HEAD:refs/for/master
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 338 bytes | 338.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1)
remote: Processing changes: refs: 1, updated: 1, done    
remote: 
remote: SUCCESS
remote: 
remote: Updated Changes:
remote:   http://172.17.202.23:29419/c/test-webhook/+/31 等待被修改的commit
remote: 
To ssh://172.17.202.23:29418/test-webhook
 * [new branch]      HEAD -> refs/for/master
```

可以看到提交了新的patchset

<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15588625928426.jpg"/></div>

可以点击对比，查看第二次修改的内容
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15588626393027.jpg"/></div>

#### 4.2 有其他change依赖当前修改的change（没有冲突）
假设我们连续提交了三个change，第一个有bug。

<div align="center"><img width="70%" height="70%" src="2019-06-09-gerrit-in-action/15600967883166.jpg"/></div>

| **Patch1** | **change1** | **change2** | **change3** |
| ------ | ------ | ------ | ------ |
| 文件A | A1 | A1 | A1 |
| 文件B | B1 | **B2** | **B2** |
| 文件C | C1 | C1 | **C2** |
| **Patch2** | **change1** | **change2** | **change3** |
| ------ | ------ | ------ | ------ | 
| 文件A | - | - | - |
| 文件B | - | - | - |
| 文件C | - | - | - |

```
▶ git log --oneline
96dc449 (HEAD -> master) 第三次提交
8b256f6 第二次提交
7abedfb 第一次提交，有bug
5fda3c0 (origin/master, origin/HEAD) Merge remote-tracking branch 'origin/master'

▶ git push origin HEAD:refs/for/master
Enumerating objects: 11, done.
Counting objects: 100% (11/11), done.
Delta compression using up to 8 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (9/9), 900 bytes | 900.00 KiB/s, done.
Total 9 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3)
remote: Processing changes: refs: 1, new: 3, done    
remote: 
remote: SUCCESS
remote: 
remote: New Changes:
remote:   http://172.17.202.23:29419/c/test-webhook/+/79 第一次提交，有bug
remote:   http://172.17.202.23:29419/c/test-webhook/+/80 第二次提交
remote:   http://172.17.202.23:29419/c/test-webhook/+/81 第三次提交
To ssh://172.17.202.23:29418/test-webhook
 * [new branch]      HEAD -> refs/for/master
```

<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600953321479.jpg"/></div>

我们按照4.1节的操作，修复这个bug。

| **Patch1** | **change1** | **change2** | **change3** |
| ------ | ------ | ------ | ------ |
| 文件A | A1 | A1 | A1 |
| 文件B | B1 | **B2** | **B2** |
| 文件C | C1 | C1 | **C2** |
| **Patch2** | **change1** | **change2** | **change3** |
| ------ | ------ | ------ | ------ | 
| 文件A | **A2** | - | - |
| 文件B | B1 | - | - |
| 文件C | C1 | - | - |

<div align="center"><img width="70%" height="70%" src="2019-06-09-gerrit-in-action/15600973194533.jpg"/></div>
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600954825090.jpg"/></div>

会发现剩下的change2, change3旁边出现了神奇的绿色字体(Indirect ancestor)，点击进入change2也会看到神奇的红色字体(Not current)。

<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600955437098.jpg"/></div>

如果这时合并了change1，会发现change2冲突了，即使change1和change2没有修改同一行代码。这是因为对于change1 patch2修改的代码行，已经合并合并的change1 patch2的代码和change2上的代码是不一样的。我们可以在合并change1之前把change2和change3调整为基于change1的patch2，否则合并后解决冲突就请参照4.3吧。

<div align="center"><img width="70%" height="70%" src="2019-06-09-gerrit-in-action/15600968431547.jpg"/></div>

我们可以进入change2页面，点击右上角rebase，大功告成。

| **Patch1** | **change1** | **change2** | **change3** |
| ------ | ------ | ------ | ------ |
| 文件A | A1 | A1 | A1 |
| 文件B | B1 | **B2** | **B2** |
| 文件C | C1 | C1 | **C2** |
| **Patch2** | **change1** | **change2** | **change3** |
| ------ | ------ | ------ | ------ | 
| 文件A | **A2** | **A2** | **A2** |
| 文件B | B1 | **B2** | **B2** |
| 文件C | C1 | C1 | **C2** |

<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600958557818.jpg"/></div>
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600958676482.jpg"/></div>

同样处理change3：

<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600959684725.jpg"/></div>
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600962448520.jpg"/></div>


#### 4.3 有其他change依赖当前修改的change（发生冲突）

| **Patch1** | **change1** | **change2** | **change3** |
| ------ | ------ | ------ | ------ |
| 文件A | A1 | **A2** | **A3** |
| **Patch2** | **change1** | **change2** | **change3** |
| ------ | ------ | ------ | ------ | 
| 文件A | **A4** | - | - |

<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600961104444.jpg"/></div>

只要change1的patch2和change2的patch1同时对一行做了修改，gerrit会认为是冲突的。  
- 当change1还没有merge的情况下，在gerrit页面上基于change2 rebase会提示冲突。
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600975723611.jpg"/></div>
- 当change1已经merge的情况下，change2页面上会直接显示冲突。submit时提示无法合并。
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600906225352.jpg"/></div>

我们需要分别在change2和change3上rebase change1最新的改动，在rebase过程中解决冲突。
先解决change2的冲突：

| **Patch1** | **change1** | **change2** | **change3** |
| ------ | ------ | ------ | ------ |
| 文件A | A1 | **A2** | **A3** |
| **Patch2** | **change1** | **change2** | **change3** |
| ------ | ------ | ------ | ------ | 
| 文件A | **A4** | **merge(A2&A4) = A5** | - |

<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600961466966.jpg"/></div>

```
# check到change2上，创建一个新本地分支local_branch3
▶ git fetch "ssh://mengnan@172.17.202.23:29418/test-webhook" refs/changes/74/74/1 && git checkout FETCH_HEAD
From ssh://172.17.202.23:29418/test-webhook
 * branch            refs/changes/74/74/1 -> FETCH_HEAD
Note: checking out 'FETCH_HEAD'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at a343549 第二次提交
▶ git checkout -b local_branch3
Switched to a new branch 'local_branch3'

# 在change2上应用change1的fix
▶ git rebase -i local_branch2
# 此处省略一系列merge的操作
▶ git rebase --continue
Successfully rebased and updated refs/heads/local_branch3.

▶ git push origin HEAD:refs/for/master
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 327 bytes | 327.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: Processing changes: refs: 1, updated: 1, done    
remote: 
remote: SUCCESS
remote: 
remote: Updated Changes:
remote:   http://172.17.202.23:29419/c/test-webhook/+/74 第二次提交
remote: 
To ssh://172.17.202.23:29418/test-webhook
 * [new branch]      HEAD -> refs/for/master
```

第二次提交的冲突已经解决：
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600923109691.jpg"/></div>

我们用同样的方法解决change3的冲突，不过这次是需要change3上应用change2。

| **Patch1** | **change1** | **change2** | **change3** |
| ------ | ------ | ------ | ------ |
| 文件A | A1 | **A2** | **A3** |
| **Patch2** | **change1** | **change2** | **change3** |
| ------ | ------ | ------ | ------ | 
| 文件A | **A4** | **merge(A2&A4) = A5** | **merge(A3&A5) = A6** |

<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600961693214.jpg"/></div>

```
# check到change3上，创建一个新本地分支local_branch4
▶ git fetch "ssh://mengnan@172.17.202.23:29418/test-webhook" refs/changes/75/75/1 && git checkout FETCH_HEAD
From ssh://172.17.202.23:29418/test-webhook
 * branch            refs/changes/75/75/1 -> FETCH_HEAD
Note: checking out 'FETCH_HEAD'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at fa14773 第三次提交
▶ git checkout -b local_branch4
Switched to a new branch 'local_branch4'

# 在change3上应用change2的fix
▶ git rebase -i local_branch3
# 此处省略一系列merge的操作
▶ git rebase --continue
Successfully rebased and updated refs/heads/local_branch4.

▶ git push origin HEAD:refs/for/master                                                                      
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 8 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (5/5), 620 bytes | 620.00 KiB/s, done.
Total 5 (delta 0), reused 0 (delta 0)
remote: Processing changes: refs: 1, updated: 1, done    
remote: 
remote: SUCCESS
remote: 
remote: Updated Changes:
remote:   http://172.17.202.23:29419/c/test-webhook/+/75 第三次提交
remote: 
To ssh://172.17.202.23:29418/test-webhook
 * [new branch]      HEAD -> refs/for/master
```
去gerrit的页面上看，第三次提交也不冲突了。
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600926045904.jpg"/></div>

### 5. 发生冲突
4.2节中我们主要讨论了对parent change改动引起的冲突，本节我们主要讨论多人改动引起的冲突。

| **Patch1** | **change1** | **change2** |
| ------ | ------ | ------ |
| 文件A | A1 | A2 |
| **Merge** | **change1** | **change2** |
| ------ | ------ | ------ | ------ | 
| 文件A | success | conflict |

<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600995242556.jpg"/></div>
当两个人同时基于一个分支进行修改，提交了两个冲突的change。
- 未合并时，gerrit会提示：
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600983415180.jpg"/></div>
- 当其中一个合并之后，另一个会提示冲突：
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600984922812.jpg"/></div>

不管有没有合并，我们都可以用同样的方式解决冲突。

| **Patch1** | **change1** | **change2** |
| ------ | ------ | ------ |
| 文件A | A1 | A2 |
| **Merge** | **change1** | **change2** |
| ------ | ------ | ------ | ------ | 
| 文件A | success | conflict |
| **Patch2** | **change1** | **change2** |
| ------ | ------ | ------ |
| 文件A | - | **merge(A1&A2) = A3** |

<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600995533529.jpg"/></div>

```
# 下载change1（未合并）或master（已合并）
▶ git fetch "ssh://mengnan@172.17.202.23:29418/test-webhook" refs/changes/88/88/1 && git checkout FETCH_HEAD
From ssh://172.17.202.23:29418/test-webhook
 * branch            refs/changes/88/88/1 -> FETCH_HEAD
Note: checking out 'FETCH_HEAD'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 514d1c5 第一个人的改动
▶ git checkout -b local_branch1  
Switched to a new branch 'local_branch1'

# 在change2上应用change1
▶ git rebase local_change1            
# 省略merge操作
▶ git rebase --continue
Applying: 第二个人的改动

▶ git push origin HEAD:refs/for/master
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 352 bytes | 352.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2)
remote: Processing changes: refs: 1, updated: 1, done    
remote: warning: 15787e8: no files changed, was rebased
remote: 
remote: SUCCESS
remote: 
remote: Updated Changes:
remote:   http://172.17.202.23:29419/c/test-webhook/+/87 第二个人的改动
remote: 
To ssh://172.17.202.23:29418/test-webhook
 * [new branch]      HEAD -> refs/for/master
```

成功解决冲突：
<div align="center"><img width="100%" height="100%" src="2019-06-09-gerrit-in-action/15600989729740.jpg"/></div>


## 五、参考资料
- http://wincent.github.io/gerrit-best-practices-tech-talk/assets/fallback/index.html
- https://www.algoclinic.com/gerrit-best-practices.html
- https://juejin.im/post/5c2d7377518825544d43dfa5#heading-39

