---
layout:     post
title:      "Tales of FAVICONS and Caches: Persistent Tracking in Modern Browsers"
subtitle:   "Favicon和缓存的故事: 现代浏览器中的持久追踪"
date:       2021-04-08
author:     "风沐白"
catalog: true
tags:
    - 论文阅读
    - Web安全
    - NDSS
    - 浏览器
---

## 1. 来源
[Solomos K, Kristoff J, Kanich C, et al. Persistent Tracking in Modern Browsers[C]//2021 Network and Distributed System Security Symposium. Network & Distributed System Security Symposium, 2021.](https://www.ndss-symposium.org/ndss-paper/tales-of-favicons-and-caches-persistent-tracking-in-modern-browsers/)

## 2. 概述

现代浏览器对于追踪用户的限制越来越严格, 而作者发现了一种利用favicon (收藏夹图标) 缓存来追踪用户的方法.
这种缓存是长期的, 而且在隐身模式下仍然可以访问, 所以可以做到持久的跨越隔离的追踪.

## 3. 背景

由于HTTP协议是无状态的, 所以诞生了cookie以帮助维护会话状态, 但同时cookie也成为了网站追踪用户的手段.
现代浏览器为了保护用户隐私, 在逐步限制cookie的使用, 甚至在可预见的未来中弃用cookie.
于是, 越来越多的攻击者和研究人员开始探索*cookie-less*的追踪方式. 尽管现代浏览器采取了诸多措施来保护用户隐私, 但依然存在一些很简单的漏洞.

**Favicon** 网页图标, 全称为*favorites icon*.
最初用于在IE浏览器的收藏夹中作为网页的图标, 现在可以在任意网页的头部加载favicon.
现代浏览器将favicon存储在一个单独的本地数据库*Favicon Cache*, 每个icon都有属于自己的缓存条目, 包括: URL, icon ID, TTL.
这些缓存的默认存活期限为6小时, 最长可达1年, 而且是受网站控制的.

[Favicon 示例](https://raw.githubusercontent.com/entr0pia/entr0pia.github.io/master/img/in-post/2021-04-08/favicon_examples.jpg)

**缓存** 只要favicon的数据格式正确, 任何网站都可以将icon写入缓存.
缓存中的icon ID与网页中的不一致或者过期时, 浏览器就会更新这些缓存.
缓存以用户为单位进行存储, 而且不会因清除cookie/历史/缓存而失效.
对于当前用户, 缓存的读写权限是全局的.
而隐身模式下, 仍具有读权限.

## 4. 威胁模型

假设用户会在正常模式下浏览至少一次攻击者的网站, 而且启用了JavaScript, 尽管理论上此攻击模式也适用于禁用了JS的情况.
攻击者创建了一种新的用户标识符, 通过站内一系列的重定向链接, 将标识符存储在Favicon缓存中.

## 5. 系统设计

总的来说, 首先为每位用户生成一个二进制的ID.
同时站内有一系列的重定向链接, 每个链接有着不同的favicon.
icon在缓存内表示1, 不在缓存表示0.
假设ID长为4 bit, 则需要4个子路径, 记作*A, B, C, D*, 每个路径都有不同的icon.
若```id=0b1001```, 则缓存内有且仅有icon *A*和*D*.
以此实现对ID的存储和追踪.

### A. 写入模式

需要确保的是, 这是用户第一次访问攻击者的网站.
作为网站的所有者, 可以观察到用户的访问路径和请求资源.
攻击者设置了一个特定的icon, 用于标志是否为第一次访问.
当用户请求这一icon时, 说明其为第一次访问, 触发写入模式.

在写入模式中, 为用户生成一个n bit的二进制ID.
然后将用户重定向到bit为1的子路径中, 将icon写入缓存; 而bit为0的对应的子路径被跳过.
重定向完成后, 用户的ID被存储到favicon缓存中.

[写入模式的请求时序图](https://raw.githubusercontent.com/entr0pia/entr0pia.github.io/master/img/in-post/2021-04-08/write.jpg)

### B. 读取模式

当攻击者没有观察到用户对**写入模式**中提到的特定icon时, 说明该用户已经访过本站点, 并存储了ID, 触发读取模式.

站点通过JS脚本控制用户重定向到所有的子路径.
攻击者若没有观察到某一子路径的icon被请求, 说明用户ID的该bit为1; 有请求则为0.
同时, 站点对该icon的请求返回404代码, 这样就不会影响到用户ID.
换言之, 在读取模式中, 用户ID始终不变.

[读取模式的请求时序图](https://raw.githubusercontent.com/entr0pia/entr0pia.github.io/master/img/in-post/2021-04-08/read.jpg)

为保证正确处理并发请求, 站点还是额外设置了一个cookie.
但这一cookie不是为了追踪用户, 仅仅标识同一会话.
而且作为第一方cookie, 该cookie也不会被拦截.

### C. 动态扩展

ID越长, 则重定向所需时间越多.
为了减小开销, 当n bit不足以标记所有用户时, ID自动扩展为*n+1* bit, 且追加在低地址位.

### D. 选择性重建

由于执行了重定向, 该机制的时间成本很高.
所以只有当检测不到cookie时, 才会触发**读取模式**, 以本方案来追踪用户.

此外作者还讨论了此方案适用的攻击范围和可能的优化选项.

## 6. 实验效果

由于目前还没有针对favicon缓存的防御措施, 所以本方案总能够有效追踪用户. 但方案的弊端是重定向带来的时间开销, 当ID长度为32 bit时, 时间开销甚至能达到10秒.

## 7. 评价

1. 以一种简单的方式实现了对用户的追踪;
2. 没有完全摆脱cookie;
3. 也许可以用一种多线程的方式来提升性能.