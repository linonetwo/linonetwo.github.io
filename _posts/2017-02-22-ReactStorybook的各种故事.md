---
layout: post
title: "ReactStorybook的各种故事"
date: 2017-02-22 22:20:19 +0800
image: 'blog-author.jpg'
description: '讲述各种各样的组件的故事'
main-class: 'frontend'
color: '#66CCFF'
tags:
- GraphQL
- kadira
- React
- storybook
categories: Review
twitter_text:
introduction: '用 @kadira/Storybook 迭代开发各种各样的 React 组件'
---
# ReactStorybook 与 Redux 的异步故事

React 开发的单页面应用，顾名思义，有着很多很多的页面（。）应用启动时可能会有启动页，然后根据本地存储中有没有存在 token 决定是否跳转到登录页面，登录后则可能有多 tab 切换的页面、点击列表项跳转的目录页等等。
  
我们使用 react-router 等路由框架在应用的各个页面之间切换，本质上是点击按钮等事件导致路由状态改变后，路由框架根据应用当前的状态渲染出不同的页面。
  
那么如果我们需要修改一个很深很深的组件里的样式，我们是不是要一次一次戳按钮点进去？当你修改了这个很深很深的组件，发现样式热替换挂了，这有时候是玄学，你需要手动刷新整个应用，你是否肯任劳任怨地重新戳回去一次？一次又一次？
  
![deep nested component dev](https://raw.githubusercontent.com/linonetwo/linonetwo.github.io/master/assets/img/posts/reduxstorybook/component%20tree.png)

如果产品需求改了，你要改的是数据流逻辑，而改完发现热替换不支持你正在修改的部分，那感觉肯定烦死了。要是有一种方法能直接跳到你要修改的组件身边就好了。

> 为什么不直接用 react-router 跳到你要去的组件那边？
  
因为有的人不用 react-router，有的人用 react-router v4。比如在 React Native 环境下，有的人用```react-native-router-flux``` 有的人用```react-router-native``` 有的人用```react-router-addon-controlled(deprecated)```，所以我们最好用一个和这些东西无关的第三方框架来辅助我们的开发。

![React Native Storybook](https://github.com/storybooks/react-native-storybook/raw/master/docs/assets/readme/screenshot.png)

## 故事书

```@kadira/Storybook```就是一个这样的框架，可以应用在 web 端和 native 端，将每个组件看作一个故事，然后用 Storybook 左边的书签在故事之间快速跳转。

## Atomic Design

2013 年 Brad Frost 提出的 Atomic Design（分级设计）建议我们创建系统中一个个小型、独立、可重用的元素，然后再组合为整个界面，而不是一次设计单个页面。

![Atomic Design](https://raw.githubusercontent.com/linonetwo/linonetwo.github.io/master/assets/img/posts/reduxstorybook/Atomic%20Design.png)
[[1]](#1)

这是一个比较抽象的，面对设计师的概念。当然它曾经有一些框架可以使用，但是比较难用 [[2]](#2)（可能是我耐心不足）。但它的愿景现在可以由 ```@kadira/Storybook``` 来实现，看下图，我给出了两个 Molecule 级别的组件，它们由数个从别的 UI 库中 import 进来的原子组件合成：



## 参考

### [<span id="1">Atomic Design</span>](http://bradfrost.com/blog/post/atomic-web-design/)

### [<span id="2">Atomic Design 的一个实现：patternlab</span>](http://patternlab.io/)

