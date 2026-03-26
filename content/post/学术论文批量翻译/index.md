---
title: 学术论文批量翻译
description: 本文主要介绍 pdf2zh_next。
date: 2026-03-26T12:40:25-08:00
draft: false
categories:
- AI
- NLP
tags:
- pdf2zh_next
---

# 学术论文批量翻译

## 使用 pdf2zh_next 翻译学术论文

**告别 API 报错！Python 批量翻译 PDF 论文：遍历文件夹 + 调用 pdf2zh_next 终端命令（100% 可用）**

在使用 **PDFMathTranslate-next (pdf2zh_next)** 翻译学术论文时，很多朋友都会遇到 Python API 调用的各种坑：导入错误、配置校验失败、类缺失、版本冲突…… 折腾半天根本跑不起来。

其实**最简单、最稳定的方案**：用 Python 遍历 PDF 文件夹，自动调用终端命令`pdf2zh_next`逐份翻译，完全避开复杂的 API 适配，手动能跑的命令，代码就一定能跑！

本文就给大家带来**零报错、开箱即用**的批量 PDF 翻译脚本，适配 ollama 本地模型，保留原文格式，批量处理论文超方便。

[PDF scientific paper translation with preserved formats - 基于 AI 完整保留排版的 PDF 文档全文双语翻译，支持 Google/DeepL/Ollama/OpenAI 等服务，提供 CLI/GUI/Docker](https://github.com/PDFMathTranslate-next/PDFMathTranslate-next)

[pdf2zh-next.com](https://pdf2zh-next.com/)

### 一、适用场景

1. 不想适配复杂的 Python API，只想快速批量翻译 PDF
2. 旧版 / 新版`pdf2zh_next`API 报错、导入失败、配置校验不通过
3. 需要**遍历文件夹**自动翻译所有论文 PDF
4. 使用 ollama 本地模型翻译，保留排版格式
5. 默认使用 SiliconFlow 的翻译 API

### 二、环境准备

#### 1. 安装依赖

确保你已经安装好`pdf2zh_next`：

```
pip install pdf2zh-next
```

#### 2. 验证终端命令可用

先手动测试命令是否能正常运行（这一步很重要）：

```
pdf2zh_next .\papers_compression\测试论文.pdf --output .\paper_translated\
```

只要手动能翻译成功，本文代码就 100% 可用。

开启 GUI 格式：

```
pdf2zh_next --gui
```

### 三、完整批量翻译代码

这个脚本实现：**遍历指定文件夹所有 PDF → 自动逐份翻译 → 实时输出日志 → 异常不中断任务**。

```python
import os
import subprocess
from pathlib import Path

# ===================== 配置区域 =====================
PDF_FOLDER = r".\papers_compression"       # 你的PDF文件夹
OUTPUT_FOLDER = r".\paper_translated"      # 输出文件夹
LANG_IN = "en"
LANG_OUT = "zh"
# SERVICE = "ollama:translategemma"
# ====================================================

# 自动创建输出目录
os.makedirs(OUTPUT_FOLDER, exist_ok=True)

# 遍历所有PDF文件
pdf_dir = Path(PDF_FOLDER)
if not pdf_dir.exists():
    print(f"错误：文件夹不存在 {PDF_FOLDER}")
    exit()

pdf_files = list(pdf_dir.glob("*.pdf"))
if not pdf_files:
    print("未找到任何PDF文件")
    exit()

print(f"找到 {len(pdf_files)} 个PDF，开始批量翻译...\n")

# 逐个翻译
for i, pdf_path in enumerate(pdf_files, 1):
    pdf_str = str(pdf_path)
    print(f"[{i}/{len(pdf_files)}] 正在翻译: {pdf_path.name}")

    # 终端命令（完全和你手动输入的一致）
    cmd = [
        "pdf2zh_next",
        pdf_str,
        "--output", OUTPUT_FOLDER,
        "--lang-in", LANG_IN,
        "--lang-out", LANG_OUT,
        # "--service", SERVICE,
        # "--thread", "4"
    ]

    try:
        # 执行子进程（会实时打印翻译日志）
        subprocess.run(cmd, check=True)
        print(f"✅ 翻译完成: {pdf_path.name}\n")
    except subprocess.CalledProcessError as e:
        print(f"❌ 翻译失败: {pdf_path.name}\n")
    except KeyboardInterrupt:
        print("\n⏹️  用户终止任务")
        break

print("🎉 批量翻译任务全部结束！")
```

### 四、代码使用说明

#### 1. 修改配置

只需要修改脚本开头的**配置区域**，适配你的文件路径：

```
PDF_FOLDER = r".\papers_compression"   # 你的论文文件夹
OUTPUT_FOLDER = r".\paper_translated"   # 翻译输出文件夹
SERVICE = "ollama:translategemma"       # 你的ollama模型名
```

#### 2. 运行脚本

直接执行 Python 文件即可自动批量翻译：

```
python pdf_translator.py
```

#### 3. 功能特性

- ✅ 自动遍历文件夹，识别所有`.pdf`文件
- ✅ 逐份翻译，单个文件失败不影响整体任务
- ✅ 实时打印翻译日志，和终端操作完全一致
- ✅ 支持`Ctrl+C`手动终止任务
- ✅ 自动创建输出文件夹，无需手动新建
- ✅ 保留 PDF 原版格式，完美适配学术论文

### 五、为什么这个方案最稳定？

1. **不依赖任何 Python API**

   完全不调用`pdf2zh_next`的库代码，彻底解决：

   - 导入错误`ModuleNotFoundError`
   - 配置校验错误`ValidationError`
   - 版本冲突、类缺失等问题

2. **和手动执行命令完全一致**

   你在终端能跑通的命令，Python 子进程就一定能执行，无兼容性问题。

3. **极简稳定**

   代码逻辑简单，没有复杂的异步、配置初始化，新手也能轻松使用。

相比于折腾复杂的 Python API，**Python 遍历 + 终端命令**是`pdf2zh_next`批量翻译最稳妥的方案。

这个脚本零报错、易修改、可直接投入使用，非常适合需要批量翻译英文论文的朋友，彻底告别 API 适配烦恼，专注于论文阅读！

## 其他工具

### pdf2zh 工具

可能会报错：'PDFPageInterpreterEx' object has no attribute 'ncs'

当遇到 *AttributeError: 'PDFPageInterpreterEx' object has no attribute 'ncs'* 错误时，通常是由于 *pdfminer.six* 库的版本更新或自定义类未正确继承所需属性导致的。

此问题可能是由于 *pdfminer.six* 的版本不兼容引起的。可以通过安装特定版本来解决，例如：

```
pip install pdfminer-six==20250416
```

此版本与许多项目的自定义实现兼容，避免了 *ncs* 属性缺失的问题。

相关 Python 代码：

```python
import os
from pdf2zh import translate
from pdf2zh.doclayout import OnnxModel

# 加载模型（只加载一次，提升效率）
model = OnnxModel.load_available()

# ====================== 配置参数 ======================
# 待翻译的 PDF 文件夹路径（请修改为你自己的文件夹路径）
PDF_FOLDER = "papers_compression"
# 翻译输出文件夹（自动创建，无需手动建）
OUTPUT_FOLDER = "translated_output"

# 翻译基础参数（和原代码一致）
params = {
    "model": model,
    "output": OUTPUT_FOLDER,
    "lang_in": "en",
    "lang_out": "zh",
    "service": "ollama:translategemma",
    "thread": 4,
}
# ======================================================

# 自动创建输出文件夹
os.makedirs(OUTPUT_FOLDER, exist_ok=True)

# 遍历文件夹，获取所有 .pdf 文件（不区分大小写）
pdf_files = []
for filename in os.listdir(PDF_FOLDER):
    if filename.lower().endswith(".pdf"):
        pdf_path = os.path.join(PDF_FOLDER, filename)
        pdf_files.append(pdf_path)

# 打印待翻译文件总数
print(f"✅ 找到 {len(pdf_files)} 个 PDF 文件，开始逐份翻译...\n")

# 逐份翻译
for idx, pdf_path in enumerate(pdf_files, 1):
    print(f"[{idx}/{len(pdf_files)}] 正在翻译：{pdf_path}")

    try:
        # 翻译当前 PDF
        file_mono, file_dual = translate(files=[pdf_path], **params)[0]

        print(f"✅ 翻译完成：")
        print(f"   单语版：{file_mono}")
        print(f"   双语版：{file_dual}\n")

    except Exception as e:
        print(f"❌ 翻译失败：{pdf_path}")
        print(f"   错误信息：{str(e)}\n")

print("🎉 所有 PDF 翻译任务执行完毕！")
```

### 📖 pdf2zh 桌面版 · 开箱即用的 PDF 学术翻译神器 🚀

[开箱即用的免费桌面 PDF 论文翻译神器｜保留排版 + 公式无损 + 批量任务+ 历史追踪 + 长文本与扫描版增强｜基于 PDFMathTranslate 的 GUI 版](https://github.com/AaronGIG/pdf2zh-desktop)

**🎉 无需安装 Python · 无需配置环境 · 下载解压双击就能用！**

> 基于 [PDFMathTranslate](https://github.com/Byaidu/PDFMathTranslate)（EMNLP 2025）打造，在原项目基础上大幅增强桌面体验。

让学术 PDF 翻译变得像复制粘贴一样简单——公式、图表、排版全部完美保留 ✨

------

#### 🤔 为什么选择桌面版？

还在为翻译一篇论文折腾 Python 环境？还在对着黑窗口敲命令行？

**桌面版帮你把这些烦恼统统打包带走 👋**

| -        | 原版（Web/CLI）🖥️    | ✨ 桌面版           |
| -------- | ------------------- | ------------------ |
| 安装方式 | 需要 Python + pip 😵 | 解压即用 🎁         |
| 操作界面 | 浏览器 / 终端       | 原生 Windows GUI 🪟 |
| 翻译预览 | 浏览器内查看        | 内置 PDF 预览器 👁️  |
| 批量处理 | 命令行参数          | 界面一键操作 🖱️     |
| 离线能力 | 不支持              | 程序本体完全离线 📴 |

------

#### ✨ 桌面版增强亮点

🎯 真正的「零门槛」

- 📦 **完全独立打包**：Python 3.12 运行时 + 所有依赖全部内置，不污染你的系统
- 🖱️ **告别命令行**：全图形化操作，拖拽文件就能翻译
- 🔧 **智能错误诊断**：出问题？程序自动弹窗告诉你怎么修
- 💼 **真·便携版**：拷贝到 U 盘带着走，换台电脑照样用

🚀 超长文档？不在话下！

- 📄 1000+ 页的大部头轻松拿下
- 🧩 **分块翻译**：自定义每块页数（5~200 页），自动分块逐段翻译，内置限流延迟，翻完自动拼接完整文档
- 🧠 **智能内存管理**：逐页释放布局数组，即使上千页也不会内存溢出
- ⏯️ 断点续传——中途退出也不怕，下次自动接着翻，不浪费一分 API 额度
- 📜 **扫描版 PDF 支持**：自动为译文区域生成白色背景，覆盖底图原文，扫描版书籍也能获得清晰的翻译结果

📚 历史记录 & 实时预览

- 🗂️ 完整翻译历史，键盘上下键快速切换，随时回看
- 👀 内置 PDF 预览器，翻译效果所见即所得
- 🔍 翻译前后对比，一目了然
- 📝 同一文件多次翻译自动编号（`文件(1)`、`文件(2)`），不覆盖历史结果

📁 批量翻译

- 📂 一次丢进来一堆 PDF，挨个翻译，每个文件独立跟踪进度
- 🎯 智能文件识别，只翻 PDF，不怕误操作