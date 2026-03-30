---
title: 生活片段分享平台
description: 本文主要介绍三个生活片段分享平台。
date: 2026-03-30T18:40:25-08:00
draft: false
categories:
- Life
- GitHub
tags:
- Ech0
- Moments
- Memos
---

# 生活片段分享平台

## 前言

去年夏天，我搭建了一个 Life-Moments 的平台用于分享自己的生活，但是因为数据存储的问题，这个工具没能成为我日常工具的一部分。

类似微信朋友圈的工具有很多，比如Moments、Memos 和 Ech0。

| 特性        | Memos                                | Moments                         | Ech0                                             |
| ----------- | ------------------------------------ | ------------------------------- | ------------------------------------------------ |
| PWA 适配    | ✅ 支持                               | ❌ 不支持                        | ✅ 支持                                           |
| 标签系统    | ✅ 完善                               | ✅ 完善                          | ✅ 支持                                           |
| S3 图床     | ✅ 支持                               | ✅ 支持                          | ✅ 支持                                           |
| Meting 音乐 | ❌ 不支持                             | ✅ 支持                          | ✅ 支持                                           |
| 视频分享    | ✅ 支持                               | ✅ 支持                          | ✅ 支持                                           |
| 设计风格    | 功能导向，略显臃肿                   | 功能全面，类似后台              | 简约美观，轻量                                   |
| 开发活跃度  | 非常活跃，API 易变                   | 相对稳定                        | 活跃，快速迭代                                   |
| 个人评价    | 功能强大，但更新太折腾，生态不稳定。 | 功能完美，但没 PWA 对我是减分项 | 界面戳我，功能追上来了，PWA 是加分项，目前的最爱 |

## Ech0：自托管个人微博客

> 自托管个人微博客：你的时间线可以被分享、讨论，同时数据完全由你掌控。

