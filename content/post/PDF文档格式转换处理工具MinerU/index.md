---
title: PDF文档格式转换处理工具MinerU
description: 本文介绍PDF文档格式转换处理工具MinerU。
date: 2025-12-26T16:34:25+08:00
draft: false
categories:
- APP
tags:
- MinerU
---

# PDF文档格式转换处理工具MinerU

## 前言

<p align="center">
🚀<a href="https://mineru.net/?source=github">MinerU 官网入口→✅ 免装在线版 ✅ 全功能客户端 ✅ 开发者API在线调用，省去部署麻烦，多种产品形态一键get，速冲！</a>
</p>

[GitHub-Repo-MinerU](https://github.com/opendatalab/MinerU)

本项目值得学习一下，特别是文档格式处理功能的实现。未选择的路~

## 项目简介

MinerU是一款将PDF转化为机器可读格式的工具（如markdown、json），可以很方便地抽取为任意格式。

MinerU诞生于[书生-浦语](https://github.com/InternLM/InternLM)的预训练过程中，我们将会集中精力解决科技文献中的符号转化问题，希望在大模型时代为科技发展做出贡献。

## 主要功能

- 删除页眉、页脚、脚注、页码等元素，确保语义连贯
- 输出符合人类阅读顺序的文本，适用于单栏、多栏及复杂排版
- 保留原文档的结构，包括标题、段落、列表等
- 提取图像、图片描述、表格、表格标题及脚注
- 自动识别并转换文档中的公式为LaTeX格式
- 自动识别并转换文档中的表格为HTML格式
- 自动检测扫描版PDF和乱码PDF，并启用OCR功能
- OCR支持109种语言的检测与识别
- 支持多种输出格式，如多模态与NLP的Markdown、按阅读顺序排序的JSON、含有丰富信息的中间格式等
- 支持多种可视化结果，包括layout可视化、span可视化等，便于高效确认输出效果与质检
- 支持纯CPU环境运行，并支持 GPU(CUDA)/NPU(CANN)/MPS 加速
- 兼容Windows、Linux和Mac平台

## 快速开始

如果安装或使用中遇到任何问题，请先查询 <a href="##faq">FAQ</a> </br>
如果遇到解析效果不及预期，参考 <a href="##known-issues">Known Issues</a></br>

### 在线体验

#### 官网在线应用

官网在线版功能与客户端一致，界面美观，功能丰富，需要登录使用  

- [![OpenDataLab](https://img.shields.io/badge/webapp_on_mineru.net-blue?logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTM0IiBoZWlnaHQ9IjEzNCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cGF0aCBkPSJtMTIyLDljMCw1LTQsOS05LDlzLTktNC05LTksNC05LDktOSw5LDQsOSw5eiIgZmlsbD0idXJsKCNhKSIvPjxwYXRoIGQ9Im0xMjIsOWMwLDUtNCw5LTksOXMtOS00LTktOSw0LTksOS05LDksNCw5LDl6IiBmaWxsPSIjMDEwMTAxIi8+PHBhdGggZD0ibTkxLDE4YzAsNS00LDktOSw5cy05LTQtOS05LDQtOSw5LTksOSw0LDksOXoiIGZpbGw9InVybCgjYikiLz48cGF0aCBkPSJtOTEsMThjMCw1LTQsOS05LDlzLTktNC05LTksNC05LDktOSw5LDQsOSw5eiIgZmlsbD0iIzAxMDEwMSIvPjxwYXRoIGZpbGwtcnVsZT0iZXZlbm9kZCIgY2xpcC1ydWxlPSJldmVub2RkIiBkPSJtMzksNjJjMCwxNiw4LDMwLDIwLDM4LDctNiwxMi0xNiwxMi0yNlY0OWMwLTQsMy03LDYtOGw0Ni0xMmM1LTEsMTEsMywxMSw4djMxYzAsMzctMzAsNjYtNjYsNjYtMzcsMC02Ni0zMC02Ni02NlY0NmMwLTQsMy03LDYtOGwyMC02YzUtMSwxMSwzLDExLDh2MjF6bS0yOSw2YzAsMTYsNiwzMCwxNyw0MCwzLDEsNSwxLDgsMSw1LDAsMTAtMSwxNS0zQzM3LDk1LDI5LDc5LDI5LDYyVjQybC0xOSw1djIweiIgZmlsbD0idXJsKCNjKSIvPjxwYXRoIGZpbGwtcnVsZT0iZXZlbm9kZCIgY2xpcC1ydWxlPSJldmVub2RkIiBkPSJtMzksNjJjMCwxNiw4LDMwLDIwLDM4LDctNiwxMi0xNiwxMi0yNlY0OWMwLTQsMy03LDYtOGw0Ni0xMmM1LTEsMTEsMywxMSw4djMxYzAsMzctMzAsNjYtNjYsNjYtMzcsMC02Ni0zMC02Ni02NlY0NmMwLTQsMy03LDYtOGwyMC02YzUtMSwxMSwzLDExLDh2MjF6bS0yOSw2YzAsMTYsNiwzMCwxNyw0MCwzLDEsNSwxLDgsMSw1LDAsMTAtMSwxNS0zQzM3LDk1LDI5LDc5LDI5LDYyVjQybC0xOSw1djIweiIgZmlsbD0iIzAxMDEwMSIvPjxkZWZzPjxsaW5lYXJHcmFkaWVudCBpZD0iYSIgeDE9Ijg0IiB5MT0iNDEiIHgyPSI3NSIgeTI9IjEyMCIgZ3JhZGllbnRVbml0cz0idXNlclNwYWNlT25Vc2UiPjxzdG9wIHN0b3AtY29sb3I9IiNmZmYiLz48c3RvcCBvZmZzZXQ9IjEiIHN0b3AtY29sb3I9IiMyZTJlMmUiLz48L2xpbmVhckdyYWRpZW50PjxsaW5lYXJHcmFkaWVudCBpZD0iYiIgeDE9Ijg0IiB5MT0iNDEiIHgyPSI3NSIgeTI9IjEyMCIgZ3JhZGllbnRVbml0cz0idXNlclNwYWNlT25Vc2UiPjxzdG9wIHN0b3AtY29sb3I9IiNmZmYiLz48c3RvcCBvZmZzZXQ9IjEiIHN0b3AtY29sb3I9IiMyZTJlMmUiLz48L2xpbmVhckdyYWRpZW50PjxsaW5lYXJHcmFkaWVudCBpZD0iYyIgeDE9Ijg0IiB5MT0iNDEiIHgyPSI3NSIgeTI9IjEyMCIgZ3JhZGllbnRVbml0cz0idXNlclNwYWNlT25Vc2UiPjxzdG9wIHN0b3AtY29sb3I9IiNmZmYiLz48c3RvcCBvZmZzZXQ9IjEiIHN0b3AtY29sb3I9IiMyZTJlMmUiLz48L2xpbmVhckdyYWRpZW50PjwvZGVmcz48L3N2Zz4=&labelColor=white)](https://mineru.net/OpenSourceTools/Extractor?source=github)

#### 基于Gradio的在线demo

基于gradio开发的webui，界面简洁，仅包含核心解析功能，免登录

- [![ModelScope](https://img.shields.io/badge/Demo_on_ModelScope-purple?logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjIzIiBoZWlnaHQ9IjIwMCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KCiA8Zz4KICA8dGl0bGU+TGF5ZXIgMTwvdGl0bGU+CiAgPHBhdGggaWQ9InN2Z18xNCIgZmlsbD0iIzYyNGFmZiIgZD0ibTAsODkuODRsMjUuNjUsMGwwLDI1LjY0OTk5bC0yNS42NSwwbDAsLTI1LjY0OTk5eiIvPgogIDxwYXRoIGlkPSJzdmdfMTUiIGZpbGw9IiM2MjRhZmYiIGQ9Im05OS4xNCwxMTUuNDlsMjUuNjUsMGwwLDI1LjY1bC0yNS42NSwwbDAsLTI1LjY1eiIvPgogIDxwYXRoIGlkPSJzdmdfMTYiIGZpbGw9IiM2MjRhZmYiIGQ9Im0xNzYuMDksMTQxLjE0bC0yNS42NDk5OSwwbDAsMjIuMTlsNDcuODQsMGwwLC00Ny44NGwtMjIuMTksMGwwLDI1LjY1eiIvPgogIDxwYXRoIGlkPSJzdmdfMTciIGZpbGw9IiMzNmNmZDEiIGQ9Im0xMjQuNzksODkuODRsMjUuNjUsMGwwLDI1LjY0OTk5bC0yNS42NSwwbDAsLTI1LjY0OTk5eiIvPgogIDxwYXRoIGlkPSJzdmdfMTgiIGZpbGw9IiMzNmNmZDEiIGQ9Im0wLDY0LjE5bDI1LjY1LDBsMCwyNS42NWwtMjUuNjUsMGwwLC0yNS42NXoiLz4KICA8cGF0aCBpZD0ic3ZnXzE5IiBmaWxsPSIjNjI0YWZmIiBkPSJtMTk4LjI4LDg5Ljg0bDI1LjY0OTk5LDBsMCwyNS42NDk5OWwtMjUuNjQ5OTksMGwwLC0yNS42NDk5OXoiLz4KICA8cGF0aCBpZD0ic3ZnXzIwIiBmaWxsPSIjMzZjZmQxIiBkPSJtMTk4LjI4LDY0LjE5bDI1LjY0OTk5LDBsMCwyNS42NWwtMjUuNjQ5OTksMGwwLC0yNS42NXoiLz4KICA8cGF0aCBpZD0ic3ZnXzIxIiBmaWxsPSIjNjI0YWZmIiBkPSJtMTUwLjQ0LDQybDAsMjIuMTlsMjUuNjQ5OTksMGwwLDI1LjY1bDIyLjE5LDBsMCwtNDcuODRsLTQ3Ljg0LDB6Ii8+CiAgPHBhdGggaWQ9InN2Z18yMiIgZmlsbD0iIzM2Y2ZkMSIgZD0ibTczLjQ5LDg5Ljg0bDI1LjY1LDBsMCwyNS42NDk5OWwtMjUuNjUsMGwwLC0yNS42NDk5OXoiLz4KICA8cGF0aCBpZD0ic3ZnXzIzIiBmaWxsPSIjNjI0YWZmIiBkPSJtNDcuODQsNjQuMTlsMjUuNjUsMGwwLC0yMi4xOWwtNDcuODQsMGwwLDQ3Ljg0bDIyLjE5LDBsMCwtMjUuNjV6Ii8+CiAgPHBhdGggaWQ9InN2Z18yNCIgZmlsbD0iIzYyNGFmZiIgZD0ibTQ3Ljg0LDExNS40OWwtMjIuMTksMGwwLDQ3Ljg0bDQ3Ljg0LDBsMCwtMjIuMTlsLTI1LjY1LDBsMCwtMjUuNjV6Ii8+CiA8L2c+Cjwvc3ZnPg==&labelColor=white)](https://www.modelscope.cn/studios/OpenDataLab/MinerU)
- [![HuggingFace](https://img.shields.io/badge/Demo_on_HuggingFace-yellow.svg?logo=data:image/png;base64,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&labelColor=white)](https://huggingface.co/spaces/opendatalab/MinerU)

### 本地部署

> [!WARNING]
> **安装前必看——软硬件环境支持说明**
>
> 为了确保项目的稳定性和可靠性，我们在开发过程中仅对特定的软硬件环境进行优化和测试。这样当用户在推荐的系统配置上部署和运行项目时，能够获得最佳的性能表现和最少的兼容性问题。
>
> 通过集中资源和精力于主线环境，我们团队能够更高效地解决潜在的BUG，及时开发新功能。
>
> 在非主线环境中，由于硬件、软件配置的多样性，以及第三方依赖项的兼容性问题，我们无法100%保证项目的完全可用性。因此，对于希望在非推荐环境中使用本项目的用户，我们建议先仔细阅读文档以及FAQ，大多数问题已经在FAQ中有对应的解决方案，除此之外我们鼓励社区反馈问题，以便我们能够逐步扩大支持范围。


<table>
    <thead>
        <tr>
            <th rowspan="2">解析后端</th>
            <th rowspan="2">pipeline <br> (精度<sup>1</sup> 82+)</th>
            <th colspan="5">vlm (精度<sup>1</sup> 90+)</th>
        </tr>
        <tr>
            <th>transformers</th>
            <th>mlx-engine</th>
            <th>vllm-engine / <br>vllm-async-engine</th>
            <th>lmdeploy-engine</th>
            <th>http-client</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th>后端特性</th>
            <td>速度快, 无幻觉</td>
            <td>兼容性好, 速度较慢</td>
            <td>比transformers快</td>
            <td>速度快, 兼容vllm生态</td>
            <td>速度快, 兼容lmdeploy生态</td>
            <td>适用于OpenAI兼容服务器<sup>6</sup></td>
        </tr>
        <tr>
            <th>操作系统</th>
            <td colspan="2" style="text-align:center;">Linux<sup>2</sup> / Windows / macOS</td>
            <td style="text-align:center;">macOS<sup>3</sup></td>
            <td style="text-align:center;">Linux<sup>2</sup> / Windows<sup>4</sup> </td>
            <td style="text-align:center;">Linux<sup>2</sup> / Windows<sup>5</sup> </td>
            <td>不限</td>
        </tr>
        <tr>
            <th>CPU推理支持</th>
            <td colspan="2" style="text-align:center;">✅</td>
            <td colspan="3" style="text-align:center;">❌</td>
            <td >不需要</td>
        </tr>
        <tr>
            <th>GPU要求</th><td colspan="2" style="text-align:center;">Volta及以后架构, 6G显存以上或Apple Silicon</td>
            <td>Apple Silicon</td>
            <td colspan="2" style="text-align:center;">Volta及以后架构, 8G显存以上</td>
            <td>不需要</td>
        </tr>
        <tr>
            <th>内存要求</th>
            <td colspan="5" style="text-align:center;">最低16GB以上, 推荐32GB以上</td>
            <td>8GB</td>
        </tr>
        <tr>
            <th>磁盘空间要求</th>
            <td colspan="5" style="text-align:center;">20GB以上, 推荐使用SSD</td>
            <td>2GB</td>
        </tr>
        <tr>
            <th>python版本</th>
            <td colspan="6" style="text-align:center;">3.10-3.13<sup>7</sup></td>
        </tr>
    </tbody>
</table> 


<sup>1</sup> 精度指标为OmniDocBench (v1.5)的End-to-End Evaluation Overall分数，基于`MinerU`最新版本测试  
<sup>2</sup> Linux仅支持2019年及以后发行版  
<sup>3</sup> MLX需macOS 13.5及以上版本支持，推荐14.0以上版本使用  
<sup>4</sup> Windows vLLM通过WSL2(适用于 Linux 的 Windows 子系统)实现支持  
<sup>5</sup> Windows LMDeploy只能使用`turbomind`后端，速度比`pytorch`后端稍慢，如对速度有要求建议通过WSL2运行  
<sup>6</sup> 兼容OpenAI API的服务器，如通过`vLLM`/`SGLang`/`LMDeploy`等推理框架部署的本地模型服务器或远程模型服务  
<sup>7</sup> Windows + LMDeploy 由于关键依赖`ray`未能在windows平台支持Python 3.13，故仅支持至3.10~3.12版本

> [!TIP]
> 除以上主流环境与平台外，我们也收录了一些社区用户反馈的其他平台支持情况，详情请参考[其他加速卡适配](https://opendatalab.github.io/MinerU/zh/usage/)。  
> 如果您有意将自己的环境适配经验分享给社区，欢迎通过[show-and-tell](https://github.com/opendatalab/MinerU/discussions/categories/show-and-tell)提交或提交PR至[其他加速卡适配](https://github.com/opendatalab/MinerU/tree/master/docs/zh/usage/acceleration_cards)文档。

### 安装 MinerU

#### 使用pip或uv安装MinerU

```bash
pip install --upgrade pip -i https://mirrors.aliyun.com/pypi/simple
pip install uv -i https://mirrors.aliyun.com/pypi/simple
uv pip install -U "mineru[core]" -i https://mirrors.aliyun.com/pypi/simple 
```

#### 通过源码安装MinerU

```bash
git clone https://github.com/opendatalab/MinerU.git
cd MinerU
uv pip install -e .[core] -i https://mirrors.aliyun.com/pypi/simple
```

> [!TIP]
> `mineru[core]`包含除`vLLM`/`LMDeploy`加速外的所有核心功能，兼容Windows / Linux / macOS系统，适合绝大多数用户。
> 如果您需要使用`vLLM`/`LMDeploy`加速VLM模型推理，或是有在边缘设备安装轻量版client端等需求，可以参考文档[扩展模块安装指南](https://opendatalab.github.io/MinerU/zh/quick_start/extension_modules/)。

---

#### 使用docker部署Mineru

MinerU提供了便捷的docker部署方式，这有助于快速搭建环境并解决一些棘手的环境兼容问题。
您可以在文档中获取[Docker部署说明](https://opendatalab.github.io/MinerU/zh/quick_start/docker_deployment/)。

---

### 使用 MinerU

最简单的命令行调用方式:

```bash
mineru -p <input_path> -o <output_path>
```

您可以通过命令行、API、WebUI等多种方式使用MinerU进行PDF解析，具体使用方法请参考[使用指南](https://opendatalab.github.io/MinerU/zh/usage/)。

## TODO

- [x] 基于模型的阅读顺序  
- [x] 正文中目录、列表识别  
- [x] 表格识别
- [x] 标题分级
- [x] 手写文本识别
- [x] 竖排文本识别
- [x] 拉丁字母重音符号识别
- [x] 正文中代码块识别
- [x] [化学式识别](docs/chemical_knowledge_introduction/introduction.pdf)(https://mineru.net)
- [ ] 图表内容识别

## Known Issues

- 阅读顺序基于模型对可阅读内容在空间中的分布进行排序，在极端复杂的排版下可能会部分区域乱序
- 对竖排文字的支持较为有限
- 目录和列表通过规则进行识别，少部分不常见的列表形式可能无法识别
- 代码块在layout模型里还没有支持
- 漫画书、艺术图册、小学教材、习题尚不能很好解析
- 表格识别在复杂表格上可能会出现行/列识别错误
- 在小语种PDF上，OCR识别可能会出现字符不准确的情况（如阿拉伯文易混淆字符等）
- 部分公式可能会无法在markdown中渲染

## Acknowledgments

- [PDF-Extract-Kit](https://github.com/opendatalab/PDF-Extract-Kit)
- [DocLayout-YOLO](https://github.com/opendatalab/DocLayout-YOLO)
- [UniMERNet](https://github.com/opendatalab/UniMERNet)
- [RapidTable](https://github.com/RapidAI/RapidTable)
- [TableStructureRec](https://github.com/RapidAI/TableStructureRec)
- [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR)
- [PaddleOCR2Pytorch](https://github.com/frotms/PaddleOCR2Pytorch)
- [layoutreader](https://github.com/ppaanngggg/layoutreader)
- [xy-cut](https://github.com/Sanster/xy-cut)
- [fast-langdetect](https://github.com/LlmKira/fast-langdetect)
- [pypdfium2](https://github.com/pypdfium2-team/pypdfium2)
- [pdftext](https://github.com/datalab-to/pdftext)
- [pdfminer.six](https://github.com/pdfminer/pdfminer.six)
- [pypdf](https://github.com/py-pdf/pypdf)
- [magika](https://github.com/google/magika)
- [vLLM](https://github.com/vllm-project/vllm)
- [LMDeploy](https://github.com/InternLM/lmdeploy)


## Links

- [Easy Data Preparation with latest LLMs-based Operators and Pipelines](https://github.com/OpenDCAI/DataFlow)
- [Vis3 (OSS browser based on s3)](https://github.com/opendatalab/Vis3)
- [LabelU (A Lightweight Multi-modal Data Annotation Tool)](https://github.com/opendatalab/labelU)
- [LabelLLM (An Open-source LLM Dialogue Annotation Platform)](https://github.com/opendatalab/LabelLLM)
- [PDF-Extract-Kit (A Comprehensive Toolkit for High-Quality PDF Content Extraction)](https://github.com/opendatalab/PDF-Extract-Kit)
- [OmniDocBench (A Comprehensive Benchmark for Document Parsing and Evaluation)](https://github.com/opendatalab/OmniDocBench)
- [Magic-HTML (Mixed web page extraction tool)](https://github.com/opendatalab/magic-html)
- [Magic-Doc (Fast speed ppt/pptx/doc/docx/pdf extraction tool)](https://github.com/InternLM/magic-doc) 
- [Dingo: A Comprehensive AI Data Quality Evaluation Tool](https://github.com/MigoXLab/dingo)











