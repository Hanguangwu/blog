---
title: 新型信息源
description: 本文介绍一些新型信息源。
date: 2026-03-07T12:34:25-08:00
draft: false
categories:
- APP
- GitHub
tags:
- NewsNow
- TrendRadar
---


# 新型信息源


## 前言

以后可以做定制化一个！

## NewsNow - 一个开源的实时热门新闻程序

### 概述

NewsNow 是一个开源的实时热门新闻聚合程序，旨在为用户提供一个简洁的界面和流畅的阅读体验，以便随时掌握最新的全球资讯。该程序支持多种部署方式，包括 Cloudflare Pages、Vercel 以及 Docker，同时提供了灵活的配置和数据库支持。

### 功能特点

1. **实时新闻更新**：NewsNow 提供实时更新的热门新闻，让用户能够及时了解全球动态。
2. **优雅界面**：用户界面简洁优雅，提升阅读体验。
3. **灵活部署**：支持 Cloudflare Pages、Vercel 以及 Docker 等多种部署方式。
4. **自定义配置**：用户可以根据需要配置环境变量和数据库，以定制新闻阅读体验。
5. **数据库支持**：支持多种数据库，包括 Cloudflare D1 数据库。
6. **开源项目**：基于 MIT 许可证，用户可以自由使用和修改。

### 使用总结

NewsNow 的主要优势在于其简洁的界面设计和实时新闻更新能力。作为一个开源项目，它允许用户根据自己的需求进行定制和部署。以下是一些关键点：

- **部署简便**：如果不需要登录和缓存功能，可以直接将 NewsNow 部署到 Cloudflare Pages 或 Vercel，只需 Fork 项目后在对应平台上导入即可。
- **登录功能**：登录功能涉及到 GitHub OAuth。创建 GitHub App 后，将获得 Client ID 和 Client Secret，不同平台的环境变量设置位置不同，请参考 `example.env.server` 文件。
- **数据库部署**：推荐使用 Cloudflare Pages 和 Docker 部署。对于 Vercel，需要自行解决数据库问题。支持的数据库可以在 db0 connectors 查看。
- **开源社区**：NewsNow Labs 是 NewsNow 的研发部门，进行自然语言处理、机器学习、基础设施管理和软件开发工具等方面的前沿研究，并公开其研究成果。

### 项目地址

- **项目地址**：[https://github.com/ourongxing/newsnow](https://github.com/ourongxing/newsnow)
- **在线地址**：[https://newsnow.busiyi.world](https://newsnow.busiyi.world/)


### 代码解读

#### zhihu.ts 获取知乎热榜

```ts

interface Res {
  data: {
    type: "hot_list_feed"
    style_type: "1"
    feed_specific: {
      answer_count: 411
    }
    target: {
      title_area: {
        text: string
      }
      excerpt_area: {
        text: string
      }
      image_area: {
        url: string
      }
      metrics_area: {
        text: string
        font_color: string
        background: string
        weight: string
      }
      label_area: {
        type: "trend"
        trend: number
        night_color: string
        normal_color: string
      }
      link: {
        url: string
      }
    }
  }[]
}

export default defineSource({
  zhihu: async () => {
    const url = "https://www.zhihu.com/api/v3/feed/topstory/hot-list-web?limit=20&desktop=true"
    const res: Res = await myFetch(url)
    return res.data
      .map((k) => {
        return {
          id: k.target.link.url.match(/(\d+)$/)?.[1] ?? k.target.link.url,
          title: k.target.title_area.text,
          extra: {
            info: k.target.metrics_area.text,
            hover: k.target.excerpt_area.text,
          },
          url: k.target.link.url,
        }
      })
  },
})

```

#### github.ts

```ts

import * as cheerio from "cheerio"
import type { NewsItem } from "@shared/types"

const trending = defineSource(async () => {
  const baseURL = "https://github.com"
  const html: any = await myFetch("https://github.com/trending?spoken_language_code=")
  const $ = cheerio.load(html)
  const $main = $("main .Box div[data-hpc] > article")
  const news: NewsItem[] = []
  $main.each((_, el) => {
    const a = $(el).find(">h2 a")
    const title = a.text().replace(/\n+/g, "").trim()
    const url = a.attr("href")
    const star = $(el).find("[href$=stargazers]").text().replace(/\s+/g, "").trim()
    const desc = $(el).find(">p").text().replace(/\n+/g, "").trim()
    if (url && title) {
      news.push({
        url: `${baseURL}${url}`,
        title,
        id: url,
        extra: {
          info: `✰ ${star}`,
          hover: desc,
        },
      })
    }
  })
  return news
})

export default defineSource({
  "github": trending,
  "github-trending-today": trending,
})

```

#### hackernews.ts

```ts
import * as cheerio from "cheerio"
import type { NewsItem } from "@shared/types"

export default defineSource(async () => {
  const baseURL = "https://news.ycombinator.com"
  const html: any = await myFetch(baseURL)
  const $ = cheerio.load(html)
  const $main = $(".athing")
  const news: NewsItem[] = []
  $main.each((_, el) => {
    const a = $(el).find(".titleline a").first()
    // const url = a.attr("href")
    const title = a.text()
    const id = $(el).attr("id")
    const score = $(`#score_${id}`).text()
    const url = `${baseURL}/item?id=${id}`
    if (url && id && title) {
      news.push({
        url,
        title,
        id,
        extra: {
          info: score,
        },
      })
    }
  })
  return news
})
```

#### wallstreetcn.ts

```ts
interface Item {
  uri: string
  id: number
  title?: string
  content_text: string
  content_short: string
  display_time: number
  type?: string
}
interface LiveRes {
  data: {
    items: Item[]
  }
}

interface NewsRes {
  data: {
    items: {
      // ad
      resource_type?: string
      resource: Item
    }[]
  }
}

interface HotRes {
  data: {
    day_items: Item[]
  }
}

// https://github.com/DIYgod/RSSHub/blob/master/lib/routes/wallstreetcn/live.ts
const live = defineSource(async () => {
  const apiUrl = `https://api-one.wallstcn.com/apiv1/content/lives?channel=global-channel&limit=30`

  const res: LiveRes = await myFetch(apiUrl)
  return res.data.items
    .map((k) => {
      return {
        id: k.id,
        title: k.title || k.content_text,
        extra: {
          date: k.display_time * 1000,
        },
        url: k.uri,
      }
    })
})

