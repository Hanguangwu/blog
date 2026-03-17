---
title: 多智能体系统架构设计案例
description: 本文介绍一些多智能体系统实现的案例。
date: 2026-03-17T12:34:25-08:00
draft: false
categories:
- AI
- Agent
tags:
- Agent Agency
- MetaGPT
- AutoGen
---

# 多智能体系统架构设计案例

## 前言

这里搜集了一些多智能体系统实现的案例。

## 框架流——MetaGPT

MetaGPT是一个由中国团队开发的多智能体框架，主要应用于软件开发等场景。其利用SOP（Standard Operating Procedures，标准作业程序）来协调基于大语言模型的多智能体系统，从而实现元编程技术。 该框架使用智能体模拟了一个虚拟软件团队，包含产品经理、架构师、项目经理、工程师、质量工程师等角色，并引入SOP成为框架的虚拟软件团队的开发流程。

[🌟 The Multi-Agent Framework: First AI Software Company, Towards Natural Language Programming](https://github.com/FoundationAgents/MetaGPT)

## 框架流——AutoGen

​AutoGen​ 是微软开发的一个用于创建多智能体AI应用 程序的编程框架，支持自主运行或与人类协同工作。该框架提供了分层和可扩展的设计，使开发者能够在不同抽象级别上构建复杂的多智能体系统。

[A programming framework for agentic AI](https://github.com/microsoft/autogen)

### 🚀 ​核心价值​

多智能体 · AI框架 · 微软开发 · 分层架构 · 开源免费

### 项目背景​

​AI智能体需求​：应对多智能体系统开发需求

​企业级应用​：企业级AI应用支持

​微软生态​：微软AI生态系统发展

​开源社区​：活跃开源社区支持

​开发效率​：提高智能体开发效率

### 项目特色​

🤖 ​多智能体​：多智能体协作系统

🏗️ ​分层架构​：分层可扩展架构

🔌 ​扩展生态​：丰富扩展生态系统

🌐 ​跨语言​：.NET和Python支持

🆓 ​开源免费​：MIT许可证开源

### 技术亮点​

​消息传递​：智能消息传递机制

​事件驱动​：事件驱动架构

​分布式运行​：本地和分布式运行时

​工具集成​：丰富工具集成

​开发工具​：完整开发工具链

**关键优势：**

✅ 管理企业目标，而不是 Pull Requests  
✅ 任何代理、任何运行时、一个组织图  
✅ 每个任务追溯到公司使命  
✅ 心跳自动执行，委托流程自动化  
✅ 月度预算，超支自动停止  
✅ 多公司隔离，完整数据隔离  
✅ 完整审计追踪，每个决策可解释  
✅ 人工监督，随时干预

## 提示词流——Agent Agency ZH

[176 个即插即用的 AI 专家人设（中文版）— 覆盖 17 个部门，支持 Claude Code / OpenClaw / Cursor / Trae 等 11 种工具 | Chinese AI agent personas for dev tools](https://github.com/jnMetaCode/agency-agents-zh)

### 🚀 这是什么？

源自 Reddit 帖子并经过数月的迭代，**The Agency** 是一个不断增长的任务交付型 AI 智能体人格合集。每位智能体都具备：

- **🎯 专业化**：在其领域拥有深厚的专业知识（而非通用的提示词模板）
- **🧠 人格驱动**：拥有独特的语气、沟通风格和处理方式
- **📋 交付导向**：产出真实的代码、流程和可衡量的成果
- **✅ 生产就绪**：经过实战检验的工作流和成功指标

**你可以把它看作**：组建你的梦想团队，只不过他们是永不疲倦、从不抱怨且使命必达的 AI 专家。

---

### ⚡ 快速入门

#### 选项 1：配合 Claude Code 使用（推荐）

```shell
# 将智能体复制到你的 Claude Code 目录
cp -r agency-agents/* ~/.claude/agents/

# 现在可以在 Claude Code 会话中激活任何智能体：
# “嘿 Claude，激活前端开发模式，帮我构建一个 React 组件”
```

#### 选项 2：作为参考使用

每个智能体文件包含：

- 身份与人格特质
- 核心使命与工作流
- 带有代码示例的技术交付物
- 成功指标与沟通风格

Browse the agents below and copy/adapt the ones you need! 浏览下方的智能体列表，复制或改编你需要的角色！

#### 选项 3：配合其他工具使用 (Cursor, Aider, Windsurf, Gemini CLI, OpenCode)

```shell
# 第 1 步 —— 为所有受支持的工具生成集成文件
./scripts/convert.sh

# 第 2 步 —— 交互式安装（自动检测你已安装的工具）
./scripts/install.sh

# 或者直接安装到特定工具
./scripts/install.sh --tool cursor
./scripts/install.sh --tool aider
./scripts/install.sh --tool windsurf
```

详见下方的 [多工具集成](https://github.com/gosinkx/agency-agents_zhCN#-%E5%A4%9A%E5%B7%A5%E5%85%B7%E9%9B%86%E6%88%90) 章节。

---

### 🎨 智能体阵容 (The Agency Roster)
#### 🛠️ 工程部

构建未来，一个 commit 一个脚印。

|智能体|专长|适用场景|
|---|---|---|
|[前端开发者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-frontend-developer.md)|React/Vue、UI 实现、性能优化|现代 Web 应用、像素级 UI|
|[后端架构师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-backend-architect.md)|API 设计、数据库架构、可扩展性|服务端系统、微服务|
|[AI 工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-ai-engineer.md)|机器学习、模型部署、AI 集成|ML 功能、数据管线|
|[DevOps 自动化](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-devops-automator.md)|CI/CD、基础设施自动化|流水线开发、部署自动化|
|[安全工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-security-engineer.md)|威胁建模、代码审计、安全架构|应用安全、漏洞评估|
|[快速原型师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-rapid-prototyper.md)|快速 POC、MVP 开发|概念验证、黑客马拉松|
|[高级开发者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-senior-developer.md)|Laravel/Livewire/FluxUI、高端 CSS、Three.js|高品质 Web 体验|
|[移动应用开发者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-mobile-app-builder.md)|iOS/Android 原生、跨平台框架|移动端开发、App 性能优化|
|[数据工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-data-engineer.md)|ETL/ELT、数据湖、Spark/dbt|数据管线、数据仓库|
|[技术文档工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-technical-writer.md)|API 文档、开发者文档、docs-as-code|技术文档、知识库|
|[自主优化架构师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-autonomous-optimization-architect.md)|自适应系统、自动调优|智能运维、自愈系统|
|[嵌入式固件工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-embedded-firmware-engineer.md)|RTOS、外设驱动、低功耗设计|IoT、嵌入式系统|
|[故障响应指挥官](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-incident-response-commander.md)|故障处置、SLO 管理、事后复盘|线上故障、应急响应|
|[威胁检测工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-threat-detection-engineer.md)|SIEM、威胁狩猎、检测规则|安全运营、威胁检测|
|[Solidity 智能合约工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-solidity-smart-contract-engineer.md)|Solidity、EVM、Gas 优化、DeFi|智能合约开发、Web3|
|[微信小程序开发者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-wechat-mini-program-developer.md) ⭐|WXML/WXSS、微信支付、云开发|微信小程序全栈开发|
|[代码审查员](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-code-reviewer.md)|代码审查、安全审计、质量把关|PR 审查、代码质量|
|[数据库优化师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-database-optimizer.md)|Schema 设计、查询优化、索引策略|数据库性能调优|
|[Git 工作流大师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-git-workflow-master.md)|分支策略、约定式提交、变基|Git 工作流规范|
|[软件架构师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-software-architect.md)|系统设计、DDD、架构决策|系统架构设计|
|[SRE](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-sre.md)|SLO、可观测性、混沌工程|站点可靠性工程|
|[AI 数据修复工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-ai-data-remediation-engineer.md)|自愈管道、SLM 语义聚类、零数据丢失|大规模数据异常修复|
|[飞书集成开发工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-feishu-integration-developer.md) ⭐|飞书机器人、审批流、多维表格|飞书生态集成开发|
|[钉钉集成开发工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/engineering/engineering-dingtalk-integration-developer.md) ⭐|钉钉机器人、酷应用、连接器|钉钉生态集成开发|

#### 🎨 设计部

让产品好看、好用、有惊喜。

|智能体|专长|适用场景|
|---|---|---|
|[UI 设计师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/design/design-ui-designer.md)|视觉设计、组件库、设计系统|界面设计、品牌一致性|
|[UX 研究员](https://github.com/jnMetaCode/agency-agents-zh/blob/main/design/design-ux-researcher.md)|用户测试、行为分析|用户研究、可用性测试|
|[UX 架构师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/design/design-ux-architect.md)|信息架构、交互设计、导航系统|复杂产品的 UX 架构|
|[品牌守护者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/design/design-brand-guardian.md)|品牌标识、一致性、定位|品牌策略、视觉规范|
|[图像提示词工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/design/design-image-prompt-engineer.md)|AI 图像生成、提示词优化|Midjourney/DALL-E 出图|
|[视觉叙事师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/design/design-visual-storyteller.md)|数据可视化、视觉叙事|信息图、演示文稿|
|[趣味注入师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/design/design-whimsy-injector.md)|微交互、彩蛋、趣味元素|产品细节体验提升|
|[包容性视觉专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/design/design-inclusive-visuals-specialist.md)|多元化视觉、无障碍设计|包容性设计、全球化视觉|

#### 📢 营销部

一个真实互动一个粉丝地增长。

**国内平台：**

|智能体|专长|适用场景|
|---|---|---|
|[小红书运营](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-xiaohongshu-operator.md) ⭐|种草笔记、达人合作、爆款内容|小红书获客、品牌种草|
|[抖音策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-douyin-strategist.md) ⭐|短视频策划、算法优化、直播带货|抖音增长、短视频营销|
|[微信公众号运营](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-wechat-operator.md) ⭐|公众号内容、社群运营、裂变增长|微信生态营销|
|[B站内容策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-bilibili-strategist.md) ⭐|UP主运营、弹幕文化、中长视频|B站内容增长、品牌合作|
|[快手策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-kuaishou-strategist.md) ⭐|下沉市场、老铁文化、直播电商|快手运营、社区信任|
|[中国电商运营师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-china-ecommerce-operator.md)|淘宝/拼多多/京东、广告投放、大促作战|电商全链路深度运营|
|[电商运营师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-ecommerce-operator.md) ⭐|淘宝/拼多多/京东、直播带货、大促|电商全平台运营（简洁版）|
|[百度 SEO 专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-baidu-seo-specialist.md) ⭐|百度优化、百科/知道/贴吧生态|百度搜索营销|
|[私域流量运营师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-private-domain-operator.md) ⭐|企微SCRM、社群运营、用户生命周期|私域体系搭建、复购增长|
|[直播电商主播教练](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-livestream-commerce-coach.md) ⭐|直播话术、选品排品、千川投放|直播带货、主播孵化|
|[跨境电商运营专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-cross-border-ecommerce.md) ⭐|Amazon/Shopee/Lazada、海外仓、品牌出海|跨境电商全链路运营|
|[短视频剪辑指导师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-short-video-editing-coach.md) ⭐|剪映/PR/达芬奇、调色、音频、特效|短视频剪辑技术指导|
|[微博运营策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-weibo-strategist.md) ⭐|热搜运营、超话、舆情公关、粉丝经济|微博全链路运营|
|[播客内容策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-podcast-strategist.md) ⭐|小宇宙/喜马拉雅、音频制作、商业化|播客内容创作与增长|
|[微信视频号运营策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-weixin-channels-strategist.md) ⭐|视频号直播、社交裂变、私域闭环|视频号运营与变现|
|[知识付费产品策划师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-knowledge-commerce-strategist.md) ⭐|得到/知识星球/小鹅通、内容定价|知识付费产品运营|
|[小红书专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-xiaohongshu-specialist.md)|生活方式内容、趋势策略|小红书品牌建设|
|[微信公众号管理](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-wechat-official-account.md)|订阅者运营、内容营销|微信公众号增长|
|[知乎策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-zhihu-strategist.md)|知识型内容、思想领袖建设|知乎品牌权威|

> ⭐ 标记的是本项目原创，更贴合国内实操。其余为上游英文版翻译。

**出海营销：**

|智能体|专长|适用场景|
|---|---|---|
|[TikTok 策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-tiktok-strategist.md)|病毒式内容、算法优化|出海短视频营销|
|[Twitter 互动官](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-twitter-engager.md)|实时互动、思想领袖|出海品牌社交|
|[Instagram 策展师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-instagram-curator.md)|视觉叙事、社区运营|出海视觉营销|
|[Reddit 社区运营](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-reddit-community-builder.md)|社区文化、真实互动|出海社区营销|
|[应用商店优化师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-app-store-optimizer.md)|ASO、转化优化|App 出海推广|

**通用：**

|智能体|专长|适用场景|
|---|---|---|
|[增长黑客](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-growth-hacker.md)|快速获客、病毒循环、实验|用户增长、转化优化|
|[内容创作者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-content-creator.md)|多平台内容、编辑日历|内容策略、品牌故事|
|[社交媒体策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-social-media-strategist.md)|跨平台策略、整合营销|全渠道社交运营|
|[SEO 专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-seo-specialist.md)|搜索引擎优化、技术 SEO|Google SEO、内容优化|
|[轮播图增长引擎](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-carousel-growth-engine.md)|轮播图内容、自动化投放|社交媒体轮播素材|
|[LinkedIn 内容创作专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-linkedin-content-creator.md)|LinkedIn 职场内容、B2B 获客|LinkedIn 品牌建设|
|[图书联合作者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/marketing/marketing-book-co-author.md)|思想领袖力图书、代笔协作|图书策划与撰写|

### 💰 付费媒体部

精准投放，每一分预算都花在刀刃上。

|智能体|专长|适用场景|
|---|---|---|
|[付费媒体审计师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/paid-media/paid-media-auditor.md)|广告账户审计、预算优化|广告效果诊断、降本增效|
|[广告创意策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/paid-media/paid-media-creative-strategist.md)|广告素材策划、A/B 测试|广告创意优化|
|[社交广告策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/paid-media/paid-media-paid-social-strategist.md)|社交平台广告投放|Meta/TikTok/LinkedIn 广告|
|[PPC 竞价策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/paid-media/paid-media-ppc-strategist.md)|搜索竞价、关键词管理|Google Ads、百度推广|
|[程序化广告采买专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/paid-media/paid-media-programmatic-buyer.md)|DSP、RTB、程序化购买|程序化广告投放|
|[搜索词分析师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/paid-media/paid-media-search-query-analyst.md)|搜索词挖掘、否词优化|搜索广告精细化运营|
|[追踪与归因专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/paid-media/paid-media-tracking-specialist.md)|转化追踪、归因模型|广告效果衡量、数据打通|

### 💼 销售部

从线索到成交，让每一单都有章法。

|智能体|专长|适用场景|
|---|---|---|
|[客户拓展策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/sales/sales-account-strategist.md)|大客户拓展、ABM 策略|重点客户攻关|
|[销售教练](https://github.com/jnMetaCode/agency-agents-zh/blob/main/sales/sales-coach.md)|销售辅导、技能提升|团队销售能力建设|
|[赢单策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/sales/sales-deal-strategist.md)|成交策略、MEDDPICC|复杂销售推进|
|[Discovery 教练](https://github.com/jnMetaCode/agency-agents-zh/blob/main/sales/sales-discovery-coach.md)|需求挖掘、客户洞察|销售前期沟通|
|[售前工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/sales/sales-engineer.md)|技术方案、Demo 演示|技术售前支持|
|[Outbound 策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/sales/sales-outbound-strategist.md)|外呼策略、Cold outreach|新客户开拓|
|[Pipeline 分析师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/sales/sales-pipeline-analyst.md)|销售漏斗、预测分析|销售数据分析、预测|
|[投标策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/sales/sales-proposal-strategist.md)|投标方案、提案撰写|招投标、方案竞标|

#### 🏦 金融部

让每一笔钱都清清楚楚。

|智能体|专长|适用场景|
|---|---|---|
|[财务预测分析师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/finance/finance-financial-forecaster.md) ⭐|收入预测、场景建模、现金流|SaaS 财务规划、融资对接|
|[发票管理专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/finance/finance-invoice-manager.md) ⭐|增值税发票、金税系统、三单匹配|发票全生命周期管理|
|[金融风控分析师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/finance/finance-fraud-detector.md) ⭐|交易风控、反洗钱、电信诈骗|支付风控、合规审查|

#### 👔 人力资源部

找对人、用好人、留住人。

|智能体|专长|适用场景|
|---|---|---|
|[招聘专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/hr/hr-recruiter.md) ⭐|Boss直聘/猎聘、校招社招、背调|招聘全流程管理|
|[绩效管理专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/hr/hr-performance-reviewer.md) ⭐|OKR/KPI、361分布、晋升答辩|绩效体系搭建与评估|

#### ⚖️ 法务部

合规是底线，风控是生命线。

|智能体|专长|适用场景|
|---|---|---|
|[合同审查专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/legal/legal-contract-reviewer.md) ⭐|民法典合同编、电子签章、风险评估|合同审查与风控|
|[制度文件撰写专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/legal/legal-policy-writer.md) ⭐|PIPL/数据安全法、隐私政策|合规制度与政策撰写|

#### 🚚 供应链部


从工厂到用户，每一环都不掉链子。

|智能体|专长|适用场景|
|---|---|---|
|[库存预测专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/supply-chain/supply-chain-inventory-forecaster.md) ⭐|需求预测、安全库存、618/双11备货|库存管理与补货优化|
|[供应商评估专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/supply-chain/supply-chain-vendor-evaluator.md) ⭐|1688供应商、验厂、国标质检|供应商准入与分级管理|
|[物流路线优化师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/supply-chain/supply-chain-route-optimizer.md) ⭐|顺丰/通达系、冷链、跨境物流|物流成本优化与路线规划|

#### 📦 产品部

在正确的时间做正确的事。

|智能体|专长|适用场景|
|---|---|---|
|[Sprint 排序师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/product/product-sprint-prioritizer.md)|敏捷规划、功能优先级|Sprint 规划、资源分配|
|[趋势研究员](https://github.com/jnMetaCode/agency-agents-zh/blob/main/product/product-trend-researcher.md)|市场情报、竞品分析|市场调研、机会评估|
|[反馈分析师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/product/product-feedback-synthesizer.md)|用户反馈分析、洞察提取|反馈分析、产品优先级|
|[行为助推引擎](https://github.com/jnMetaCode/agency-agents-zh/blob/main/product/product-behavioral-nudge-engine.md)|行为心理学、用户引导|用户行为设计、转化提升|
|[产品经理](https://github.com/jnMetaCode/agency-agents-zh/blob/main/product/product-manager.md)|产品全生命周期、PRD、路线图|产品策略与交付管理|

#### 📋 项目管理部


让项目按时按质交付。

|智能体|专长|适用场景|
|---|---|---|
|[高级项目经理](https://github.com/jnMetaCode/agency-agents-zh/blob/main/project-management/project-manager-senior.md)|需求拆解、范围管控|大型项目管理|
|[项目牧羊人](https://github.com/jnMetaCode/agency-agents-zh/blob/main/project-management/project-management-project-shepherd.md)|跨团队协调、进度跟踪|多团队项目协调|
|[实验追踪员](https://github.com/jnMetaCode/agency-agents-zh/blob/main/project-management/project-management-experiment-tracker.md)|A/B 测试、实验管理|数据驱动决策|
|[工作室制片人](https://github.com/jnMetaCode/agency-agents-zh/blob/main/project-management/project-management-studio-producer.md)|创意项目管理、资源调度|内容/创意项目|
|[工作室运营](https://github.com/jnMetaCode/agency-agents-zh/blob/main/project-management/project-management-studio-operations.md)|工作室日常运营管理|团队运营效率|
|[Jira 工作流管家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/project-management/project-management-jira-workflow-steward.md)|Jira 配置、工作流优化|Jira 项目管理|

#### 🧪 测试部


打破一切，让用户不必承受。

|智能体|专长|适用场景|
|---|---|---|
|[证据收集者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/testing/testing-evidence-collector.md)|截图 QA、视觉验证|UI 测试、Bug 文档|
|[现实检验者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/testing/testing-reality-checker.md)|证据驱动认证、质量关卡|生产就绪评估|
|[API 测试员](https://github.com/jnMetaCode/agency-agents-zh/blob/main/testing/testing-api-tester.md)|API 验证、集成测试|接口测试、端点验证|
|[性能基准师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/testing/testing-performance-benchmarker.md)|性能测试、优化|压测、性能调优|
|[无障碍审核员](https://github.com/jnMetaCode/agency-agents-zh/blob/main/testing/testing-accessibility-auditor.md)|WCAG 审核、辅助技术测试|无障碍合规、包容性设计|
|[测试结果分析师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/testing/testing-test-results-analyzer.md)|测试数据分析、质量度量|质量趋势、发布决策|
|[工具评估师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/testing/testing-tool-evaluator.md)|工具选型、功能对比|技术选型、工具采购|
|[工作流优化师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/testing/testing-workflow-optimizer.md)|流程分析、自动化|效率提升、流程改进|

#### 🤝 支持部


运营的中流砥柱。

|智能体|专长|适用场景|
|---|---|---|
|[客服响应者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/support/support-support-responder.md)|客户服务、工单处理|客户支持、用户体验|
|[数据分析师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/support/support-analytics-reporter.md)|数据分析、仪表盘|商业智能、KPI 追踪|
|[法务合规员](https://github.com/jnMetaCode/agency-agents-zh/blob/main/support/support-legal-compliance-checker.md)|合规审查、法规检查|法律合规、风险管理|
|[高管摘要师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/support/support-executive-summary-generator.md)|业务摘要、战略沟通|高管汇报、决策支持|
|[财务追踪员](https://github.com/jnMetaCode/agency-agents-zh/blob/main/support/support-finance-tracker.md)|财务分析、预算管理|财务规划、成本管控|
|[基础设施运维师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/support/support-infrastructure-maintainer.md)|系统运维、可靠性工程|基础设施管理、故障排查|
|[招聘运营专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/support/support-recruitment-specialist.md) ⭐|Boss直聘/猎聘、劳动法、校招社招|招聘全流程与HR合规|
|[供应链采购策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/support/support-supply-chain-strategist.md) ⭐|1688采购、质检、供应商管理、ERP|供应链与采购管理|

#### 🔬 专项部

不走寻常路的专家。

|智能体|专长|适用场景|
|---|---|---|
|[智能体编排者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/agents-orchestrator.md)|多智能体协调、工作流管理|复杂项目的多智能体协作|
|[提示词工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/prompt-engineer.md) ⭐|LLM 提示词设计、优化、评测|提示词开发、AI 应用优化|
|[身份信任架构师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/agentic-identity-trust.md)|AI 身份验证、信任框架|AI 系统安全与信任|
|[数据整合师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/data-consolidation-agent.md)|多源数据整合、仪表盘|数据汇总与可视化|
|[LSP 索引工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/lsp-index-engineer.md)|代码智能、语义索引|代码导航、IDE 集成|
|[报告分发师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/report-distribution-agent.md)|报告分发、多渠道推送|自动化报告分发|
|[销售数据提取师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/sales-data-extraction-agent.md)|销售数据采集、结构化|CRM 数据处理|
|[合规审计师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/compliance-auditor.md)|SOC 2/ISO 27001/HIPAA 合规|合规审计、安全认证|
|[应付账款智能体](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/accounts-payable-agent.md)|发票处理、付款自动化|财务流程自动化|
|[身份图谱操作员](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/identity-graph-operator.md)|身份解析、多源匹配|用户身份治理|
|[文化智能策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/specialized-cultural-intelligence-strategist.md)|文化洞察、跨文化设计|全球化产品、本地化策略|
|[开发者布道师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/specialized-developer-advocate.md)|开发者关系、DX 工程|开发者社区、技术推广|
|[模型 QA 专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/specialized-model-qa.md)|ML 模型审计、质量验证|模型上线前检查|
|[ZK 管家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/zk-steward.md)|Zettelkasten 知识管理|知识库构建、笔记系统|
|[区块链安全审计师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/blockchain-security-auditor.md)|智能合约审计、漏洞检测|合约安全、DeFi 审计|
|[留学规划顾问](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/study-abroad-advisor.md) ⭐|多国申请策略、选校定位|留学规划、文书指导|
|[政务数字化售前顾问](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/government-digital-presales-consultant.md) ⭐|方案设计、标书、等保/信创|政务ToG项目售前|
|[企业培训课程设计师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/corporate-training-designer.md) ⭐|ADDIE/SAM、企业学习平台、TTT|培训体系搭建与课程开发|
|[MCP 构建器](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/specialized-mcp-builder.md)|MCP 服务器、工具设计、API 集成|MCP 开发、AI 工具扩展|
|[文档生成器](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/specialized-document-generator.md)|PDF/PPTX/DOCX/XLSX 生成|程序化文档创建|
|[工作流架构师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/specialized-workflow-architect.md)|工作流树设计、交接契约、故障恢复|系统流程规格化|
|[自动化治理架构师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/automation-governance-architect.md)|自动化审计、n8n 工作流治理、风险评估|业务自动化决策|
|[Salesforce 架构师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/specialized-salesforce-architect.md)|Salesforce 多云设计、集成、数据模型|企业级 Salesforce 架构|
|[医疗健康营销合规师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/healthcare-marketing-compliance.md) ⭐|医疗广告法、NMPA、互联网医疗|医疗健康营销合规|
|[高考志愿填报顾问](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/gaokao-college-advisor.md) ⭐|平行志愿、位次法、冲稳保策略|高考志愿填报规划|
|[动态定价策略师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/specialized-pricing-optimizer.md) ⭐|淘宝/京东/拼多多定价、大促机制|电商定价与促销策略|
|[AI 治理政策专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/specialized-ai-policy-writer.md) ⭐|算法备案、生成式AI管理、伦理审查|AI 合规与治理框架|
|[企业风险评估师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/specialized-risk-assessor.md) ⭐|COSO本土化、国企风控、ESG|企业风险管理与审计|
|[会议效率专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/specialized/specialized-meeting-assistant.md) ⭐|飞书/钉钉/腾讯会议、OKR周会|会议管理与纪要输出|

#### 🥽 空间计算部

构建下一代空间交互体验。

|智能体|专长|适用场景|
|---|---|---|
|[visionOS 空间工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/spatial-computing/visionos-spatial-engineer.md)|visionOS、SwiftUI 空间 UI|Apple Vision Pro 开发|
|[macOS Metal 空间工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/spatial-computing/macos-spatial-metal-engineer.md)|Metal、GPU 渲染|macOS 高性能图形|
|[XR 界面架构师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/spatial-computing/xr-interface-architect.md)|空间 UI 架构、交互设计|XR 应用界面设计|
|[XR 沉浸式开发者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/spatial-computing/xr-immersive-developer.md)|WebXR、沉浸式体验|VR/AR 应用开发|
|[XR 座舱交互专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/spatial-computing/xr-cockpit-interaction-specialist.md)|座舱 UI、多模态交互|汽车/航空 XR 交互|
|[终端集成专家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/spatial-computing/terminal-integration-specialist.md)|终端模拟、系统集成|空间计算终端工具|

#### 🎮 游戏开发部

从独立游戏到 3A 大作，全引擎覆盖。

**通用：**

|智能体|专长|适用场景|
|---|---|---|
|[游戏设计师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/game-designer.md)|游戏机制、系统设计、平衡性|游戏核心玩法设计|
|[关卡设计师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/level-designer.md)|关卡布局、节奏控制、空间叙事|关卡设计、场景构建|
|[叙事设计师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/narrative-designer.md)|剧情设计、对话系统、世界观|游戏剧情、互动叙事|
|[技术美术](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/technical-artist.md)|Shader、渲染管线、美术工具|画面效果、性能优化|
|[游戏音频工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/game-audio-engineer.md)|音效设计、音频引擎、空间音频|游戏音效、配乐|

**Unity：**

|智能体|专长|适用场景|
|---|---|---|
|[Unity 架构师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/unity/unity-architect.md)|Unity 架构、ECS、性能优化|Unity 项目架构|
|[Unity 编辑器工具开发者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/unity/unity-editor-tool-developer.md)|编辑器扩展、自定义工具|Unity 工具链开发|
|[Unity 多人游戏工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/unity/unity-multiplayer-engineer.md)|Netcode、同步、网络架构|Unity 联机游戏|
|[Unity Shader Graph 美术师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/unity/unity-shader-graph-artist.md)|Shader Graph、URP/HDRP|Unity 视觉效果|

**Unreal Engine：**

|智能体|专长|适用场景|
|---|---|---|
|[Unreal 多人游戏架构师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/unreal-engine/unreal-multiplayer-architect.md)|Replication、网络同步|UE 联机架构|
|[Unreal 系统工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/unreal-engine/unreal-systems-engineer.md)|Gameplay 框架、C++ 系统|UE 核心系统开发|
|[Unreal 技术美术](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/unreal-engine/unreal-technical-artist.md)|材质、Niagara、渲染管线|UE 画面与性能|
|[Unreal 世界构建师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/unreal-engine/unreal-world-builder.md)|开放世界、地形、关卡串流|UE 场景构建|

**Blender：**

|智能体|专长|适用场景|
|---|---|---|
|[Blender 插件工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/blender/blender-addon-engineer.md)|Python 插件、资源验证、导出自动化|Blender 管线工具开发|

**Godot：**

|智能体|专长|适用场景|
|---|---|---|
|[Godot 游戏脚本开发者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/godot/godot-gameplay-scripter.md)|GDScript、场景树、信号系统|Godot 游戏逻辑|
|[Godot 多人游戏工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/godot/godot-multiplayer-engineer.md)|MultiplayerAPI、网络同步|Godot 联机游戏|
|[Godot Shader 开发者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/godot/godot-shader-developer.md)|Godot Shader Language、视觉效果|Godot 画面效果|

**Roblox Studio：**

|智能体|专长|适用场景|
|---|---|---|
|[Roblox 虚拟形象创作者](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/roblox-studio/roblox-avatar-creator.md)|虚拟形象、UGC 资产|Roblox 角色设计|
|[Roblox 体验设计师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/roblox-studio/roblox-experience-designer.md)|体验设计、游戏循环|Roblox 游戏设计|
|[Roblox 系统脚本工程师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/game-development/roblox-studio/roblox-systems-scripter.md)|Luau 脚本、数据存储|Roblox 游戏开发|

#### 📖 学术部

为叙事设计、世界构建和文化研究提供学术级支撑。

|智能体|专长|适用场景|
|---|---|---|
|[人类学家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/academic/academic-anthropologist.md)|文化体系、仪式、民族志|世界观设计、文化构建|
|[地理学家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/academic/academic-geographer.md)|自然与人文地理、空间分析|地图构建、场景设计|
|[历史学家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/academic/academic-historian.md)|历史分析、史料考证|历史题材验证、年代设定|
|[叙事学家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/academic/academic-narratologist.md)|叙事理论、故事结构|剧情设计、角色弧线|
|[心理学家](https://github.com/jnMetaCode/agency-agents-zh/blob/main/academic/academic-psychologist.md)|行为心理、人格理论|角色心理塑造、动机设计|
|[学习规划师](https://github.com/jnMetaCode/agency-agents-zh/blob/main/academic/academic-study-planner.md) ⭐|考研/考公/法考备考、学习方法论|个性化学习计划与备考规划|

#### 🎯 战略部

从发现到运营的全流程战略指导。详见 [strategy/](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy) 目录。

| 文档                                                                                                                     | 内容                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [高管简报](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/EXECUTIVE-BRIEF.md)                           | NEXUS 战略概览                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| [快速上手](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/QUICKSTART.md)                                | 5 分钟上手指南                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| [完整战略](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/nexus-strategy.md)                            | 运营纲领全文                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| [智能体激活提示词](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/coordination/agent-activation-prompts.md) | 各智能体的激活指令                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| [交接模板](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/coordination/handoff-templates.md)            | 智能体间的交接规范                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Phase 0-6 Playbooks                                                                                                    | [发现](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/playbooks/phase-0-discovery.md) · [策略](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/playbooks/phase-1-strategy.md) · [基础](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/playbooks/phase-2-foundation.md) · [构建](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/playbooks/phase-3-build.md) · [加固](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/playbooks/phase-4-hardening.md) · [上线](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/playbooks/phase-5-launch.md) · [运营](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/playbooks/phase-6-operate.md) |
| 场景 Runbook                                                                                                             | [创业 MVP](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/runbooks/scenario-startup-mvp.md) · [企业功能](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/runbooks/scenario-enterprise-feature.md) · [事故响应](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/runbooks/scenario-incident-response.md) · [营销活动](https://github.com/jnMetaCode/agency-agents-zh/blob/main/strategy/runbooks/scenario-marketing-campaign.md)                                                                                                                                                                                                                                                                             |

## 提示词流——Agent Agency

[A complete AI agency at your fingertips - From frontend wizards to Reddit community ninjas, from whimsy injectors to reality checkers. Each agent is a specialized expert with personality, processes, and proven deliverables.](https://github.com/msitarzewski/agency-agents)

### 🗺️ Roadmap

- [ ]  Interactive agent selector web tool
- [x]  Multi-agent workflow examples -- see [examples/](https://github.com/msitarzewski/agency-agents/blob/main/examples)
- [x]  Multi-tool integration scripts (Claude Code, GitHub Copilot, Antigravity, Gemini CLI, OpenCode, OpenClaw, Cursor, Aider, Windsurf, Qwen Code)
- [ ]  Video tutorials on agent design
- [ ]  Community agent marketplace
- [ ]  Agent "personality quiz" for project matching
- [ ]  "Agent of the Week" showcase series

---

### 🌐 Community Translations & Localizations

Community-maintained translations and regional adaptations. These are independently maintained -- see each repo for coverage and version compatibility.

|Language|Maintainer|Link|Notes|
|---|---|---|---|
|🇨🇳 简体中文 (zh-CN)|[@jnMetaCode](https://github.com/jnMetaCode)|[agency-agents-zh](https://github.com/jnMetaCode/agency-agents-zh)|100 translated agents + 9 China-market originals|
|🇨🇳 简体中文 (zh-CN)|[@dsclca12](https://github.com/dsclca12)|[agent-teams](https://github.com/dsclca12/agent-teams)|Independent translation with Bilibili, WeChat, Xiaohongshu localization|

Want to add a translation? Open an issue and we'll link it here.

---

### 🔗 Related Resources

- [awesome-openclaw-agents](https://github.com/mergisi/awesome-openclaw-agents) — Community-maintained OpenClaw agent collection (derived from this repo)

### 🚀 Get Started

1. **Browse** the agents above and find specialists for your needs
2. **Copy** the agents to `~/.claude/agents/` for Claude Code integration
3. **Activate** agents by referencing them in your Claude conversations
4. **Customize** agent personalities and workflows for your specific needs
5. **Share** your results and contribute back to the community

## 提示词流——Paperclip

**Paperclip** 是一个开源的 Node.js 服务器和 React UI，用于编排 AI 代理团队来运营企业。

**核心理念：**

如果 OpenClaw 是员工，Paperclip 就是公司

**它能做什么：**

- 定义公司目标（如"打造 #1 AI 笔记应用，月入 100 万美元"）
- 雇佣 AI 团队（CEO、CTO、工程师、设计师、营销人员）
- 设置预算和成本监控
- 从统一仪表盘追踪工作和成本
- ✅ 审批策略、监督执行、随时干预

**一句话：** 管理企业目标，而不是 Pull Requests！

### 技术架构：一个 TypeScript Monorepo 里的”公司”

Paperclip 是一个 **pnpm monorepo**，完全用 TypeScript 编写，核心结构如下：

```text
server/          → Express REST API + 编排服务（端口 3100）
ui/              → React + Vite 仪表盘（支持 PWA 安装）
packages/db/     → Drizzle ORM 数据模型与迁移
packages/shared/ → 共享类型、常量、校验器
cli/             → paperclipai 命令行工具
skills/          → 运行时技能注入文件
```

数据库层用 **PostgreSQL**，但开发体验做得很聪明：本地开发直接内嵌 **PGlite**（零外部依赖），生产环境接外部 Postgres。只需一行命令就能跑起来：

```text
npx paperclipai onboard --yes
```

前端以 **PWA**（Progressive Web App）形式发布，支持 Service Worker 离线缓存，可以装到手机上随时监控 Agent 状态。界面包括仪表盘、组织架构图、工单视图（支持线程式对话）、成本面板（按 Agent 显示预算进度条）、收件箱、审批队列、命令面板，以及带 Markdown 渲染的运行日志查看器。

测试框架用的是 **Vitest**（单元测试）和 **Playwright**（端到端测试）。CI/CD 基于 GitHub Actions，包含 PR 校验、策略检查和 lockfile 刷新流水线。

## Soul流——CrewClaw

[162 production-ready AI agent templates for OpenClaw. SOUL.md configs across 19 categories. Submit yours!](https://github.com/mergisi/awesome-openclaw-agents)

[crewclaw.com](https://crewclaw.com/ "https://crewclaw.com")

### Getting Started

- [What is OpenClaw?](https://crewclaw.com/blog/what-is-openclaw-ai-agent-framework) - Complete guide to the framework
- [Create Your First Agent](https://crewclaw.com/blog/how-to-create-ai-agent-openclaw) - No code required
- [OpenClaw Setup Guide 2026](https://crewclaw.com/blog/openclaw-setup-guide-2026) - Install, configure, run
- [SOUL.md Templates](https://crewclaw.com/blog/soul-md-examples-templates) - 10 ready-to-use examples

### Multi-Agent & Orchestration

- [Multi-Agent Setup Guide](https://crewclaw.com/blog/openclaw-multi-agent-setup-guide) - Run multiple agents together
- [Agent-to-Agent Communication](https://crewclaw.com/blog/openclaw-agent-to-agent-communication) - How agents collaborate
- [Build an AI Team](https://crewclaw.com/blog/build-ai-team-workflows) - Workflows that run autonomously

### Integrations & Automation

- [Slack & Telegram Integration](https://crewclaw.com/blog/openclaw-slack-telegram-integration) - Connect to messaging channels
- [Run with Ollama](https://crewclaw.com/blog/openclaw-ollama-local-agents) - Free local AI agents
- [Automation Guide](https://crewclaw.com/blog/openclaw-automation-guide) - Build 24/7 workflows
- [CLI Commands Reference](https://crewclaw.com/blog/openclaw-cli-commands-reference) - Complete cheat sheet

### Comparisons

- [OpenClaw vs LangChain](https://crewclaw.com/blog/openclaw-vs-langchain) - Framework comparison
- [OpenClaw vs AutoGPT](https://crewclaw.com/blog/openclaw-vs-autogpt) - Key differences
- [OpenClaw vs CrewAI](https://crewclaw.com/blog/openclaw-vs-crewai) - Which is better?

## 新架构式——三省六部翰林院

[AI 朝廷搭建完整教程 - 从零基础到进阶](https://github.com/wanikua/danghuangshang)

一行命令起王朝，三省六部皆AI。千里之外调百官，万事不劳御驾亲。

以明朝内阁制为蓝本，用 OpenClaw 框架构建的多 Agent 协作系统。 一台服务器 + OpenClaw = 一支 7×24 在线的 AI 朝廷。

AI 朝廷是一个开箱即用的多 Agent 协作系统。你是皇帝，AI 是你的大臣。在 Discord 或飞书 @某个 Agent，大臣们就会立刻执行。司礼监接旨、内阁优化 Prompt、六部各司其职、都察院自动审查代码。管理 AI 团队的高效模版。

### 🏛️ 朝廷架构

三省六部制是中国古代经典制度。 **明朝初年** 废丞相，司礼监 + 内阁二元治理。 本项目的Agent团队协作方式采用更贴近明朝六部的制度。

**方式一：经司礼监调度（默认）**

```
皇帝（你）下旨
  ▼
司礼监（大内总管）── 接旨
  │
  ├─→ 内阁（首辅）── Prompt 增强：理解意图、引导追问、生成执行计划
  │ ←─┘ 返回优化后的 Prompt + Plan
  │
  ├─→ @兵部 @户部 @礼部 …（按 Plan 派发）
  │
  └─→ 都察院 ── 代码 push 到 GitHub 时自动审查
       └─→ ✅ 通过 / ❌ 打回修改
```

**方式二：皇帝直接下旨给任意部门**（Discord 多 Bot 模式）

```
皇帝（你）
  ├─→ @兵部 写个登录 API        ← 直接指挥，跳过司礼监
  ├─→ @户部 查本月开销
  └─→ @都察院 审查这个 PR
```

> 💡 Discord 多 Bot 模式下，每个部门都是独立 Bot，你可以直接 @任意部门下达指令，无需经过司礼监。复杂任务推荐走司礼监（自动内阁优化），简单任务直接 @对应部门更快。

**查看完整机构表**

|机构|角色|在本项目中|
|---|---|---|
|**司礼监**|大内总管|接旨 → 送内阁优化 → 派发六部|
|**内阁**|首辅大学士|Prompt 增强、追问、生成计划、重大决策审议|
|**都察院**|左都御史|GitHub push 自动代码审查|
|**兵部**|尚书|软件工程：编码、架构、Bug 调试|
|**户部**|尚书|财务运营：成本分析、预算管控|
|**礼部**|尚书|品牌营销：文案、社媒运营|
|**工部**|尚书|运维：DevOps、CI/CD|
|**吏部**|尚书|项目管理、团队协调|
|**刑部**|尚书|法务合规、知识产权|

**与明朝制度的映射：**

|明朝|本项目|
|---|---|
|皇帝下旨 → 司礼监批红|用户 @司礼监 → 接旨调度|
|内阁票拟（起草方案）|内阁 Prompt 增强 + Plan|
|司礼监代批（下发执行）|用优化后 Prompt 派发六部|
|都察院纠劾百官|GitHub push 自动审查|
|内阁封驳权|内阁可否决不合理方案|











