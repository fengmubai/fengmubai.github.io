---
layout:     post
title:      "Windows平台Java开发环境配置"
subtitle:   "2021年的现代方法"
date:       2021-04-11
author:     "风沐白"
catalog: true
tags:
    - 开发环境
    - Windows
---

在2021年, 我们应该抛弃古老的传统, 而用现代化的方法来安装Java.
<p align="right" >--不是鲁迅说的</p>

## 1. 安装方法

打开PowerShell, 输入以下命令:

```powershell
# 网络环境不好的话, 请使用代理
# $env:https_proxy='<host>:<port>'
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')
scoop bucket add java
scoop update
scoop install microsoft-jdk # 微软推出的java 11开发套件
# scoop install openjdk8-redhat # 红帽维护的java 8
```

## 2. 为什么这样做

### 2.1 纯命令行

人们都或多或少接触过Linux.
在Linux下面安装软件, 基本只需要在命令行使用包管理器即可完成软件的查询, 安装, 更新和卸载.

Windows下同样也有包管理器, **Chocolatey**, **Scoop**, 以及微软自己的**winget**等等.
这里使用的就是**Scoop**, 你可以到它的[Github](https://github.com/lukesampson/scoop)页面了解更多.

虽然Windows的包管理器不可能完全覆盖Windows软件生态, 但对于开发人员来说, 其常用的软件肯定是优先纳入的.
除了Java, 你还可以用Scoop配置Golang, Python, GCC等环境.
只需要一行命令, 不用再自己去配置环境变量.

### 2.2 版本管理

你可以用Scoop同时安装几个jdk版本, 同样只需要一行命令:
```powershell
scoop reset <jdk_package_name>
```
就可以在不同版本之间切换.


同样的, 更新小版本号也仅需一行命令:
```powershell
scoop update <jdk_package_name>
```

### 2.3 规避法律风险

传统的方法总是会引导人们去下载安装**Oracle JDK**, 实际上这是有一定的法律风险的.
在生产环境下使用Oracle JDK是需要授权的.
虽然Oracle公司不一定会找上门来, 但还是要做一个遵纪守法的好公民的.