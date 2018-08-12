---
layout: post
title: git rebase --skip后commit丢失的拯救办法
---

今天从git仓库同步代码，执行 `git pull --rebase origin master` 命令后出现了冲突，解决冲突后执行 `git rebase --continue` 时提示没有改动，执行 `git rebase --abort` 或 `git rebase --skip` 命令。当时也不是太明白什么意思，stackoverflow搜了下，说是可以执行 `git rebase --skip` 命令。本着我对git强大恢复功能的信任，我毫不犹豫地执行了skip命令。然后...然后杯具的事情就发生了，之前一个commit修改的内容没了，就这样没了，可是我辛辛苦苦几个晚上写的东西呢，肿么可以这样子...  
然而，我肯定是不甘心辛苦创作的内容就这么没了的现实，开始找办法恢复。这时候 `git reflog` 就派上了用场，用这个命令看看最近发生了什么

![git reflog]({{ site.baseurl }}/images/git_reflog.jpg)

找到rebase前的一个commit, 执行命令 `git reset d2293c1`，本以为这样就大功告成，结果确实图样图森破，之前的修改还是没有找到。这下子我有点急了，怎么会没有呢？再往前reset两个commit试试，结果还是这样子。怎么办？怎么办？怎么办？那就google啊，然后找到了下面的办法：  
1. 先执行 `git reflog` 命令找到rebase前的一个commit，也就是上图中的 `d2293c1` 这个commit
2. 执行 `git checkout -b recovery d22293c1` 创建一个新的分支
3. 切到 `master` 分支，然后执行 `git merge recovery` 命令合并 `recovery` 分支，大功告成
