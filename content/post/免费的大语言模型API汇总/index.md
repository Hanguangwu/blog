---
title: 免费的大语言模型API汇总
description: 本文介绍一些免费的大语言模型API。
date: 2026-01-03T15:34:25+08:00
draft: false
categories:
- AI
tags:
- GitHub
- LLM
---

# 免费的大语言模型API汇总

## 前言

[GitHub-Repo-地址](https://github.com/cheahjs/free-llm-api-resources)

OpenRouter网站可以只使用邮箱注册，这里只推荐OpenRouter网站。

下面列出模型名称，方便查阅。

## [OpenRouter](https://openrouter.ai/)

**Limits:**

[20 requests/minute  50 requests/day  Up to 1000 requests/day with $10 lifetime topup](https://openrouter.ai/docs/api-reference/limits)

Models share a common quota.

- [Gemma 3 12B Instruct](https://openrouter.ai/google/gemma-3-12b-it:free)
- [Gemma 3 27B Instruct](https://openrouter.ai/google/gemma-3-27b-it:free)
- [Gemma 3 4B Instruct](https://openrouter.ai/google/gemma-3-4b-it:free)
- [Hermes 3 Llama 3.1 405B](https://openrouter.ai/nousresearch/hermes-3-llama-3.1-405b:free)
- [Llama 3.1 405B Instruct](https://openrouter.ai/meta-llama/llama-3.1-405b-instruct:free)
- [Llama 3.2 3B Instruct](https://openrouter.ai/meta-llama/llama-3.2-3b-instruct:free)
- [Llama 3.3 70B Instruct](https://openrouter.ai/meta-llama/llama-3.3-70b-instruct:free)
- [Mistral 7B Instruct](https://openrouter.ai/mistralai/mistral-7b-instruct:free)
- [Mistral Small 3.1 24B Instruct](https://openrouter.ai/mistralai/mistral-small-3.1-24b-instruct:free)
- [Qwen 2.5 VL 7B Instruct](https://openrouter.ai/qwen/qwen-2.5-vl-7b-instruct:free)
- [alibaba/tongyi-deepresearch-30b-a3b:free](https://openrouter.ai/alibaba/tongyi-deepresearch-30b-a3b:free)
- [allenai/olmo-3-32b-think:free](https://openrouter.ai/allenai/olmo-3-32b-think:free)
- [allenai/olmo-3.1-32b-think:free](https://openrouter.ai/allenai/olmo-3.1-32b-think:free)
- [arcee-ai/trinity-mini:free](https://openrouter.ai/arcee-ai/trinity-mini:free)
- [cognitivecomputations/dolphin-mistral-24b-venice-edition:free](https://openrouter.ai/cognitivecomputations/dolphin-mistral-24b-venice-edition:free)
- [deepseek/deepseek-r1-0528:free](https://openrouter.ai/deepseek/deepseek-r1-0528:free)
- [google/gemma-3n-e2b-it:free](https://openrouter.ai/google/gemma-3n-e2b-it:free)
- [google/gemma-3n-e4b-it:free](https://openrouter.ai/google/gemma-3n-e4b-it:free)
- [kwaipilot/kat-coder-pro:free](https://openrouter.ai/kwaipilot/kat-coder-pro:free)
- [mistralai/devstral-2512:free](https://openrouter.ai/mistralai/devstral-2512:free)
- [moonshotai/kimi-k2:free](https://openrouter.ai/moonshotai/kimi-k2:free)
- [nex-agi/deepseek-v3.1-nex-n1:free](https://openrouter.ai/nex-agi/deepseek-v3.1-nex-n1:free)
- [nvidia/nemotron-3-nano-30b-a3b:free](https://openrouter.ai/nvidia/nemotron-3-nano-30b-a3b:free)
- [nvidia/nemotron-nano-12b-v2-vl:free](https://openrouter.ai/nvidia/nemotron-nano-12b-v2-vl:free)
- [nvidia/nemotron-nano-9b-v2:free](https://openrouter.ai/nvidia/nemotron-nano-9b-v2:free)
- [openai/gpt-oss-120b:free](https://openrouter.ai/openai/gpt-oss-120b:free)
- [openai/gpt-oss-20b:free](https://openrouter.ai/openai/gpt-oss-20b:free)
- [qwen/qwen3-4b:free](https://openrouter.ai/qwen/qwen3-4b:free)
- [qwen/qwen3-coder:free](https://openrouter.ai/qwen/qwen3-coder:free)
- [tngtech/deepseek-r1t-chimera:free](https://openrouter.ai/tngtech/deepseek-r1t-chimera:free)
- [tngtech/deepseek-r1t2-chimera:free](https://openrouter.ai/tngtech/deepseek-r1t2-chimera:free)
- [tngtech/tng-r1t-chimera:free](https://openrouter.ai/tngtech/tng-r1t-chimera:free)
- [xiaomi/mimo-v2-flash:free](https://openrouter.ai/xiaomi/mimo-v2-flash:free)
- [z-ai/glm-4.5-air:free](https://openrouter.ai/z-ai/glm-4.5-air:free)

# 附录

## [GitHub Models 讓你免費玩 GPT、Llama、Phi，還提供 API 串接](https://blog.jiatool.com/posts/github_models/)

### 前言

我前陣子才發現，原來 GitHub 上面也有可以免費使用的 LLM，像是 GPT-4o、Llama-3.3、Phi-3.5…等等，甚至還提供 API 可串接程式！

雖然 GitHub Models 主要是讓我們在開發生成式 AI 應用程式測試用，算是試用性質，所以有 速率 & Token 數量限制，但我覺得用作個人專案還蠻不錯的，每日請求上限也不算太少 (50~150 次，依模型而定)，有需求的網友可以試試~

![GitHub Models](https://res.cloudinary.com/jiablog/github_models/github_models.jpg)

GitHub Models

如果你發現還不能使用，可加入候補名單：[https://github.com/marketplace/models/waitlist](https://github.com/marketplace/models/waitlist)

### 速率限制

在開始使用之前，首先來看看它速率限制到底是多少。

- [Rate limits | GitHub Docs](https://docs.github.com/en/github-models/prototyping-with-ai-models#rate-limits)

![GitHub Models 使用速率限制](https://res.cloudinary.com/jiablog/github_models/rate_limits.jpg)

GitHub Models 使用速率限制

GitHub Models 是依照模型分成 Low、High、Embedding 等級去做限制，  
在 模型介紹 和 Playground 頁面都有寫此模型是採用哪個限制等級 ("Rate limit tier")。

* 表格底下雖然還有 Azure OpenAI o1-preview 和 Azure OpenAI o1-mini，但我好像沒辦法使用。

例如 GPT-4o 是 "High"，那它的限制就是：

- 每分鐘請求數：10 次
- 每天的請求數：50 次
- 每個請求的 Tokens：輸入 8000, 輸出 4000
- 並發請求：2 個

* 當然假如你是 Copilot Business 或 Copilot Enterprise，那可使用次數就會更多。

### 模型清單

GitHub Models 有提供哪些模型讓我們試用呢？

這邊有完整支援的模型清單：[https://github.com/marketplace?type=models](https://github.com/marketplace?type=models)

![GitHub Models 支援模型清單](https://res.cloudinary.com/jiablog/github_models/models_list.jpg)

GitHub Models 支援模型清單

例如聊天的模型有：

- OpenAI GPT-4o
- OpenAI GPT-4o mini
- DeepSeek-V3-0324
- DeepSeek-R1
- Llama 4 Maverick 17B 128E Instruct FP8
- Llama-3.3-70B-Instruct
- Llama-3.2-90B-Vision-Instruct
- Phi-4
- Phi-3.5-MoE instruct
- Phi-3.5-vision instruct
- Mistral Large
- Mistral Small 3.1
- Codestral 25.01
- Cohere Command R+
- AI21 Jamba 1.5
- JAIS 30b Chat
- …(更多)

還有 Embedding 嵌入模型：

- OpenAI Text Embedding 3
- Cohere Embed v3 Multilingual
- …(更多)

點選任一個模型後，會進入模型介紹頁面。

會有模型的相關說明介紹、測試評估分數、License 等等，右邊區塊還有像是 簡介、Context (模型"本身" 輸入、輸出 tokens 限制)、訓練資料日期、速率限制等級 (Rate limit tier)、提供者、支援語言 等等資訊。

![模型介紹頁面](https://res.cloudinary.com/jiablog/github_models/model_readme.jpg)

模型介紹頁面

### API

如同文章標題提到的，除了在網頁使用 Playground 介面，GitHub Models 還提供 API 供我們串接自己的程式做測試。

#### 創建 GitHub Token

在開始使用 API 之前，我們要先去 GitHub 建立 Token，用作身份驗證。

* 關於 GitHub 的 Token 介紹，可以參考這篇官方文件： [Managing your personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)


Settings > 左側最下方 Developer settings > Personal access tokens > [Fine-grained tokens](https://github.com/settings/personal-access-tokens)

點選 Generate new token。

Expiration 過期時間可以改成 "No expiration" (無期限)。

Repository access 欄位維持 "Public Repositories" 即可。

![Expiration、Repository access 欄位設定](https://res.cloudinary.com/jiablog/github_models/expiration_repository.jpg)

Expiration、Repository access 欄位設定

**重點**：  
Permissions > Account permissions > Models 要改為 "Read-only"，這樣才有 GitHub Models 的權限。

![Permissions > Account permissions > Models 權限](https://res.cloudinary.com/jiablog/github_models/models_readonly.jpg)

Permissions > Account permissions > Models 權限

填寫完欄位後，最下方點擊 Generate token 按鈕。

![創建新的 Token](https://res.cloudinary.com/jiablog/github_models/generate_token_0.jpg)

創建新的 Token

將 token 複製並保存好，之後忘記就只能再重新產生了。

Fine-grained personal access token 會長的類似這樣：`github_pat_11AHxxxxxxxxxxxxxxxxxxxxxxxxxCp7pLSr3a`

![將 Token 複製存起來，之後就沒辦法再看到了](https://res.cloudinary.com/jiablog/github_models/generate_token_2.jpg)

將 Token 複製存起來，之後就沒辦法再看到了

## GPT-API-free / DeepSeek-API-free

[GitHub-Repo](https://github.com/chatanywhere/GPT_API_free)

免费使用 gpt-5 | deepseek

支持 gpt | deepseek | claude | gemini | grok

国内动态加速 直连无需代理 协议统一接入便捷

[快速开始](https://github.com/chatanywhere/GPT_API_free#%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8) / [API文档](https://chatanywhere.apifox.cn/) / [申请内测免费Key](https://api.chatanywhere.tech/v1/oauth/free/render) / [支持付费Key](https://api.chatanywhere.tech/#/shop/) / [服务可用性](https://status.chatanywhere.tech/)

[QQ群: 1075658240](https://qm.qq.com/cgi-bin/qm/qr?k=azzJOyPvbruLGK1HjZK2I4B5T6x3T4Al&jump_from=webapi&authKey=drsWlRk+MrABfJfH3/1WuX9Gebch5pCrt8kOpifk0Fk2Ot3TvKnuos4MpfZG8Mjj)

[![](https://camo.githubusercontent.com/86a9ff8873f5483803b78e9863d0dda8a90fbba7895b56752d028e697a59792a/68747470733a2f2f7374617475732e63686174616e7977686572652e6f72672f6170692f62616467652f362f757074696d652f32343f6c6162656c5072656669783d4750543a)](https://status.chatanywhere.tech/) [![](https://camo.githubusercontent.com/65f4ae0de922c2c97b4ee9701bcd0dcdcb773791443dd260c34c399b189df733/68747470733a2f2f7374617475732e63686174616e7977686572652e6f72672f6170692f62616467652f31302f757074696d652f32343f6c6162656c5072656669783d4750542d43412545372542332542422545352538382539373a)](https://status.chatanywhere.tech/)

[![](https://camo.githubusercontent.com/2482c2fa5aad779b89b666e2bbb4bdf30af34239913e1aba78714c826c70a148/68747470733a2f2f7374617475732e63686174616e7977686572652e6f72672f6170692f62616467652f382f757074696d652f32343f6c6162656c5072656669783d436c617564653a)](https://status.chatanywhere.tech/) [![](https://camo.githubusercontent.com/6ea636a8657fd9b083b09466e89cf593ffa40f50287891aab21be19963574928/68747470733a2f2f7374617475732e63686174616e7977686572652e6f72672f6170692f62616467652f332f757074696d652f32343f6c6162656c5072656669783d47656d696e693a)](https://status.chatanywhere.tech/) [![](https://camo.githubusercontent.com/8702e9117e3f72078068726c3ae9a6b1c57f5a2280eee256fb7eada28089621f/68747470733a2f2f7374617475732e63686174616e7977686572652e6f72672f6170692f62616467652f342f757074696d652f32343f6c6162656c5072656669783d446565707365656b3a)](https://status.chatanywhere.tech/)

### 隐私声明

该项目高度重视隐私，致力于保护其用户的隐私。该项目不会以任何方式收集、记录或存储用户输入的任何文本或由 OpenAI 服务器返回的任何文本。该项目不会向 OpenAI 或任何第三方提供有关 API 调用者的身份的任何信息，包括但不限于 IP 地址和用户代理字符串。

但OpenAI官方会根据其[数据使用政策](https://platform.openai.com/docs/data-usage-policies)保留 30 天的数据。

### 特点

1. 支持 gpt | deepseek | claude | gemini | grok 等排名靠前的常用大模型。
2. 免费版支持gpt-5.2, gpt-5.1, gpt-5, gpt-4o，gpt-4.1一天5次；支持deepseek-r1, deepseek-v3, deepseek-v3-2-exp一天30次，支持gpt-4o-mini，gpt-3.5-turbo，gpt-4.1-mini，gpt-4.1-nano, gpt-5-mini，gpt-5-nano一天200次。
3. 与官方完全一致的接口标准，兼容各种软件/插件。
4. 支持流式响应。
5. 国内线路使用动态加速，体验远优于使用代理连接官方。
6. 无需科学上网，国内环境直接可用。
7. 个人完全免费使用。
8. 协议统一使用openai标准协议，其他厂商模型仅需更换模型名称，接入便捷

### 🚩注意事项

❗️_如果遇到无回复，报错等情况，可以查看 [status.chatanywhere.tech](https://status.chatanywhere.tech/)，确认服务状态是否正常，以帮助排查问题。_

❗️_免费API Key gpt-5系列模型的推理能力较弱，若需要更强的推理能力，可以购买付费API_

❗️**免费API Key仅可用于个人非商业用途，教育，非营利性科研工作中。免费API Key严禁商用，严禁大规模训练商用模型！训练科研用模型请提前加群联系我们。**

❗️我们将不定期对被滥用的Key进行封禁，如发现自己的key被误封请通过QQ群联系我们。

❗️我们的系统仅供内部评估测试使用，商用或面向大众使用请自行承担风险。

为了该项目长久发展，免费API Key限制**200请求/天/IP&Key**调用频率（gpt和embedding分开计算，各200次），也就是说你如果在一个IP下使用多个Key，所有Key的每天请求数总和不能超过200；同理，你如果将一个Key用于多个IP，这个Key的每天请求数也不能超过200。(**付费版API没有这个限制**)

### 免费使用

- **🚀[申请领取内测免费API Key](https://api.chatanywhere.tech/v1/oauth/free/render)**
- 免费版支持deepseek, gpt-3.5-turbo, embedding, gpt-4o系列, gpt-5系列。
- **转发Host1: `https://api.chatanywhere.tech` (国内中转，延时更低)**
- **转发Host2: `https://api.chatanywhere.org` (国外使用)**

我们会定期根据使用量进行相应的扩容，只要不被官方制裁我们会一直提供免费API，如果该项目对你有帮助，还请为我们点一个_**Star**_。如果遇到问题可以在[Issues](https://github.com/chatanywhere/GPT_API_free/issues)中反馈，有空会解答。

该API Key用于转发API，需要将Host改为`api.chatanywhere.tech`(国内首选)或者`api.chatanywhere.org`(国外使用)。


