const news = defineSource(async () => {
  const apiUrl = `https://api-one.wallstcn.com/apiv1/content/information-flow?channel=global-channel&accept=article&limit=30`

  const res: NewsRes = await myFetch(apiUrl)
  return res.data.items
    .filter(k => k.resource_type !== "theme" && k.resource_type !== "ad" && k.resource.type !== "live" && k.resource.uri)
    .map(({ resource: h }) => {
      return {
        id: h.id,
        title: h.title || h.content_short,
        extra: {
          date: h.display_time * 1000,
        },
        url: h.uri,
      }
    })
})

const hot = defineSource(async () => {
  const apiUrl = `https://api-one.wallstcn.com/apiv1/content/articles/hot?period=all`

  const res: HotRes = await myFetch(apiUrl)
  return res.data.day_items
    .map((h) => {
      return {
        id: h.id,
        title: h.title!,
        url: h.uri,
      }
    })
})

export default defineSource({
  "wallstreetcn": live,
  "wallstreetcn-quick": live,
  "wallstreetcn-news": news,
  "wallstreetcn-hot": hot,
})
```
#### sspai.ts

```ts
interface Res {
  data: {
    id: number
    title: string
  }[]
}

export default defineSource(async () => {
  const timestamp = Date.now()
  const limit = 30
  const url = `https://sspai.com/api/v1/article/tag/page/get?limit=${limit}&offset=0&created_at=${timestamp}&tag=%E7%83%AD%E9%97%A8%E6%96%87%E7%AB%A0&released=false`
  const res: Res = await myFetch(url)
  return res.data.map((k) => {
    const url = `https://sspai.com/post/${k.id}`
    return {
      id: k.id,
      title: k.title,
      url,
    }
  })
})
```

## AI 舆情监控助手与热点筛选工具——TrendRadar

⭐AI-driven public opinion & trend monitor with multi-platform aggregation, RSS, and smart alerts.🎯 告别信息过载，你的 AI 舆情监控助手与热点筛选工具！聚合多平台热点 + RSS 订阅，支持关键词精准筛选。AI 翻译 + AI 分析简报直推手机，也支持接入 MCP 架构，赋能 AI 自然语言对话分析、情感洞察与趋势预测等。支持 Docker ，数据本地/云端自持。集成微信/飞书/钉钉/Telegram/邮件/ntfy/bark/slack 等渠道智能推送。

[sansan0.github.io/TrendRadar/](https://sansan0.github.io/TrendRadar/ "https://sansan0.github.io/TrendRadar/")

[GitHub-Repo-TrendRadar](https://github.com/sansan0/TrendRadar)
## ✨ 核心功能

### **全网热点聚合**

- 知乎
- 抖音
- bilibili 热搜
- 华尔街见闻
- 贴吧
- 百度热搜
- 财联社热门
- 澎湃新闻
- 凤凰网
- 今日头条
- 微博

默认监控 11 个主流平台，也可自行增加额外的平台。

### **RSS 订阅源支持**（v4.5.0 新增）

支持 RSS/Atom 订阅源抓取，按关键词分组统计（与热榜格式一致）：

- **统一格式**：RSS 与热榜使用相同的关键词匹配和显示格式
- **简单配置**：直接在 `config.yaml` 中添加 RSS 源
- **合并推送**：热榜和 RSS 合并为一条消息推送

> 💡 RSS 使用与热榜相同的 `frequency_words.txt` 进行关键词过滤

### **可视化配置编辑器**

提供基于 Web 的图形化配置界面，无需手动编辑 YAML 文件，通过表单即可完成所有配置项的修改与导出。

👉 **在线体验**：[https://sansan0.github.io/TrendRadar/](https://sansan0.github.io/TrendRadar/)

### **智能推送策略**

**三种推送模式**：

|模式|适用场景|推送特点|
|---|---|---|
|**当日汇总** (daily)|企业管理者/普通用户|按时推送当日所有匹配新闻（会包含之前推送过的）|
|**当前榜单** (current)|自媒体人/内容创作者|按时推送当前榜单匹配新闻（持续在榜的每次都出现）|
|**增量监控** (incremental)|投资者/交易员|仅推送新增内容，零重复|

> 💡 **快速选择指南：**
> 
> - 不想看到重复新闻 → 用 `incremental`（增量监控）
> - 想看完整榜单趋势 → 用 `current`（当前榜单）
> - 需要每日汇总报告 → 用 `daily`（当日汇总）
> 
> 详细对比和配置教程见 [配置详解 - 推送模式详解](https://github.com/sansan0/TrendRadar#3-%E6%8E%A8%E9%80%81%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3)

**附加功能**（可选）：

|功能|说明|默认|
|---|---|---|
|**调度系统**|按周一到周日逐日编排：为每天分配不同时间段、推送模式和 AI 分析策略。内置 5 种预设（always_on / morning_evening / office_hours / night_owl / custom），也可自定义。支持工作日/周末差异化、跨午夜时段、per-period 去重（v6.0.0）|morning_evening|
|**内容顺序配置**|通过 `display.region_order` 调整各区域（热榜、新增热点、RSS、独立展示区、AI 分析）的显示顺序；通过 `display.regions` 控制各区域是否显示（v5.2.0）|见配置文件|
|**显示模式切换**|`keyword`=按关键词分组，`platform`=按平台分组（v4.6.0 新增）|keyword|

> 💡 详细配置教程见 [推送内容怎么显示？](https://github.com/sansan0/TrendRadar#7-%E6%8E%A8%E9%80%81%E5%86%85%E5%AE%B9%E6%80%8E%E4%B9%88%E6%98%BE%E7%A4%BA) 和 [什么时候给我推送？](https://github.com/sansan0/TrendRadar#8-%E4%BB%80%E4%B9%88%E6%97%B6%E5%80%99%E7%BB%99%E6%88%91%E6%8E%A8%E9%80%81)

### **精准内容筛选**

设置个人关键词（如：AI、比亚迪、教育政策），只推送相关热点，过滤无关信息

> 💡 **基础配置教程**：[关键词配置 - 基础语法](https://github.com/sansan0/TrendRadar#%E5%85%B3%E9%94%AE%E8%AF%8D%E5%9F%BA%E7%A1%80%E8%AF%AD%E6%B3%95)
> 
> 💡 **高级配置教程**：[关键词配置 - 高级配置](https://github.com/sansan0/TrendRadar#%E5%85%B3%E9%94%AE%E8%AF%8D%E9%AB%98%E7%BA%A7%E9%85%8D%E7%BD%AE)
> 
> 💡 也可以不做筛选，完整推送所有热点（将 frequency_words.txt 留空）

### **热点趋势分析**

实时追踪新闻热度变化，让你不仅知道"什么在热搜"，更了解"热点如何演变"

- **时间轴追踪**：记录每条新闻从首次出现到最后出现的完整时间跨度
- **热度变化**：统计新闻在不同时间段的排名变化和出现频次
- **新增检测**：实时识别新出现的热点话题，用🆕标记第一时间提醒
- **持续性分析**：区分一次性热点话题和持续发酵的深度新闻
- **跨平台对比**：同一新闻在不同平台的排名表现，看出媒体关注度差异

> 💡 推送格式说明见 [消息样式说明](https://github.com/sansan0/TrendRadar#5-%E6%88%91%E6%94%B6%E5%88%B0%E7%9A%84%E6%B6%88%E6%81%AF%E9%95%BF%E4%BB%80%E4%B9%88%E6%A0%B7)

### **个性化热点算法**

不再被各个平台的算法牵着走，TrendRadar 会重新整理全网热搜

> 💡 三个比例可以调整，详见 [配置详解 - 热点权重调整](https://github.com/sansan0/TrendRadar#4-%E7%83%AD%E7%82%B9%E6%9D%83%E9%87%8D%E8%B0%83%E6%95%B4)

### **多渠道多账号推送**

支持**企业微信**(+ 微信推送方案)、**飞书**、**钉钉**、**Telegram**、**邮件**、**ntfy**、**Bark**、**Slack**，消息直达手机和邮箱

> 💡 详细配置教程见 [推送到多个群/设备](https://github.com/sansan0/TrendRadar#10-%E6%8E%A8%E9%80%81%E5%88%B0%E5%A4%9A%E4%B8%AA%E7%BE%A4%E8%AE%BE%E5%A4%87)

### **AI 多语言翻译**（v5.2.0 新增）

将推送内容翻译为任意语言，打破语言壁垒，无论是阅读国内热点还是通过 RSS 订阅海外资讯，都能以母语轻松获取

- **一键翻译**：在 `config.yaml` 中设置 `ai_translation.enabled: true` 和目标语言即可
- **多语言支持**：支持 English、Korean、Japanese、French 等任意语言
- **智能批量处理**：自动批量翻译，减少 API 调用次数，节省成本
- **自定义风格**：通过 `ai_translation_prompt.txt` 自定义翻译风格和术语
- **共享模型配置**：与 AI 分析功能共用 `ai` 配置段的模型设置

```yaml
# config.yaml 快速启用示例
ai_translation:
  enabled: true
  language: "English"  # 翻译目标语言
