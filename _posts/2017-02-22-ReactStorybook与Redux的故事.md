---
layout: post
title: "ReactStorybook与Redux的故事"
date: 2017-02-22 22:20:19 +0800
image: 'blog-author.jpg'
description: '讲述异步故事'
main-class: 'frontend'
color: '#66CCFF'
tags:
- Redux
- kadira
- React
- storybook
categories: Review
twitter_text:
introduction: '用 @kadira/Stoeybook 迭代开发 React 组件之余，要是能给它们注入来自 Web API 的异步数据就好了'
---
# ReactStorybook 与 Redux 的异步故事

## Storybook 好处都有啥

React 开发的单页面应用，顾名思义，有着很多很多的页面（。）
  
应用启动时可能会有启动页，然后根据本地存储中有没有存在 token 决定是否跳转到登录页面，登录后则可能有多 tab 切换的页面、点击列表项跳转的目录页等等。我们使用 react-router 等路由框架在应用的各个页面之间切换，本质上是路由框架根据应用当前的状态渲染出不同的页面。
  


![React Native Storybook](https://github.com/storybooks/react-native-storybook/raw/master/docs/assets/readme/screenshot.png)

## 参考

