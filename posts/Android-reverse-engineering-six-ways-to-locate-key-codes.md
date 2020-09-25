---
title: Android reverse engineering six ways to locate key codes
description:
date: 2019-09-19
tags:
  - python
layout: layouts/post.njk
image:
---
## 1. 信息反馈法

先运行目标程序，利用特定的反馈信息作为突破口，如错误信息字符串。字符串一般存储在 `String.xml` 或硬编码中，如果是前者，反汇编代码通过调用 `id` 的形式访问。可用 `apktool` 查看 `String.xml` ，获取 `id` , 后者直接在反汇编代码中搜索。

## 2. 特征函数法

搜索特殊的函数如 `Toast.MakeText.Show()`

## 3. 顺序查看法

根据软件执行顺序依次查看，常用于病毒分析。

## 4. 代码注入法

手动修改 `apk` 文件的反汇编代码，加入 `Log` ，配合 `LogCat` 查看运行信息，常用于解密程序数据。

## 5. 栈跟踪法

动态调试方法，输出栈跟踪信息，查看函数调用序列理解方法执行流程。

## Method Profiling（方法剖析）

动态调试方法，常用于热点分析和性能优化，能记录每个函数占用CPU的时间和跟踪所有函数调用关系，提供比栈跟踪更详细的函数调用序列。

摘自非虫的《Android安全与逆向分析》。
