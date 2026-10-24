---
title: MAS：一条命令永久激活Windows和Office
description: 本文介绍MAS激活工具。
date: 2025-10-15T17:34:25+08:00
draft: false
categories:
- APP
tags:
- GitHub
- MAS
---




# MAS：一条命令永久激活Windows和Office


## 简介

[GitHub-MAS](https://github.com/massgravel/Microsoft-Activation-Scripts)

[MAS：一条命令永久激活Windows和Office+实现原理解读](https://zhuanlan.zhihu.com/p/1906620655807497264)

利用一条命令即可永久激活 Windows 和 Office，到底是如何实现的？Microsoft Activation Scripts，简称：MAS，是一个由开源社区开发并维护的项目，旨在通过一系列自动化脚本来永久激活 Windows 和 Office。注意是"永久激活"，非 KMS 180天。该程序通常被用作绕过 Microsoft 的正版验证机制，允许用户在未购买合法产品密钥的情况下使用 Windows 和 Office 正版软件。程序中的脚本和工具通常会通过对操作系统和应用程序进行特定的修改来实现激活操作。

本文介绍一下该激活命令的使用方法，并解读一下实现原理。这条命令是：

```text
irm https://get.activated.win | iex
```

## **命令解析**

我们将命令拆解成 irm 和 iex 两部分：

irm 是 PowerShell中的别名，代表 Invoke-RestMethod，这是一个用于发送 HTTP 请求并从远程服务器获取数据的命令。它的作用是从指定的 URL 下载内容并将其返回给 PowerShell。

具体来说，这里通过 irm 向 get.activated.win 发送一个 GET 请求。该 URL 是一个远程服务器地址，提供激活脚本。请求后，服务器将返回一段包含激活信息或脚本内容的数据。

iex 是 PowerShell 中的别名，代表 Invoke-Expression，它的作用是执行作为字符串传入的 PowerShell 脚本或命令。该命令接收通过 irm 获取的内容（即返回的脚本），然后执行它。

## **实现原理**

当执行该命令时，PowerShell 会先向 get.activated.win 发送请求，下载服务器上托管的脚本内容：

下载的脚本内容通常会包括一些预定义的命令或代码，这些代码用于绕过 Microsoft 的产品激活机制。下载并执行该脚本后，系统会尝试通过修改激活状态或注入激活密钥等方式，激活 Windows 操作系统和 Office 应用程序。

该脚本中包含了 HWID（硬件 ID）、Ohook、KMS38、Online KMS 激活方法，至于它如何实现“永久激活”，限于篇幅，iHackSoft 将另外发文做详细剖析。

## **使用方法**

- 右键点击 Windows 开始菜单，以管理员身份运行 Windows PowerShell （注意不是 CMD）。
- 输入下面的代码并回车：

```text
irm https://get.activated.win | iex
```

- 之后会看到如下图界面，其实就是从服务器上下载了 Microsoft Activation Scripts 这个脚本：

![](https://picx.zhimg.com/v2-8fb762b4d3479003d94e3bb9486fe267_1440w.jpg)

选择“1”：采用 HWID 激活方式，数字许可证永久激活Windows！需联网。  

选择“2”：采用 Ohook 激活方式，通过修改一个激活dll文件，实现永久激活Office，无需联网。

选择“3”：TSForge是一款由MAS团队开发的工具，能够绕过微软的在线验证机制，实现Windows和Office的永久激活。  

选择“4”：采用 KMS38 激活方式，激活 Windows/Server 到2038年，无需联网。  

选择“5”： 采用 Online KMS 激活方式，激活 Windows/Server/Office 一次180天，到期自动续，需要联网。

选择相应数字，回车之后，稍等片刻，成功激活！