```

> 💡 翻译功能与 AI 分析功能共享模型配置，只需配置一次 `ai.api_key` 即可同时使用两个功能

**RSS 源参考**：以下是一些 RSS 订阅源合集，可按需选用

- [awesome-tech-rss](https://github.com/tuan3w/awesome-tech-rss) - 科技、创业、编程领域博客和媒体
- [awesome-rss-feeds](https://github.com/plenaryapp/awesome-rss-feeds) - 世界各国主流新闻媒体 RSS 合集

> ⚠️ 部分海外媒体内容可能涉及敏感话题，AI 模型可能拒绝翻译，建议根据实际需求筛选订阅源

### **灵活存储架构**（v4.0.0 重大更新）

**多存储后端支持**：

- **远程云存储**：GitHub Actions 环境默认，支持 S3 兼容协议（R2/OSS/COS 等），数据存储在云端，不污染仓库
- **本地 SQLite 数据库**：Docker/本地环境默认，数据完全可控
- **自动后端选择**：根据运行环境智能切换存储方式

> 💡 详细说明见 [数据保存在哪里？](https://github.com/sansan0/TrendRadar#11-%E6%95%B0%E6%8D%AE%E4%BF%9D%E5%AD%98%E5%9C%A8%E5%93%AA%E9%87%8C)

### **多端部署**

- **GitHub Actions**：定时自动爬取 + 远程云存储（需签到续期）
- **Docker 部署**：支持多架构容器化运行，数据本地存储
- **本地运行**：Windows/Mac/Linux 直接运行

### **AI 分析推送（v5.0.0 新增）**

使用 AI 大模型对推送内容进行深度分析，自动生成热点洞察报告

- **智能分析**：自动分析热点趋势、关键词热度、跨平台关联、潜在影响
- **多提供商**：支持 DeepSeek、OpenAI、Gemini 及 OpenAI 兼容接口
- **灵活推送**：可选仅原始内容、仅 AI 分析、或两者都推送
- **自定义提示词**：通过 `config/ai_analysis_prompt.txt` 自定义分析角度

> 💡 详细配置教程见 [让 AI 帮我分析热点](https://github.com/sansan0/TrendRadar#12-%E8%AE%A9-ai-%E5%B8%AE%E6%88%91%E5%88%86%E6%9E%90%E7%83%AD%E7%82%B9)

### **独立展示区（v5.0.0 新增）**

为指定平台提供完整热榜展示，不受关键词过滤影响

- **完整热榜**：指定平台的热榜完整展示，适合想看完整排名的用户
- **RSS 独立展示**：RSS 源内容可完整展示，不受关键词限制
- **AI 深度分析**：可独立开启 AI 对完整热榜的趋势分析，无需在推送中展示
- **灵活配置**：支持配置展示平台、RSS 源、最大条数

> 💡 详细配置教程见 [推送内容怎么显示？ - 独立展示区](https://github.com/sansan0/TrendRadar#7-%E6%8E%A8%E9%80%81%E5%86%85%E5%AE%B9%E6%80%8E%E4%B9%88%E6%98%BE%E7%A4%BA)

### **AI 智能分析（v3.0.0 新增）**

基于 MCP (Model Context Protocol) 协议的 AI 对话分析系统，让你用自然语言深度挖掘新闻数据

> **💡 使用提示**：AI 功能需要本地新闻数据支持
> 
> - 项目自带测试数据，可立即体验功能
> - 建议自行部署运行项目，获取更实时的数据
> 
> 详见 [AI 智能分析](https://github.com/sansan0/TrendRadar#-ai-%E6%99%BA%E8%83%BD%E5%88%86%E6%9E%90)

### **网页部署**

运行后根目录生成 `index.html`，即为完整的新闻报告页面。

> **部署方式**：点击 **Use this template** 创建仓库，可部署到 Cloudflare Pages 或 GitHub Pages 等静态托管平台。
> 
> **💡 提示**：启用 GitHub Pages 可获得在线访问地址，进入仓库 Settings → Pages 即可开启。[效果预览](https://sansan0.github.io/TrendRadar/)
> 
> ⚠️ 原 GitHub Actions 自动存储功能已下线（该方案曾导致 GitHub 服务器负载过高，影响平台稳定性）。

### **减少 APP 依赖**

从"被算法推荐绑架"变成"主动获取自己想要的信息"

**适合人群：** 投资者、自媒体人、企业公关、关心时事的普通用户

**典型场景：** 股市投资监控、品牌舆情追踪、行业动态关注、生活资讯获取

### 代码分析

#### crawler/fetcher.py

```python
# coding=utf-8
"""
数据获取器模块

负责从 NewsNow API 抓取新闻数据，支持：
- 单个平台数据获取
- 批量平台数据爬取
- 自动重试机制
- 代理支持
"""

