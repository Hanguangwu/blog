---
title: 辅助学习的AI工具
description: 本文介绍辅助学习的AI工具——open notebook和deeptutor。
date: 2026-05-23T12:34:25-08:00
draft: false
categories:
- APP
tags:
- GitHub
- DeepTutor
- Open Notebook
---


# 辅助学习的AI工具

## 前言

这里分享一些辅助学习的AI工具。

## Open NotebookLM——一个私密、多模型、100%本地化、功能齐全的 Notebook LM 替代方案

项目地址：[https://github.com/lfnovo/open-notebook](https://github.com/lfnovo/open-notebook)

在人工智能主导的世界里，拥有思考 🧠 和获取新知识 💡 的能力，不应是少数人的特权，也不应局限于单一供应商。

**Open Notebook 使您能够：**

- 🔒 **掌控你的数据** - 保持研究内容的私密与安全
- 🤖 **选择你的AI模型** - 支持包括 OpenAI、Anthropic、Ollama、LM Studio 等在内的 18+ 家提供商
- 📚 **组织多模态内容** - PDF、视频、音频、网页等
- 🎙️ **生成专业播客** - 先进的多说话人播客生成
- 🔍 **智能搜索** - 对所有内容进行全文和向量搜索
- 💬 **基于上下文聊天** - 由你的研究驱动的AI对话
- 🌐 **多语言用户界面** - 支持英语、葡萄牙语、中文（简体与繁体）、日语、俄语和孟加拉语

了解更多项目信息请访问 [https://www.open-notebook.ai](https://www.open-notebook.ai/)

---

### 🆚 Open Notebook 与 Google Notebook LM 对比

|功能特性|Open Notebook|Google Notebook LM|优势|
|---|---|---|---|
|**隐私与控制**|自托管，您的数据|仅限 Google 云|完全的数据主权|
|**AI 提供商选择**|18+ 家提供商 (OpenAI, Anthropic, Ollama, LM Studio 等)|仅限 Google 模型|灵活性与成本优化|
|**播客发言人**|1-4 位发言人，支持自定义配置|仅限 2 位发言人|极高的灵活性|
|**内容转换**|自定义与内置转换|选项有限|无限的处理能力|
|**API 访问**|完整的 REST API|无 API|完全的自动化|
|**部署方式**|Docker、云端或本地|仅限 Google 托管|随处部署|
|**引用功能**|基础引用（将改进）|附带来源的全面引用|研究完整性|
|**自定义能力**|开源，完全可定制|封闭系统|无限的可扩展性|
|**成本**|仅需支付 AI 使用费|免费层级 + 月度订阅|透明且可控|

**为何选择 Open Notebook？**

- 🔒 **隐私优先**：您的敏感研究数据完全保持私密
- 💰 **成本控制**：可选择更经济的AI服务提供商，或通过Ollama本地运行
- 🎙️ **更优质的播客**：完整脚本控制与多说话人灵活性，相比有限的双人深度对话格式
- 🔧 **无限定制**：按需修改、扩展和集成
- 🌐 **无供应商锁定**：自由切换服务提供商，随处部署，完全拥有您的数据

### 技术栈

[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/) [![Next.js](https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=next.js&logoColor=white)](https://nextjs.org/) [![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://reactjs.org/) [![SurrealDB](https://img.shields.io/badge/SurrealDB-FF5E00?style=for-the-badge&logo=databricks&logoColor=white)](https://surrealdb.com/) [![LangChain](https://img.shields.io/badge/LangChain-3A3A3A?style=for-the-badge&logo=chainlink&logoColor=white)](https://www.langchain.com/)

### 🚀 快速开始 (2 分钟)

#### 环境要求

- 已安装 [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- 就这些！(API 密钥稍后在 UI 中配置)

#### 步骤 1：获取 docker-compose.yml 文件

**选项 A:** 直接下载

```bash
curl -o docker-compose.yml https://raw.githubusercontent.com/lfnovo/open-notebook/main/docker-compose.yml
```

**选项 B:** 手动创建文件 将以下内容复制到一个名为 `docker-compose.yml` 的新文件中：

```yaml
services:
  surrealdb:
    image: surrealdb/surrealdb:v2
    command: start --log info --user root --pass root rocksdb:/mydata/mydatabase.db
    user: root
    ports:
      - "8000:8000"
    volumes:
      - ./surreal_data:/mydata
    restart: always

  open_notebook:
    image: lfnovo/open_notebook:v1-latest
    ports:
      - "8502:8502"
      - "5055:5055"
    environment:
      - OPEN_NOTEBOOK_ENCRYPTION_KEY=change-me-to-a-secret-string
      - SURREAL_URL=ws://surrealdb:8000/rpc
      - SURREAL_USER=root
      - SURREAL_PASSWORD=root
      - SURREAL_NAMESPACE=open_notebook
      - SURREAL_DATABASE=open_notebook
    volumes:
      - ./notebook_data:/app/data
    depends_on:
      - surrealdb
    restart: always
```

#### 步骤 2：设置您的加密密钥

编辑 `docker-compose.yml` 并更改这一行：

```yaml
- OPEN_NOTEBOOK_ENCRYPTION_KEY=change-me-to-a-secret-string
```

为任意密钥值 (例如：`my-super-secret-key-123`)

#### 步骤 3：启动服务

```bash
docker-compose up -d
```

等待 15-20 秒，然后打开：**[http://localhost:8502](http://localhost:8502/)**

#### 步骤 4：配置 AI 提供商

1. 前往 **设置** → **API 密钥**
2. 点击 **添加凭证**
3. 选择您的提供商 (OpenAI, Anthropic, Google 等)
4. 粘贴您的 API 密钥并点击 **保存**
5. 点击 **测试连接** → **发现模型** → **注册模型**

完成！您可以开始创建您的第一个笔记本了。

> **需要 API 密钥？** 从以下获取： [OpenAI](https://platform.openai.com/api-keys) · [Anthropic](https://console.anthropic.com/) · [Google](https://aistudio.google.com/) · [Groq](https://console.groq.com/) (免费套餐)

> **想要免费的本地 AI？** 查看 [examples/docker-compose-ollama.yml](https://github.com/lfnovo/open-notebook/blob/main/examples/) 了解 Ollama 设置

---

### ✨ 核心功能

#### 核心能力

- **🔒 隐私优先**: 您的数据始终由您掌控 - 无云端依赖
- **🎯 多笔记本组织**: 无缝管理多个研究项目
- **📚 通用内容支持**: PDF、视频、音频、网页、Office 文档等
- **🤖 多模型 AI 支持**: 支持 18+ 家提供商，包括 OpenAI、Anthropic、Ollama、Google、LM Studio 等
- **🎙️ 专业播客生成**: 支持多说话者的高级播客，带有剧集配置文件
- **🔍 智能搜索**: 对所有内容进行全文和向量搜索
- **💬 上下文感知聊天**: 由您的研究资料驱动的 AI 对话
- **📝 AI 辅助笔记**: 生成见解或手动撰写笔记

#### 高级功能

- **⚡ 推理模型支持**：全面支持DeepSeek-R1和Qwen3等思维模型
- **🔧 内容转换**：强大的可定制操作，用于总结和提取见解
- **🌐 完整REST API**：提供完整的编程接口，支持自定义集成[![API文档](https://img.shields.io/badge/API-%E6%96%87%E6%A1%A3-blue?style=flat-square)](http://localhost:5055/docs)
- **🔐 可选密码保护**：通过身份验证确保公共部署的安全性
- **📊 细粒度上下文控制**：精确选择与AI模型共享的内容
- **📎 引用功能**：获取带有准确来源引用的答案

## 专属 AI 导师——DeepTutor

[将任何文档转化为多智能体驱动的互动学习体验，打造你的专属 AI 导师。](https://github.com/HKUDS/DeepTutor)

DeepTutor -- Agent-native, Open-sourced Personalized Tutoring. [https://deeptutor.info/](https://deeptutor.info/).

DeepTutor 是由香港大学数据智能实验室（HKUDS）开发的开源 Agent-Native 个性化教学平台。它不仅仅是一个文档问答工具，而是通过多智能体协作架构，将教材、论文和手册转化为深度互动的学习环境。DeepTutor 能够构建基于知识图谱的学习路径，提供带有精准引用的逐步解题指导，并能根据用户的学习进度自动演化出个性化的记忆与档案，真正实现从“被动搜索”到“主动引导”的学习范式转移。 该平台集成了深度研究、题目生成、数学动画生成以及交互式可视化等多种前沿能力。无论是科研工作者进行文献综述，还是学生进行针对性的考前练习，DeepTutor 都能通过其独特的“TutorBot”系统提供 24/7 的陪伴式教学。其底层支持 RAG（检索增强生成）与轻量化 Agent 引擎，确保了回答的严谨性与逻辑性，是当前开源社区中极具竞争力的 AI 教育解决方案。
### 核心功能

多智能体协同解题：采用双循环推理架构，模拟人类导师的思考过程，提供带有文档来源引用的深度逻辑解答，而非简单的文字堆砌。

智能互动“活手册”：利用 Book Engine 将静态资料编译为包含 14 种组件的互动电子书，支持即时测验、知识图谱映射和交互式演示。

持久化个性化记忆：系统会自动记录用户的学习轨迹、偏好与薄弱环节，随着使用时间的增长，AI 导师会越来越懂你的学习节奏。

多模态与数学可视化：原生支持 LaTeX 数学公式、代码执行以及基于 Manim 的数学动画，将抽象概念具象化为易于理解的动态图表。

### ✨ 核心亮点

**工作面**

- Chat —— Chat、Solve、Quiz、Research、Visualize 共享同一会话、知识库与引用历史,可在对话中逐级升级而不丢失上下文。
- Co-Writer —— 分栏 Markdown 工作区,任意选区都可改写、扩写或缩写,可选择由 KB 或网络支撑。草稿可一键存入笔记本。
- Book Engine —— 多智能体流水线把你的材料编译成交互式「活书」,包含 13 种块类型:测验、闪卡、时间线、概念图、内嵌的 GeoGebra 浏览器、动画等。页面对 KB 做指纹标记,任何漂移都可检测。

**你的资料库**

- Knowledge Bases —— 版本化、可用于 RAG 的文档集合,端到端基于 LlamaIndex。每次(重新)索引都被追踪、可比较、可回滚。
- Space —— 个人复习库,聚合聊天历史、笔记本、题库与用户自建的 Skills(`SKILL.md`),后者可切换 DeepTutor 的人格。
- 三层 Memory —— 仅追加的 L1 轨迹、L2 按工作面整理且带引用的事实、L3 跨工作面的综合。可审查的工作台与 Memory Graph 让你审视 DeepTutor「为什么知道」这些信息。

**可扩展性与可控性**

- 可组合的工具 —— RAG、网络搜索、代码执行、推理、头脑风暴、论文搜索、GeoGebra 分析,以及聊天助手(`ask_user`、`web_fetch`、`write_note`、`list_notebook`、`github_query`)。MCP 服务器与内置工具并列接入。
- 个人 TutorBot —— 持续、自主的导师,各自拥有工作区、Soul、Skills 与频道(Telegram、Discord、Slack、Matrix、Zulip 等)。基于 [nanobot](https://github.com/HKUDS/nanobot) 构建。
- 统一 Settings —— 唯一的草稿 / Apply 工作台,统管外观、模型、嵌入、搜索、能力、记忆、MCP 服务器与工具,并共享按调用计费的成本跟踪。
- 智能体原生 CLI —— 每个能力、KB、会话与 TutorBot 一条命令搞定;人类看富文本,智能体看结构化 JSON。把根目录的 [`SKILL.md`](https://github.com/HKUDS/DeepTutor/blob/main/SKILL.md) 交给任意可调用工具的 LLM,它就能自己驾驭 DeepTutor。
- 可选身份认证 —— 默认关闭;启用后即可部署多用户场景,具备 bcrypt + JWT、管理员面板,以及可选的 PocketBase / OAuth 侧车。
















