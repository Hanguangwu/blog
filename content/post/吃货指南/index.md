---
title: 吃货指南
description: 本文介绍使用一些介绍中国美食的仓库。
date: 2026-01-18T18:34:25-08:00
draft: false
categories:
- 生活
tags:
- GitHub
- Foodie
---

## 前言

这里暂时收纳了一些介绍中国菜肴的GitHub仓库，日后我要结合学过的《中国饮食文化》课程等做一份适合自己的全球食谱！还可以考虑抗炎、「轻断食」等方面。

作为一个吃货，未来可期！

## 吃啥好呢

[GitHub-Repo](https://github.com/jonssonyan/what-to-eat)

个性化菜谱推荐与收藏系统，支持多条件定制、国际化、第三方登录，助你轻松解决“今天吃什么”的难题。

### 主要功能

- 🍽️ 个性化菜谱推荐：支持饮食偏好、口味、烹饪时间、难度等多条件定制
- ⭐ 菜谱收藏夹：一键收藏、管理你的专属菜谱
- 🏷️ 用户称号系统：根据收藏数量自动授予等级称号
- 🌍 国际化支持：中英文切换，界面友好
- 🔒 第三方登录：支持 Google 登录
- 📝 更新日志与反馈：随时查看历史变更，欢迎提出建议
- 📱 移动端适配：良好的移动端体验

## 🍳 一饭封神

[GitHub-Repo](https://github.com/liu-ziting/what-to-eat)

一饭封神：一个基于 AI 的智能菜谱生成平台，支持中华八大菜系 + 国际料理，提供营养分析、酒水推荐、菜谱效果图生成等全方位烹饪指导。

[eat.lz-t.top](http://eat.lz-t.top/ "http://eat.lz-t.top")

> 🚀 **Vibe Coding**  
> 通过 Kiro 编辑器，实现了从需求分析、架构设计到代码实现的全流程开发。 [English](https://github.com/liu-ziting/what-to-eat/blob/master/README_EN.md) | 中文

基于 AI 的智能菜谱生成平台，支持中华八大菜系 + 国际料理，提供营养分析、酒水推荐、菜谱效果图生成等功能。
### 🚀 核心功能

- **智能菜谱生成** - 基于食材和菜系偏好生成专业菜谱
- **营养分析** - 详细营养成分分析和健康评分
- **AI 效果图** - 一键生成精美菜品图片
- **酒水搭配** - 专业侍酒师推荐
- **酱汁设计** - 定制化调料配方
- **收藏管理** - 保存和管理喜爱的菜谱
- **料理占卜** - 趣味性饮食运势
- **配置管理** - 动态配置 AI 模型参数，支持多服务商切换

### 🛠️ 技术栈

- **前端框架：** Vue 3.4 + TypeScript 5.3+
- **样式方案：** Tailwind CSS 3.4+
- **构建工具：** Vite 5.0+
- **AI 服务：** OpenAI 标准
- **部署平台：** Vercel + Netlify

## 程序员做饭指南

[GitHub-Repo](https://github.com/Anduin2017/HowToCook)

最近宅在家做饭，作为程序员，我偶尔在网上找找菜谱和做法。但是这些菜谱往往写法千奇百怪，经常中间莫名出来一些材料。对于习惯了形式语言的程序员来说极其不友好。

所以，我计划自己搜寻菜谱并结合实际做菜的经验，准备用更清晰精准的描述来整理常见菜的做法，以方便程序员在家做饭。

同样，我希望它是一个由社区驱动和维护的开源项目，使更多人能够一起做一个有趣的仓库。所以非常欢迎大家贡献它~

程序员在家做饭方法指南。Programmer's guide about how to cook at home (Simplified Chinese only).

[cook.aiursoft.com](https://cook.aiursoft.com/ "https://cook.aiursoft.com")

### 本地部署

如果需要在本地部署菜谱 Web 服务，可以在安装 Docker 后运行下面命令：

```shell
docker pull ghcr.io/anduin2017/how-to-cook:latest
docker run -d -p 5000:80 ghcr.io/anduin2017/how-to-cook:latest
```

如需下载 PDF 版本，可以在浏览器中访问 [/document.pdf](https://cook.aiursoft.com/document.pdf)