import json
import random
import time
from typing import Dict, List, Tuple, Optional, Union

import requests


class DataFetcher:
    """数据获取器"""

    # 默认 API 地址
    DEFAULT_API_URL = "https://newsnow.busiyi.world/api/s"

    # 默认请求头
    DEFAULT_HEADERS = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
        "Accept": "application/json, text/plain, */*",
        "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
        "Connection": "keep-alive",
        "Cache-Control": "no-cache",
    }

    def __init__(
        self,
        proxy_url: Optional[str] = None,
        api_url: Optional[str] = None,
    ):
        """
        初始化数据获取器

        Args:
            proxy_url: 代理服务器 URL（可选）
            api_url: API 基础 URL（可选，默认使用 DEFAULT_API_URL）
        """
        self.proxy_url = proxy_url
        self.api_url = api_url or self.DEFAULT_API_URL

    def fetch_data(
        self,
        id_info: Union[str, Tuple[str, str]],
        max_retries: int = 2,
        min_retry_wait: int = 3,
        max_retry_wait: int = 5,
    ) -> Tuple[Optional[str], str, str]:
        """
        获取指定ID数据，支持重试

        Args:
            id_info: 平台ID 或 (平台ID, 别名) 元组
            max_retries: 最大重试次数
            min_retry_wait: 最小重试等待时间（秒）
            max_retry_wait: 最大重试等待时间（秒）

        Returns:
            (响应文本, 平台ID, 别名) 元组，失败时响应文本为 None
        """
        if isinstance(id_info, tuple):
            id_value, alias = id_info
        else:
            id_value = id_info
            alias = id_value

        url = f"{self.api_url}?id={id_value}&latest"

        proxies = None
        if self.proxy_url:
            proxies = {"http": self.proxy_url, "https": self.proxy_url}

        retries = 0
        while retries <= max_retries:
            try:
                response = requests.get(
                    url,
                    proxies=proxies,
                    headers=self.DEFAULT_HEADERS,
                    timeout=10,
                )
                response.raise_for_status()

                data_text = response.text
                data_json = json.loads(data_text)

                status = data_json.get("status", "未知")
                if status not in ["success", "cache"]:
                    raise ValueError(f"响应状态异常: {status}")

                status_info = "最新数据" if status == "success" else "缓存数据"
                print(f"获取 {id_value} 成功（{status_info}）")
                return data_text, id_value, alias

            except Exception as e:
                retries += 1
                if retries <= max_retries:
                    base_wait = random.uniform(min_retry_wait, max_retry_wait)
                    additional_wait = (retries - 1) * random.uniform(1, 2)
                    wait_time = base_wait + additional_wait
                    print(f"请求 {id_value} 失败: {e}. {wait_time:.2f}秒后重试...")
                    time.sleep(wait_time)
                else:
                    print(f"请求 {id_value} 失败: {e}")
                    return None, id_value, alias

        return None, id_value, alias

    def crawl_websites(
        self,
        ids_list: List[Union[str, Tuple[str, str]]],
        request_interval: int = 100,
    ) -> Tuple[Dict, Dict, List]:
        """
        爬取多个网站数据

        Args:
            ids_list: 平台ID列表，每个元素可以是字符串或 (平台ID, 别名) 元组
            request_interval: 请求间隔（毫秒）

        Returns:
            (结果字典, ID到名称的映射, 失败ID列表) 元组
        """
        results = {}
        id_to_name = {}
        failed_ids = []

        for i, id_info in enumerate(ids_list):
            if isinstance(id_info, tuple):
                id_value, name = id_info
            else:
                id_value = id_info
                name = id_value

            id_to_name[id_value] = name
            response, _, _ = self.fetch_data(id_info)

            if response:
                try:
                    data = json.loads(response)
                    results[id_value] = {}

                    for index, item in enumerate(data.get("items", []), 1):
                        title = item.get("title")
                        # 跳过无效标题（None、float、空字符串）
                        if title is None or isinstance(title, float) or not str(title).strip():
                            continue
                        title = str(title).strip()
                        url = item.get("url", "")
                        mobile_url = item.get("mobileUrl", "")

                        if title in results[id_value]:
                            results[id_value][title]["ranks"].append(index)
                        else:
                            results[id_value][title] = {
                                "ranks": [index],
                                "url": url,
                                "mobileUrl": mobile_url,
                            }
                except json.JSONDecodeError:
                    print(f"解析 {id_value} 响应失败")
                    failed_ids.append(id_value)
                except Exception as e:
                    print(f"处理 {id_value} 数据出错: {e}")
                    failed_ids.append(id_value)
            else:
                failed_ids.append(id_value)

            # 请求间隔（除了最后一个）
            if i < len(ids_list) - 1:
                actual_interval = request_interval + random.randint(-10, 20)
                actual_interval = max(50, actual_interval)
                time.sleep(actual_interval / 1000)

        print(f"成功: {list(results.keys())}, 失败: {failed_ids}")
        return results, id_to_name, failed_ids