[GitHub——Ech0 – An open-source, self-hosted lightweight publishing platform for personal idea sharing.](https://github.com/lin-snow/Ech0)

[Ech0 helps individuals publish thoughts, notes, and links with full data ownership, ultra-light footprint, and a focused writing experience.](https://www.ech0.app/)

![img](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20260330230221504.png)

Ech0 示例可以参考： https://mm.liushen.fun/

像 Memos 这样的工具非常适合快速记录想法。Ech0 更关注“记录之后”的阶段：把内容发布到个人时间线，让更多人可以持续关注和互动。 你可以在自己的服务器上托管内容，保留完整控制权，同时通过可选评论与分享保持轻连接，而不是变成复杂社交平台。 它保持轻量、易部署、完全开源。

**适合你，如果你想：**

- 搭建一个自己的公开或半公开动态站
- 用统一界面发布短文、链接与媒体卡片
- 兼顾数据主权，同时保留 RSS 与评论等能力
- 让个人内容具备轻社交连接能力，而不需要重型社交产品

**不太适合你，如果你需要：**

- 双链知识库式的笔记工作流（例如 Obsidian 风格）
- 团队优先的协作文档平台（例如 Notion 风格）
- 纯私密备忘且不关注发布/时间线场景

## Memos：私有、轻量、开源、自托管的备忘录

**Memos** 是一种隐私优先的轻量级笔记解决方案，可让您轻松捕捉和分享您的想法。它开源且免费支持自部署。生态上支持第三方的手机端APP、浏览器插件等方式记录笔记，优势是生态够强，缺点是开发者每次更新大版本后之前的API可能就直接废弃了。

![img](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20260330225617504.png)

这是一个采用 React+Tailwind+TypeScript+Go 开发的备忘录中心，相当于极简的微博。支持私有/公开备忘录、标签、互动式日历等功能，以及 Docker 部署。

**Memos** 的官方文档： https://www.usememos.com/docs

开源地址： https://github.com/usememos/memos

### 介绍

- `Memos`是一个开源的碎片化笔记工具.
- 它小巧,功能强大,易于部署,且永久免费.
- 最重要区别于市面上其他的笔记软件的区别便是私有化.自托管.

### 功能

- 支持Markdown语法
- 支持S3存储
- 支持多用户
- 支持评论
- 支持MYsql或者SQLite作为数据库
- 支持API

### Memos 相关工具

- [Moe Memos](https://link.zhihu.com/?target=https%3A//memos.moe/) - 适用于 iOS 和 Android 的第三方客户端
- [lmm214/memos-bber](https://link.zhihu.com/?target=https%3A//github.com/lmm214/memos-bber) - Chrome 扩展
- [Rabithua/memos_wmp](https://link.zhihu.com/?target=https%3A//github.com/Rabithua/memos_wmp) - 微信小程序
- [qazxcdswe123/telegramMemoBot](https://link.zhihu.com/?target=https%3A//github.com/qazxcdswe123/telegramMemoBot) - 电报机器人
- [eallion/memos.top](https://link.zhihu.com/?target=https%3A//github.com/eallion/memos.top) - 使用 Memos API 呈现的静态页面
- [eindex/logseq-memos-sync](https://link.zhihu.com/?target=https%3A//github.com/EINDEX/logseq-memos-sync) - Logseq 插件
- [JakeLaoyu/memos-import-from-flomo](https://link.zhihu.com/?target=https%3A//github.com/JakeLaoyu/memos-import-from-flomo) - 导入数据。来自flomo、微信读书的支持
- [发送到备忘录](https://link.zhihu.com/?target=https%3A//sharecuts.cn/shortcut/12640)- iOS 的快捷方式
- [备忘录 Raycast Extension](https://link.zhihu.com/?target=https%3A//www.raycast.com/JakeYu/memos) - Raycast 扩展
- [Memos Desktop](https://link.zhihu.com/?target=https%3A//github.com/xudaolong/memos-desktop) - 适用于 MacOS 和 Windows 的第三方客户端
- [MemosGallery](https://link.zhihu.com/?target=https%3A//github.com/BarryYangi/MemosGallery) - 使用 Memos API 呈现的静态图库

## Moments - 极简朋友圈

moments 是一个极简设计的朋友圈社交平台，旨在为用户提供一个简洁而高效的分享空间。它允许用户轻松记录和分享生活中的点滴时刻、创意作品以及其他任何想要记录的内容。通过其直观的界面和一系列实用功能，moments 促进了用户之间的交流与互动。

[GitHub——极简朋友圈](https://github.com/kingwrcy/moments)

作者的 Demo：https://m.mblog.club/

![9eb713eceaeabeaa3a5c0e9ad8852ec.jpg](https://img2024.cnblogs.com/blog/2555265/202404/2555265-20240428144039560-1724832785.jpg)

### 功能说明

用户系统：

- 默认管理员：`admin/a123456`，登录后可在后台修改密码
- 多用户模式：可以在后台设置是否允许用户注册

Memo：

- 支持使用 Markdown 语法编写
- 支持修改发布时间，若发布时间晚于当前时间，则对游客不可见
- 支持添加标签，方便分类查找
- 支持上传图片、视频
- 支持引用外部音乐、外部视频、外部链接
- 支持引用豆瓣读书、豆瓣电影
- 支持点赞、评论，可在后台开启/关闭评论功能

文件上传：

上传到服务器时：

- 支持上传到服务器的 `$UPLOAD_DIR` 目录下
- 上传文件时，会先通过 SHA256 检查文件是否被上传过，若上传过则可以“秒传”
- 上传图片时，会自动创建图片的缩略图，在浏览 Memo 时默认加载缩略图
- 上传文件时，会和 Memo 产生关联，可以在设置中清理无用的文件，无用的文件将会被整理到 `$UPLOAD_DIR/removed` 目录下

上传到 S3 时：

- 支持上传到兼容 S3 的存储服务
- 开启 S3 时，可以设置图片压缩后缀，在浏览 Memo 时，会默认请求带后缀的图片加载缩略图

以下功能没有详细介绍，请自行探索：

- 友情链接
- 邮件通知
- RSS 订阅
- 暗黑模式
- 数据库自动备份，当检测到启动的版本发生变化时，自动备份数据库文件到 `$DB` 目录下

## 参考资料

[从Moments迁移到Ech0](https://blog.liushen.fun/posts/e0b4d5a/)