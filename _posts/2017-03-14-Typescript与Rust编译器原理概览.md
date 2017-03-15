---
layout: post
title: "Typescript与Rust编译器类型检查概览"
date: 2017-03-14 22:28:19 +0800
image: 'blog-author.jpg'
description: '简单介绍Typescript与Rust这两门前端常用语言（误）的编译器类型检查原理之异同'
main-class: 'frontend'
color: '#66CCFF'
tags:
- typescript
- rust
- WIP
categories: Review
twitter_text:
introduction: '编译原理课作业'
---
# Typescript与Rust编译器原理概览

TypeScript 是一门 2017 年的前端常用的「编译到JS语言」（Compile-to-js-language），它是 ECMAScript6+ 的超集，以强大的静态类型检查闻名，其在前端界最大的竞争对手 FlowType 以相同的语法提供运行时 I/O 边界检查，皆是前端工程化开发的强大工具。
  
Rust 是一门 2017 年的前端不常用的「编译到[WebAssembly](https://webassembly.github.io/)语言」，其语法与 TypeScript 有一定的相似性，提供和 TypeScript 类似的、循循善诱的类型检查，和声声泪下用法劝告，因而受到人民群众的拥护和喜爱。
  
这两门语言都以类型检查著称，本文对比这两门语言于之异同于此，透明办事程序，让干群相互了解。编译工作效率显能力，办事公开透明落实处。