```

## WorldMonitor:开源的“全球战情室”软件，可追踪伊朗局势

Real-time global intelligence dashboard — AI-powered news aggregation, geopolitical monitoring, and infrastructure tracking in a unified situational awareness interface

[worldmonitor.app](https://worldmonitor.app/ "https://worldmonitor.app")

https://github.com/koala73/worldmonitor

**World Monitor** 是一个由独立开发者 koala73（Elie Habib）开发的实时全球情报仪表板。该项目旨在将原本分散在数十个独立工具中的地缘政治、军事、基础设施及金融数据整合至统一的态势感知界面（Single Pane of Glass）。截至 2026 年初，该项目在 GitHub 已获得超过 2.4 万个 Star，被广泛誉为“开源版彭博终端”或“平民版战情室”。

[World Monitor：开源的全球情报监测仪表板](https://zhichai.net/htmlpages/topic_177168715.html)
### 核心功能分析

World Monitor 的功能矩阵围绕**可视化、实时性、AI 化**三个维度展开：

- **多维数据图层（40+ Data Layers）：**
- **军事与冲突：** 实时追踪全球冲突热点、220+ 军事基地、核设施分布及 GPS 干扰区。
- **基础设施：** 覆盖全球海底光缆、油气管道、主要港口及 AI 数据中心。
- **实时动态：** 集成活体军机（ADS-B）、民用舰船（AIS）追踪及 NASA FIRMS 卫星火点监测。

- **AI 驱动的情报处理：**

	- 自动简报（World Brief）： 调用 LLM（如 GPT-4, Claude 或本地 Ollama）对每日全球要闻进行结构化摘要。
	- RAG 语义检索： 内置 Headline Memory，支持对历史事件进行语义搜索，而非简单的关键词匹配。
	- 不稳定指数（CII）： 基于实时信号量化评估不同国家/地区的动荡风险。

- **交互式 3D 可视化：**

	- 基于 **deck.gl** 与 **MapLibre GL JS** 开发，支持丝滑的 3D 地球旋转与多层数据叠加，提供极佳的视觉感知力。

- **三大专业变体：**
	- **World：** 侧重地缘政治与军事安全。
	- **Tech：** 侧重半导体供应链、AI 基础设施与科技趋势。
	- **Finance：** 侧重全球 92 家交易所动态、央行政策及预测市场（如 Polymarket）。

### 使用方法指南

轻量化访问（PWA）： 通过浏览器访问 https://worldmonitor.app ，可直接将其安装为 PWA 应用。其支持离线地图缓存，适合日常快速查阅。

World Monitor 核心入口：项目地址：https://github.com/koala73/worldmonitor在线体验（强烈建议立刻打开）： https://worldmonitor.app（世界版）， https://tech.worldmonitor.app （科技版） ， https://finance.worldmonitor.app （金融版）

桌面端运行（Tauri）： 下载适用于 Windows、macOS 或 Linux 的原生客户端。相比 Web 版，桌面端能更好地保护本地隐私，并支持系统级的实时通知推送。

界面实战操作指南

- 全方位监控： 使用鼠标拖拽旋转 3D 地球，点击热点（如冲突区或舰艇图标）查看详细数据。
- 一键 AI 总结： 在底部新闻面板中，点击单条新闻的 “Summarize” 按钮，AI 会快速给出核心解读。
- 快捷键助力：Cmd+K（或 Ctrl+K）可唤起全局搜索；面板支持自由拖拽和重组，适配超宽屏办公。
- 多语言支持： 设置中可切换为简体/繁体中文，降低阅读障碍。
- 本地部署与隐私模式（Docker/Source）：
- 安装Ollama： 登录Ollama 官网 下载安装Ollama，建议下载一个推理能力较强且显存占用适中的模型（如 Llama 3.2 或 Qwen 2.5）。
- World Monitor 端配置： 在 World Monitor 中将 AI 引擎从默认的云端（如 OpenAI/Gemini）切换至本地 Ollama。World Monitor 通常通过 HTTP API 与 Ollama 通信。默认地址为：http://localhost:11434，即可实现情报的本地化分析，确保敏感搜索数据不上传至云端。

### 技术栈

- **前端**: TypeScript + Vite + React
- **地图渲染**: deck.gl (WebGL) + MapLibre GL JS
- **AI/ML**: Groq (Llama 3.1) + OpenRouter + Transformers.js (浏览器端 NER/嵌入)
- **缓存**: Redis (Upstash) + 三层缓存架构（内存 → Redis → 上游）
- **跨用户 AI 去重**: 相同查询共享 LLM 结果，减少 API 调用

### 部署方式——零配置模式（推荐新手）

无需 API Key，基础功能正常运行。

#### 自建部署

```bash
git clone https://github.com/koala73/worldmonitor.git
cd worldmonitor
pnpm install
pnpm build
pnpm preview --host 0.0.0.0 --port 8888
```

#### 可选配置 (.env.local)

```plaintext
# AI/ML（可选）
VITE_GROQ_API_KEY=your_groq_key
VITE_OPENROUTER_API_KEY=your_openrouter_key

