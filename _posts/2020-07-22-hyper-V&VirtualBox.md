---
layout:     post
title:      "Windows 10 (1024) WSL2"
subtitle:   "Windows 10 (2004) 启用wsl2, 并与VirtualBox 6.0+共存"
date:       2020-07-22 13:04:59
author:     "风沐白"
catalog: true
tags:
    - Windows
    - 虚拟机
    - 开发环境
    - WSL
---

## 1. 前提要求
1. Windows 版本: 2004+
2. VirtualBox 版本: 6.0+
3. CPU启用虚拟化

*注 1* : 迁移自本人[CSDN文章](https://blog.csdn.net/qq_36992069/article/details/104750248)

*注 2* : 本文中提到的命令一般都需要管理员权限

## 2. Windows 10 (2004) 中的wsl2新特性

Windows Subsystem for Linux (WSL) 2改进

1. 使用localhost从Windows访问wsl
>在使用 WSL 2 发布的第一个版本中，需要通过远程 IP 地址访问网络应用程序，但现在这个问题已经解决，现在可以使用 localhost 从 Windows 访问 Linux 网络应用程序。
2. WSL 全局配置
>所有 WSL 2 发行版都运行在同一个虚拟机(VM)上，因此，在此 VM 的任何配置选项都将全局应用于所有 WSL 2 发行版。在这个新的更新中，增加了为 WSL 使用全局配置选项的能力，这些选项是针对那些希望进一步定制他们的 WSL 体验的超级用户。  
在用户文件夹中创建一个名为 .wslconfig 的新文件(C:\Users\<yourUsername>\where <yourUsername> 是你的 Windows 登录名)。wslconfig 文件是以 INI 文件为模型的，就像 .gitconfig 一样。
3. 在 WSL 2 中使用自定义内核
>提供了一个 WSL 2 的 Linux 内核，它是在 Windows 中提供的。如果你希望有一个特定的内核为你的 WSL 2 发行版提供电源，例如使用特定的内核模块，现在可以在 .wslconfig 文件中使用内核选项来指定到机器上内核的路径，并且该内核在启动时将被加载到 WSL 2 VM 中。如果没有指定选项，将回到使用 Windows 提供的 Linux 内核作为 WSL 2 的一部分。

## 3. 步骤

### 1. VirtualBox启用hyper-v支持

进入VirtualBox安装目录, 确定当前目录下存在VBoxManage.exe文件, 在当前目录打开powershell. 或者你将VBoxManage.exe所在目录加入环境变量, 任意路径下打开powershell.
执行以下命令:

```shell
#指定vbox下的虚拟系统开启这个功能
./VBoxManage.exe setextradata "<虚拟机名字>" "VBoxInternal/NEM/UseRing0Runloop" 0

#或指定vbox所有虚拟系统开启
./VBoxManage.exe setextradata global "VBoxInternal/NEM/UseRing0Runloop" 0
```

### 2. 启用wsl2

在cmd或powershell窗口中执行以下命令:

```shell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart #若已启用"Windows sandbox"或"容器"功能, 可跳过此行命令
```

设置切换wsl版本命令:

```shell
wsl --set-version <Distro> 2
```

## 4. 错误处理

### 1. 启动wsl时报错

Error: ==参考的对象类型不支持尝试的操作==  
执行命令:

```shell
netsh winsock reset
```

### 2. 切换wsl版本失败

Error: ==请启用虚拟机平台==  
执行命令:

```shell
bcdedit /set hypervisorlaunchtype auto
```

## 5. 效果展示

1. 切换wsl2, 并在wsl2中启用网络服务
![切换wsl2, 并在wsl2中启用网络服务](https://img-blog.csdnimg.cn/2020030913344329.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_UEQu6aOO5rKQ55m9,size_24,color_FFFFFF,t_70)
2. 在Windows下访问wsl2网络服务
![在Windows下访问wsl2网络服务](https://img-blog.csdnimg.cn/20200309133212348.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_UEQu6aOO5rKQ55m9,size_24,color_FFFFFF,t_70)
3. VirtualBox虚拟机示例
![VirtualBox虚拟机示例](https://img-blog.csdnimg.cn/20200309133038847.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_UEQu6aOO5rKQ55m9,size_24,color_FFFFFF,t_70)

## 6. References

1. [[图]Windows 10 Version 2004新功能盘点 - Windows 10 - cnBeta.COM](https://www.cnbeta.com/articles/tech/952745.htm)
2. [【杂项】Virtualbox 6与Hyper-V共存](https://www.rehtt.com/index.php/archives/225/)
3. [安装 WSL 2 | Microsoft Docs](https://docs.microsoft.com/zh-cn/windows/wsl/wsl2-install)
4. [参考的对象类型不支持尝试的操作。 · Issue #4194 · microsoft/WSL](https://github.com/microsoft/WSL/issues/4194)
5. [Windows 10 打开Windows Sandbox提示找不到虚拟机监控程序，请启用虚拟机监控程序支持 - Microsoft Community](https://answers.microsoft.com/zh-hans/windows/forum/all/windows-10-%E6%89%93%E5%BC%80windows/286f8a35-6a74-433c-b00f-bcd03812d180)