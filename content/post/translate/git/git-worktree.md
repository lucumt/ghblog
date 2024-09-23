---
title: "[译]Git Worktree使用说明"
date: 2024-08-23T13:17:18+08:00
lastmod: 2024-08-23T13:17:18+08:00
draft: false
keywords: ["Git","Worktree"]
description: "翻译一篇国外的关于Git Worktree使用的博文，主要用于解决多分支开发时切换不方便的问题"
tags: ["Git"]
categories: ["翻译"]
author: "Rosen Lu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: false
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false
centerImage: false
borderImage: true

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

mermaidDiagrams: 
  enable: false
  options: ""

highchartsDiagrams: 
  enable: false
  options: ""
---

本文翻译自[**Git Worktree**](https://fev.al/posts/git-worktree/)

<!--more-->

我们或许会面临这样的情形：正在开发某个功能时一个bug出现并且需要及时修复、处于一个耗时的构建过程中突然需要在另外一个分支做快速修复、发现了一个不相关的bug想在不污染当前分支的前提下修复。不论哪种情况，通常我们都想再次`clone`一个仓库到本地，然而`Git`已经提供了对应的工具。

这就是`worktree`，通过它我们可在不必多次`clone`的前提下通过不同的分支创建不同的工作空间。

可通过`git worktree add ../my_second_worktree the_other_branch`来创建一个新的`worktree`，该指令会生成一个名为`my_second_worktree`的新文件夹并使用``the_other_branch`分支中的代码，之后可在里面独立运行并使用各种常规的`git`指令。

可通过下述指令来快速查看创建的所有`worktree`

```bash
$ git worktree list
/Users/charlesfeval/git/my_repo                   bd502b1 [user/chfeval/240614_that_feature]
/Users/charlesfeval/git/my_repo--other_worktree   15d0988 [user/chfeval/240616_fix_versionin]
/Users/charlesfeval/git/my_repo--bugfix_2  9c26708 [user/chfeval/240617_fix_rockin]
```

如果已经用`worktree`完成了相关操作，可使用类似` git worktree remove ../my_second_worktree`来删除。

下面是[官方文档](https://git-scm.com/docs/git-worktree)上关于该指令的说明

```bash
usage: git worktree add [-f] [--detach] [--checkout] [--lock [--reason <string>]]
                        [-b <new-branch>] <path> [<commit-ish>]
   or: git worktree list [-v | --porcelain [-z]]
   or: git worktree lock [--reason <string>] <worktree>
   or: git worktree move <worktree> <new-path>
   or: git worktree prune [-n] [-v] [--expire <expire>]
   or: git worktree remove [-f] <worktree>
   or: git worktree repair [<path>...]
   or: git worktree unlock <worktree>
```

