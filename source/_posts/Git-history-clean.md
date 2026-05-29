---
title: 关于怎么从Git历史里删除文件
author: CYandYUE
index_img: /index_img/36_git_rm_log.png
date: 2026-05-29 21:40:28
tags:
- Git
- Blog
categories:
- Git
- Blog
---

## 0. Motivation
今天写博客的时候遇到一个问题

我之前把一张图片加进了 commit

后来发现图片不清晰，想换成另一张新的图片

如果我只是删掉旧图片，再 commit 一张新图片

那旧图片是不是还会永远留在 Git 历史里？


## 1. Overall
Git 保存的每个 commit 都记录了当时仓库的一个快照

所以只要某个文件进入过某个 commit，它就属于仓库历史的一部分

即使后面的 commit 把这个文件删掉了，它也只是从“当前版本”里消失

但在之前的 commit 里，它仍然存在

比如：

```bash
commit A: add old.jpg
commit B: remove old.jpg, add new.png
```

在最新的 `commit B` 里看不到 `old.jpg`

但是切回 `commit A`，还是能看到它

```bash
git checkout commit_A
```

这也是为什么如果不小心提交了大文件、隐私文件、token，单纯 `git rm` 是不够的

## 2. Case 1: 还没有 push
如果这个 commit 只是存在本地，还没有 push 到 GitHub

那就比较好办

可以直接修改最后一个 commit

假设旧文件是：

```bash
source/index_img/old.jpeg
```

新文件是：

```bash
source/index_img/new.png
```

先从 Git 当前版本中删除旧文件：

```bash
git rm source/index_img/old.jpeg
```

然后加入新文件和修改过的文章：

```bash
git add source/index_img/new.png source/_posts/xxx.md
```

把这些修改合并进上一个 commit：

```bash
git commit --amend
```

这样新的最后一个 commit 里就只有 `new.png`

从正式历史来看，`old.jpeg` 就没有出现过

不过本地 `.git` 里可能还会暂时有旧对象，比如 reflog 还记着旧 commit

一般不需要管，后面 Git 自己会清理

## 3. Case 2: 已经 push
如果已经 push 到远端仓库了

那就要分情况

如果只是普通的小文件，比如一张几 KB 的图片

那其实问题不大

直接新建一个 commit 删除它就行：

```bash
git rm source/index_img/old.jpeg
git add source/index_img/new.png source/_posts/xxx.md
git commit -m "replace old image"
git push
```

这样最新版本里不会再有旧图片

只是 Git 历史里仍然会保留它

如果是大文件，或者隐私文件，比如密码、token、私钥

那就不能只靠 `git rm`

需要重写历史

常见工具是：

```bash
git filter-repo
```

或者：

```bash
BFG Repo-Cleaner
```

重写历史之后还需要 force push：

```bash
git push --force
```

这个操作会影响已经 clone 过仓库的人

所以如果是多人协作项目，要非常谨慎

## 4. Check
可以用下面命令看看某个文件有没有出现在历史里：

```bash
git log --oneline -- source/index_img/old.jpeg
```

如果有输出，说明这个文件至少出现在某个 commit 里

也可以看当前仓库状态：

```bash
git status -sb
```

如果显示：

```bash
## master...origin/master [ahead 1]
```

说明本地比远端多了 1 个 commit

也就是这个 commit 还没 push

这时候用 `git commit --amend` 修改最后一个 commit，一般是很舒服的选择

## 5. Summary
简单总结：

- `git rm` 只是让文件从当前版本消失
- 只要文件进入过 commit，它就会存在于历史里
- 还没 push 的 commit，可以用 `git commit --amend` 改掉
- 已经 push 的普通小文件，通常直接新 commit 删除即可
- 已经 push 的大文件或隐私文件，需要重写历史

所以 Git 里真正需要紧张的不是普通废图

而是大文件和敏感信息