# 缓存（可选）
VITE_UPSTASH_REDIS_URL=your_redis_url
VITE_UPSTASH_REDIS_TOKEN=your_redis_token
```
### 优劣分析

**优势（Value）：**

- **效率革命：** 极大降低了情报搜集的时间成本。原本需要数小时的跨平台监测，现在可在 10 分钟内通过仪表板完成。
- **门槛降低：** 专业的 OSINT 工具以往多为付费或仅供机构使用，World Monitor 实现了专业能力的平民化。
- **高度可扩展：** 作为开源项目，支持用户自定义 RSS 源与 API 接口，具有极强的生命力。

**局限性（Limitations）：**

- **移动端适配：** 目前主打桌面端体验，屏幕尺寸小于 768px 时交互受限，不适合碎片化移动办公。
- **数据延迟：** 依赖公开数据源（Open Source），部分极端敏感事件可能存在分钟级甚至小时级的滞后。
- **语言限制：** 尽管 UI 支持中文，但部分深层专业图层的数据标签仍以英文为主。

### 使用建议

- **建立“早晚报”机制：** 建议分析从业者利用其 AI Brief 功能，每天固定生成两份针对特定区域的简报，快速建立背景感知。
- **信号叠加分析：** 当地图上某一区域同时出现“GPS 干扰”、“异常飞行轨迹”与“火点”时，应将其视为高等级预警信号。
- **结合本地 LLM：** 建议使用 Llama 3 或类似参数量的模型进行推理，既能保证分析逻辑的严密性，又能规避在线 API 的额度限制。
- **资产监控：** 投资者可利用 Finance 变体监控关键航运枢纽（如苏伊士运河）的船只密度，作为判断大宗商品走势的辅助指标。

### 代码解析

#### telegram-channels.json

```json
{
  "version": 1,
  "updatedAt": "2026-02-23T18:37:10Z",
  "note": "Product-managed curated list. Not user-configurable.",
  "channels": {
    "full": [
      {
        "handle": "VahidOnline",
        "label": "Vahid Online",
        "topic": "politics",
        "tier": 1,
        "enabled": true,
        "region": "iran",
        "maxMessages": 20
      },
      {
        "handle": "abualiexpress",
        "label": "Abu Ali Express",
        "topic": "middleeast",
        "tier": 2,
        "enabled": true,
        "region": "middleeast",
        "maxMessages": 25
      },
      {
        "handle": "AuroraIntel",
        "label": "Aurora Intel",
        "topic": "conflict",
        "tier": 2,
        "enabled": true,
        "region": "global",
        "maxMessages": 20
      },
      {
        "handle": "BNONews",
        "label": "BNO News",
        "topic": "breaking",
        "tier": 2,
        "enabled": true,
        "region": "global",
        "maxMessages": 25
      },
      {
        "handle": "ClashReport",
        "label": "Clash Report",
        "topic": "conflict",
        "tier": 2,
        "enabled": true,
        "region": "global",
        "maxMessages": 30
      },
      {
        "handle": "DeepStateUA",
        "label": "DeepState",
        "topic": "conflict",
        "tier": 2,
        "enabled": true,
        "region": "ukraine",
        "maxMessages": 20
      },
      {
        "handle": "DefenderDome",
        "label": "The Defender Dome",
        "topic": "conflict",
        "tier": 2,
        "enabled": true,
        "region": "global",
        "maxMessages": 25
      },
      {
        "handle": "englishabuali",
        "label": "Abu Ali Express EN",
        "topic": "middleeast",
        "tier": 2,
        "enabled": true,
        "region": "middleeast",
        "maxMessages": 25
      },
      {
        "handle": "iranintltv",
        "label": "Iran International",
        "topic": "politics",
        "tier": 2,
        "enabled": true,
        "region": "iran",
        "maxMessages": 20
      },
      {
        "handle": "kpszsu",
        "label": "Air Force of the Armed Forces of Ukraine",
        "topic": "alerts",
        "tier": 2,
        "enabled": true,
        "region": "ukraine",
        "maxMessages": 20
      },
      {
        "handle": "LiveUAMap",
        "label": "LiveUAMap",
        "topic": "breaking",
        "tier": 2,
        "enabled": true,
        "region": "global",
        "maxMessages": 25
      },
      {
        "handle": "OSINTdefender",
        "label": "OSINTdefender",
        "topic": "conflict",
        "tier": 2,
        "enabled": true,
        "region": "global",
        "maxMessages": 25
      },
      {
        "handle": "OsintUpdates",
        "label": "Osint Updates",
        "topic": "breaking",
        "tier": 2,
        "enabled": true,
        "region": "global",
        "maxMessages": 20
      },
      {
        "handle": "bellingcat",
        "label": "Bellingcat",
        "topic": "osint",
        "tier": 3,
        "enabled": true,
        "region": "global",
        "maxMessages": 10
      },
      {
        "handle": "CyberDetective",
        "label": "CyberDetective",
        "topic": "cyber",
        "tier": 3,
        "enabled": true,
        "region": "global",
        "maxMessages": 15
      },
      {
        "handle": "GeopoliticalCenter",
        "label": "GeopoliticalCenter",
        "topic": "geopolitics",
        "tier": 3,
        "enabled": true,
        "region": "global",
        "maxMessages": 20
      },
      {
        "handle": "Middle_East_Spectator",
        "label": "Middle East Spectator",
        "topic": "middleeast",
        "tier": 3,
        "enabled": true,
        "region": "middleeast",
        "maxMessages": 20
      },
      {
        "handle": "MiddleEastNow_Breaking",
        "label": "Middle East Now Breaking",
        "topic": "middleeast",
        "tier": 3,
        "enabled": true,
        "region": "middleeast",
        "maxMessages": 15
      },
      {
        "handle": "nexta_tv",
        "label": "NEXTA",
        "topic": "politics",
        "tier": 3,
        "enabled": true,
        "region": "europe",
        "maxMessages": 15
      },
      {
        "handle": "OSINTIndustries",
        "label": "OSINT Industries",
        "topic": "osint",
        "tier": 3,
        "enabled": true,
        "region": "global",
        "maxMessages": 15
      },
      {
        "handle": "Osintlatestnews",
        "label": "OSIntOps News",
        "topic": "osint",
        "tier": 3,
        "enabled": true,
        "region": "global",
        "maxMessages": 20
      },
      {
        "handle": "osintlive",
        "label": "OSINT Live",
        "topic": "osint",
        "tier": 3,
        "enabled": true,
        "region": "global",
        "maxMessages": 15
      },
      {
        "handle": "OsintTv",
        "label": "OsintTV",
        "topic": "geopolitics",
        "tier": 3,
        "enabled": true,
        "region": "global",
        "maxMessages": 15
      },
      {
        "handle": "spectatorindex",
        "label": "The Spectator Index",
        "topic": "breaking",
        "tier": 3,
        "enabled": true,
        "region": "global",
        "maxMessages": 15
      },
      {
        "handle": "wfwitness",
        "label": "Witness",
        "topic": "breaking",
        "tier": 3,
        "enabled": true,
        "region": "global",
        "maxMessages": 15
      },
      {
        "handle": "war_monitor",
        "label": "monitor",
        "topic": "alerts",
        "tier": 3,
        "enabled": true,
        "region": "ukraine",
        "maxMessages": 20
      },
      {
        "handle": "nayaforiraq",
        "label": "Naya for Iraq",
        "topic": "politics",
        "tier": 3,
        "enabled": true,
        "region": "middleeast",
        "maxMessages": 20
      },
      {
        "handle": "yediotnews25",
        "label": "Yedioth News",
        "topic": "breaking",
        "tier": 3,
        "enabled": true,
        "region": "middleeast",
        "maxMessages": 20
      },
      {
        "handle": "DDGeopolitics",
        "label": "DD Geopolitics",
        "topic": "geopolitics",
        "tier": 3,
        "enabled": true,
        "region": "global",
        "maxMessages": 20
      },
      {
        "handle": "FotrosResistancee",
        "label": "Fotros Resistance",
        "topic": "conflict",
        "tier": 3,
        "enabled": true,
        "region": "iran",
        "maxMessages": 20
      },
      {
        "handle": "RezistanceTrench1",
        "label": "Resistance Trench",
        "topic": "conflict",
        "tier": 3,
        "enabled": true,
        "region": "middleeast",
        "maxMessages": 20
      },
      {
        "handle": "geopolitics_prime",
        "label": "Geopolitics Prime",
        "topic": "geopolitics",
        "tier": 3,
        "enabled": true,
        "region": "global",
        "maxMessages": 20
      },
      {
        "handle": "thecradlemedia",
        "label": "The Cradle",
        "topic": "middleeast",
        "tier": 3,
        "enabled": true,
        "region": "middleeast",
        "maxMessages": 20
      },
      {
        "handle": "LebUpdate",
        "label": "Lebanon Update",
        "topic": "breaking",
        "tier": 3,
        "enabled": true,
        "region": "middleeast",
        "maxMessages": 20
      }
    ],
    "tech": [],
    "finance": []
  }
}
```

#### gamma-irradiators.json

The **IAEA** has recently published its updated **Database on Industrial Irradiation Facilities** (**DIIF**), featuring an interactive map with information on nearly 300 gamma irradiators and electron accelerators from around the world.

```json
{
  "source": "IAEA DIIF - Database on Industrial Irradiation Facilities",
  "tableauUrl": "https://public.tableau.com/app/profile/acceleratorknowledgeportal/viz/IrradiatorDatabase/Home",
  "extracted": "2026-01-09",
  "totalFacilities": 136,
  "note": "Gamma irradiator facilities worldwide. Coordinates extracted from Tableau Public visualization.",
  "facilities": [
    { "id": "gi-001", "city": "Vega Alta", "country": "Puerto Rico", "lat": 18.420295, "lon": -66.334995 },
    { "id": "gi-002", "city": "Northborough, MA", "country": "USA", "lat": 42.311091, "lon": -71.649221 },
    { "id": "gi-003", "city": "Chester, NY", "country": "USA", "lat": 41.323774, "lon": -74.289753 },
    { "id": "gi-136", "city": "Ibaraki", "country": "Japan", "lat": 5.677879, "lon": 140.48205 }
  ]
}
```

#### mirta-processed.json

```json
{
  "metadata": {
    "source": "MIRTA - Military Installations, Ranges and Training Areas",
    "url": "https://geospatial-usace.opendata.arcgis.com/maps/fc0f38c5a19a46dbacd92f2fb823ef8c",
    "fetchedAt": "2026-02-27T20:15:03.751Z",
    "totalInstallations": 832,
    "fromPoints": 737,
    "fromBoundariesOnly": 95
  },
  "installations": [
    {
      "name": "1LT John A Fera USARC",
      "branch": "Army Reserve",
      "status": "Active",
      "state": "Massachusetts",
      "lat": 42.589288,
      "lon": -70.943131,
      "kind": "Installation",
      "component": "Reserve",
      "jointBase": false
    },
    {
      "name": "Yukon Weapons Range",
      "branch": "Air Force",
      "status": "Active",
      "state": "Alaska",
      "lat": 64.768855128,
      "lon": -146.894347173,
      "kind": "Installation",
      "component": "Active",
      "jointBase": false
    },
    {
      "name": "Yuma Proving Ground",
      "branch": "Army",
      "status": "Active",
      "state": "Arizona",
      "lat": 33.1700688670001,
      "lon": -114.353450857,
      "kind": "Installation",
      "component": "Active",
      "jointBase": false
    }
  ]
}
```

#### .env.example

```
# ============================================
# World Monitor — Environment Variables
# ============================================
# Copy this file to .env.local and fill in the values you need.
# All keys are optional — the dashboard works without them,
# but the corresponding features will be disabled.
#
#   cp .env.example .env.local
#
# ============================================


