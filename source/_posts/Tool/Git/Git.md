---
title: Git
date: 2022-04-25 12:50:03
tags: [Tool]
categories: Tool
math: true
---

# Git

  Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。

## Install

```bash
sudo apt-get install libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev
```

```bash
sudo apt-get install git
```

## WorkFlow

- 克隆 Git 资源作为工作目录。
- 在克隆的资源上添加或修改文件。
- 如果其他人修改了，你可以更新资源。
- 在提交前查看修改。
- 提交修改。
- 在修改完成后，如果发现错误，可以撤回提交并再次修改并提交。

<img src="/image/Git/1.png" alt="2" style="zoom:100%;" />

## Create Repository

### `git init`

使用当前目录作为 Git 仓库，我们只需使它初始化。

```bash
git init
```

初始化后，会在 newrepo 目录下会出现一个名为 .git 的目录，所有 Git 需要的数据和资源都存放在这个目录中。

如果当前目录下有几个文件想要纳入版本控制，需要先用 git add 命令告诉 Git 开始对这些文件进行跟踪，然后提交：

```bash
git add *.c
git add README
git commit -m '初始化项目版本'
```

以上命令将目录下以 .c 结尾及 README 文件提交到仓库中。

### `git clone`

我们使用 **git clone** 从现有 Git 仓库中拷贝项目。

```bash
git clone <repo>
```



## Replenish

| Command           | Description                            |
| ----------------- | -------------------------------------- |
| `git add`         | 添加文件到暂存区                       |
| `git status`      | 查看仓库当前的状态，显示有变更的文件   |
| `git diff`        | 比较文件的不同，即暂存区和工作区的差异 |
| `git commit`      | 提交暂存区到本地仓库                   |
| `git reset`       | 回退版本                               |
| `git revert`      | 回退版本                               |
| `git rm`          | 删除工作区文件                         |
| `git mv`          | 移动或重命名工作区文件                 |
| `git log`         | 查看历史提交记录                       |
| `git blame<file>` | 以列表形式查看指定文件的历史修改记录   |
| `git remote`      | 远程仓库操作                           |
| `git fetch`       | 从远程获取代码库                       |
| `git pull`        | 下载远程代码并合并                     |
| `git push`        | 上传远程代码并合并                     |

