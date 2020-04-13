---
layout: post
title: '用TiddlyWiki替代Notion和EverNote作为个人知识管理系统'
date: 2020-04-13 15:58:31 +0800
image: 'blog-author.jpg'
description: '不仅免费，而且好用还好看，那它的缺点是什么？'
main-class: 'memo'
color: '#9E9E9E'
tags:
  - memo
categories: Journal
twitter_text:
introduction: '配置免费云同步的非线性知识管理系统 TiddlyWiki，身兼私有笔记系统和公开 Wiki 二职'
---

我曾经用过很多年的印象笔记，里面装着四处收集来的碎片内容，比如[什么是「共产中文腔调」？ - 调查类问题 - 知乎](http://www.zhihu.com/question/19687065)、我高考的分数截图等等；我也用了几年的 Notion，用它记待办事项、给认识的人写备忘小传；我还用了很久的 Anki，用它把英语单词、数学公式、算法套路等碎片知识装进脑中。

但这些工具似乎都对两个对我来说很关键的概念缺乏关注：「元信息」和「自动化」。我认为，只有能充分保存元信息的自动化的知识管理系统才能称得上「好用」。

我关注一个信息是什么，它从哪里来，要到哪里去……也就是关于信息的信息：标签、源谱、关联。我们的大脑会保存和一个信息相关的很多其他信息，而如果一个知识管理系统也同样保存了这些元信息，就可以用很贴近我们大脑的形式存储知识，让我们查找、利用知识都更加便捷。

自动化则能减少很多复制黏贴文本的劳作，还能让信息被以各种方式聚合，出现在更多地方，更容易被找到。

这么说来，个人知识管理系统似乎有一个不可能三角：**免费-好用-好看，只能拥有其中两样**。Notion 要付费、印象笔记不好用、Quip和飞书等系统不是为个人服务的……

然而 [TiddlyWiki](https://tiddlywiki.com/) 似乎打破了这个不可能三角，引入了第四个元素：它 **免费、好用，还蛮好看** ，就是需要具备动手能力才能把积木式的它拼成自己想要的形状。

![截图 - Tiddlywiki 桌面应用：桌面版笔记工具](https://raw.githubusercontent.com/linonetwo/linonetwo.github.io/master/assets/img/posts/tiddlywiki/tiddlywiki-desktop-nodejs-webcatalog.png)
（↑图：Tiddlywiki 桌面应用：桌面版笔记工具）

![截图 - Tiddlywiki 桌面应用：目录栏快速搜索小工具](https://raw.githubusercontent.com/linonetwo/linonetwo.github.io/master/assets/img/posts/tiddlywiki/menubar-tiddlywiki-quick-search-tool.png)
（↑图：Tiddlywiki 桌面应用：目录栏快速搜索小工具）

## 积木式的知识管理系统

TiddlyWiki 是一个自由的软件，

## 重构：改善既有文档的设计

在我构思一篇文章时，我的思绪经常在记忆的网上。

大脑和知识管理系统是不同步的，很多内容并不存在于机器里，而只存在于大脑中。先用一个链接占坑

```
<<reuse-tiddler "允许用git-sync脚本自动同步代码">>
```

![截图 - 通过 Transclusion 稍后再编辑具体内容](https://raw.githubusercontent.com/linonetwo/linonetwo.github.io/master/assets/img/posts/tiddlywiki/edit-later.png)

## 与其他知识利用系统联动