# ------ AI Summarization (Vercel) ------

# Groq API (primary — 14,400 req/day on free tier)
# Get yours at: https://console.groq.com/
GROQ_API_KEY=

# OpenRouter API (fallback — 50 req/day on free tier)
# Get yours at: https://openrouter.ai/
OPENROUTER_API_KEY=


# ------ Cross-User Cache (Vercel — Upstash Redis) ------

# Used to deduplicate AI calls and cache risk scores across visitors.
# Create a free Redis database at: https://upstash.com/
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=


# ------ Market Data (Vercel) ------

# Finnhub (primary stock quotes — free tier available)
# Register at: https://finnhub.io/
FINNHUB_API_KEY=


# ------ Energy Data (Vercel) ------

# U.S. Energy Information Administration (oil prices, production, inventory)
# Register at: https://www.eia.gov/opendata/
EIA_API_KEY=


# ------ Economic Data (Vercel) ------

# FRED (Federal Reserve Economic Data)
# Register at: https://fred.stlouisfed.org/docs/api/api_key.html
FRED_API_KEY=


# ------ Aviation Intelligence (Vercel) ------

# AviationStack (live flight data, airport flights, carrier ops)
# Register at: https://aviationstack.com/
AVIATIONSTACK_API=

# ICAO API (NOTAM airport closures — optional, MENA region)
# Register at: https://applications.icao.int/
ICAO_API_KEY=

# Travelpayouts (flight price search — optional, demo only)
# Register at: https://www.travelpayouts.com/
TRAVELPAYOUTS_API_TOKEN=


# ------ Aircraft Tracking (Vercel) ------

# Wingbits aircraft enrichment (owner, operator, type)
# Contact: https://wingbits.com/
WINGBITS_API_KEY=


# ------ Conflict & Protest Data (Vercel) ------

# ACLED (Armed Conflict Location & Event Data — free for researchers)
# Register at: https://acleddata.com/
ACLED_ACCESS_TOKEN=

# UCDP (Uppsala Conflict Data Program — access token required since 2025)
# Register at: https://ucdp.uu.se/apidocs/
UCDP_ACCESS_TOKEN=


# ------ Internet Outages (Vercel) ------

# Cloudflare Radar API (requires free Cloudflare account with Radar access)
CLOUDFLARE_API_TOKEN=


# ------ Satellite Fire Detection (Vercel) ------

# NASA FIRMS (Fire Information for Resource Management System)
# Register at: https://firms.modaps.eosdis.nasa.gov/
NASA_FIRMS_API_KEY=


# ------ Railway Relay (scripts/ais-relay.cjs) ------
# The relay server handles AIS vessel tracking + OpenSky aircraft data + RSS proxy.
# It can also run the Telegram OSINT poller (stateful MTProto) when configured.
# Deploy on Railway with: node scripts/ais-relay.cjs

# AISStream API key for live vessel positions
# Get yours at: https://aisstream.io/
AISSTREAM_API_KEY=

# OpenSky Network OAuth2 credentials (higher rate limits for cloud IPs)
# Register at: https://opensky-network.org/
OPENSKY_CLIENT_ID=
OPENSKY_CLIENT_SECRET=


# ------ Telegram OSINT (Railway relay) ------
# Telegram MTProto keys (free): https://my.telegram.org/apps
TELEGRAM_API_ID=
TELEGRAM_API_HASH=

# GramJS StringSession generated locally (see: scripts/telegram/session-auth.mjs)
TELEGRAM_SESSION=

# Which curated list bucket to ingest: full | tech | finance
TELEGRAM_CHANNEL_SET=full

# ------ Railway Relay Connection (Vercel → Railway) ------

# Server-side URL (https://) — used by Vercel edge functions to reach the relay
WS_RELAY_URL=

# Optional client-side URL (wss://) — local/dev fallback only
VITE_WS_RELAY_URL=

# Shared secret between Vercel and Railway relay.
# Must be set to the SAME value on both platforms in production.
RELAY_SHARED_SECRET=

# Header name used to send the relay secret (must match on both platforms)
RELAY_AUTH_HEADER=x-relay-key

# Emergency production override to allow unauthenticated relay traffic.
# Leave unset/false in production.
ALLOW_UNAUTHENTICATED_RELAY=false

# Rolling window size (seconds) used by relay /metrics endpoint.
RELAY_METRICS_WINDOW_SECONDS=60


# ------ Public Data Sources (no keys required) ------

# UNHCR (UN Refugee Agency) — public API, no auth (CC BY 4.0)
# Open-Meteo — public API, no auth (processes Copernicus ERA5)
# WorldPop — public API, no auth needed


# ------ Site Configuration ------

# Site variant: "full" (worldmonitor.app) or "tech" (tech.worldmonitor.app)
VITE_VARIANT=full

# API base URL for web redirect. When set, browser fetch calls to /api/*
# are redirected to this URL. Leave empty for same-domain API (local installs).
# Production: https://api.worldmonitor.app
VITE_WS_API_URL=

# Client-side Sentry DSN (optional). Leave empty to disable error reporting.
VITE_SENTRY_DSN=

# Map interaction mode:
# - "flat" keeps pitch/rotation disabled (2D interaction)
# - "3d" enables pitch/rotation interactions (default)
VITE_MAP_INTERACTION_MODE=3d

# Self-hosted map tiles (optional — PMTiles on Cloudflare R2 or any HTTP server)
# Leave empty to use free OpenFreeMap tiles. Set to your own PMTiles URL for self-hosted tiles.
# See: https://protomaps.com/docs/pmtiles for how to generate PMTiles files.
VITE_PMTILES_URL=
# Public CORS-enabled URL for the same PMTiles file (used by Tauri desktop app).
# If your VITE_PMTILES_URL is behind a reverse proxy without CORS, set this to the
# direct R2/S3 public URL. The desktop app uses this URL; the web app uses VITE_PMTILES_URL.
VITE_PMTILES_URL_PUBLIC=


# ------ Desktop Cloud Fallback (Vercel) ------

# Comma-separated list of valid API keys for desktop cloud fallback.
# Generate with: openssl rand -hex 24 | sed 's/^/wm_/'
WORLDMONITOR_VALID_KEYS=


# ------ Registration DB (Convex) ------

# Convex deployment URL for email registration storage.
# Set up at: https://dashboard.convex.dev/
CONVEX_URL=
```

## Situation Monitor Dashboard

[A geopolitics-focused news aggregation dashboard with 3D globe visualization, designed for monitoring global situations, US policy changes, and conflict zones.](https://github.com/rgodse/situation-monitor)
## 其他

### 全球地缘政治事件时仪表盘工具汇总

链接： https://zhuanlan.zhihu.com/p/2012091361713730005

汇总一下提供全球新闻、市场和地缘政治事件的实时仪表盘类工具，方便跟踪当下的伊朗等国际地缘政治热点事件。

**Pentagon Pizza Watch & Polyglobe**

[https://www.pizzint.watch/](https://link.zhihu.com/?target=https%3A//www.pizzint.watch/)

五角大楼披萨指数，监测美国五角大楼周边披萨店指数，观察潜在的军事活动趋势

**Polyglobe**

[https://www.pizzint.watch/polyglobe](https://link.zhihu.com/?target=https%3A//www.pizzint.watch/polyglobe)

[https://www.pizzint.watch/about/polyglobe](https://link.zhihu.com/?target=https%3A//www.pizzint.watch/about/polyglobe)

Pentagon Pizza Watch 的 Polyglobe，用于可视化地缘政治预测市场的实时 [Polymarket](https://zhida.zhihu.com/search?content_id=270889834&content_type=Article&match_order=1&q=Polymarket&zhida_source=entity) 地图

**[Liveuamap](https://zhida.zhihu.com/search?content_id=270889834&content_type=Article&match_order=1&q=Liveuamap&zhida_source=entity)**

[https://www.liveuamap.com/](https://link.zhihu.com/?target=https%3A//www.liveuamap.com/)

全球冲突事件实时地图

**Glint**

[Glint - Real-time Prediction Market Intelligence](https://link.zhihu.com/?target=https%3A//glint.trade/)

专为预测市场交易者设计的实时开源情报仪表盘，将突发新闻、社会媒体信号、开源情报 和军事飞行数据实时映射到Polymarket等平台的合约赔率上

**[Situation Monitor](https://zhida.zhihu.com/search?content_id=270889834&content_type=Article&match_order=1&q=Situation+Monitor&zhida_source=entity)**

[https://hipcityreg-situation-monitor.vercel.app/](https://link.zhihu.com/?target=https%3A//hipcityreg-situation-monitor.vercel.app/)

[https://github.com/hipcityreg/situation-monitor](https://link.zhihu.com/?target=https%3A//github.com/hipcityreg/situation-monitor)

开源监控全球新闻、市场和地缘政治事件的实时仪表盘

**[Watchtower](https://zhida.zhihu.com/search?content_id=270889834&content_type=Article&match_order=1&q=Watchtower&zhida_source=entity)**

[https://github.com/lajosdeme/watchtower](https://link.zhihu.com/?target=https%3A//github.com/lajosdeme/watchtower)

开源基于终端的全球情报仪表盘

**[Iran Monitor](https://zhida.zhihu.com/search?content_id=270889834&content_type=Article&match_order=1&q=Iran+Monitor&zhida_source=entity)**

[https://www.iranmonitor.org/](https://link.zhihu.com/?target=https%3A//www.iranmonitor.org/)

一站式获取与伊朗有关可靠资讯的开源情报站，追踪伊朗局势、军事动态等信息的首选工具。

**[DeepStateMap](https://zhida.zhihu.com/search?content_id=270889834&content_type=Article&match_order=1&q=DeepStateMap&zhida_source=entity)**

[https://deepstatemap.live/](https://link.zhihu.com/?target=https%3A//deepstatemap.live/)

专注于俄乌战争前线动态的开源地图，目前公认对乌克兰前线局势更新最细致、最准确的地图之一

**[ACLED](https://zhida.zhihu.com/search?content_id=270889834&content_type=Article&match_order=1&q=ACLED&zhida_source=entity)**

[https://acleddata.com/platform/explorer](https://link.zhihu.com/?target=https%3A//acleddata.com/platform/explorer)

全球冲突数据分析，提供全球政治暴力、示威和抗议事件的数据库，并提供交互式地图

**[UnderstandingWar](https://zhida.zhihu.com/search?content_id=270889834&content_type=Article&match_order=1&q=UnderstandingWar&zhida_source=entity)**

[https://understandingwar.org/](https://link.zhihu.com/?target=https%3A//understandingwar.org/)

提供深入战况分析报告、地缘政治解读和交互式地图产品

[https://yeeach.com/2975/](https://link.zhihu.com/?target=https%3A//yeeach.com/2975/)













