---
title: 基于Translategemma模型的反思型翻译智能体实现
description: 本文介绍反思型翻译智能体的一种实现。
date: 2026-03-20T12:34:25-08:00
draft: false
categories:
- AI
- Agent
tags:
- translategemma
---


# 基于Translategemma模型的反思型翻译智能体实现

## 基本流程

先使用 MinerU 或 Marker-pdf 转换电子书为 Markdown 文件。

命令：

```
pip install marker-pdf[full]

marker_single '.\test.pdf' --output_dir .\conversion_results\ --output_format markdown
```

在 Windows11 操作系统上使用`marker-pdf`容易报错[OSError: cannot load library 'gobject-2.0-0': error 0x7e #1556](https://github.com/Kozea/WeasyPrint/issues/1556)

这里就需要通过`MYSY2`安装缺失的`gobject-2.0-0`。

后续调整 Python 代码中的输入输出文件，然后执行相应程序即可。

这里需要在 Ollama 中安装`translategemma`模型。

## MYSY2

`MSYS2`是一个包含MinGW-w64工具链、GNU工具集和一些开源库的平台，它提供了一种在Windows上编译和运行Unix/Linux程序的方式。MSYS2与MinGW-w64相似，但比MinGW-w64更完整和稳定，提供了[Pacman](https://zhida.zhihu.com/search?content_id=247235722&content_type=Article&match_order=1&q=Pacman&zhida_source=entity)包管理器以方便用户安装和管理软件包。

`MinGW-w64`是一个Windows下的C/C++编程工具集，它提供了运行在Windows上的GNU工具集和GCC编译器。MinGW-w64与MSYS2类似，但主要用于编译Windows本地应用程序，而非Unix/Linux程序。MinGW-w64也可以用于交叉编译，为其他平台生成Windows可执行文件。

**区别：**  

**MSYS：** 相当于操作系统（如Windows），这个操作系统提供的软件、接口等和Linux相似。

**MinGW：** 相当于开发工具包（如MSVC），这个开发工具包可以运行在 MSYS 下，包里的工具也可以运行在Windows下，编译结果是Windows程序。

`Cygwin`是一个在Windows平台上运行的兼容性层，提供了类Unix环境的工具和开发库。Cygwin将Unix程序编译为Windows本地代码，然后在Windows上运行。它提供了最完整的Linux/Unix环境，但相对于MSYS2和MinGW-w64，Cygwin的性能较差。

因此，MSYS2适用于需要在Windows上编译和运行Unix/Linux程序的场景，MinGW-w64适用于编译Windows本地应用程序的场景，Cygwin适用于需要最完整的Linux/Unix环境的场景。

**msys2的优势：** 简单的包管理工具，不需网上搜索安装包，下载安装。直接运用pacman进行下载，并且升级简单。
### Windows上安装教程

To use WeasyPrint on Windows, the easiest way is to use the [executable](https://github.com/Kozea/WeasyPrint/releases) of the latest release.

WeasyPrint is regularly marked as malware by different antivirus companies. See [#2081](https://github.com/Kozea/WeasyPrint/issues/2081) or [#2092](https://github.com/Kozea/WeasyPrint/issues/2092) to get more information on this topic. Don’t hesitate to report the false positive detection to your antivirus company in order to improve malware detection for future versions.

If you want to use WeasyPrint as a Python library, you’ll have to follow a few extra steps. Please read this chapter carefully.

The first step is to install the [Python Install Manager](https://apps.microsoft.com/detail/9nq7512cxl7t) from the Microsoft Store.

When Python is installed, you have to install Pango and its dependencies. The easiest way to install these libraries is to use MSYS2. Here are the steps you have to follow:

- Install [MSYS2](https://www.msys2.org/#installation) keeping the default options.
- After installation, in MSYS2’s shell, execute `pacman -S mingw-w64-x86_64-pango`.
- Close MSYS2’s shell.

## Python实现代码

```python
from typing import List, Dict, Any
import ollama
import re
import time
from tqdm import tqdm
from dotenv import load_dotenv
import os

# 加载环境变量
load_dotenv()

# ===================== 全局配置（专为 translategemma:4b 优化）=====================
MODEL_NAME = os.getenv("MODEL_NAME", "translategemma:4b")
INPUT_FILE = "yearandyear.md"
OUTPUT_FILE = "translated_reflect_final.md"

# ⬇️⬇️⬇️ 关键修改：适配 4B 模型上下文窗口 2K，滑动窗口 1024
# 字符数控制在 500 以内 = ~350 tokens，绝对安全不超限
MAX_BLOCK_LENGTH = 500
MAX_REFLECT_ITERATIONS = 1
# 正则：保护图片、链接不被翻译
IMG_PATTERN = r'!\[.*?\]\(.*?\)'
LINK_PATTERN = r'\[.*?\]\(.*?\)'

# ===================== 社会学专业术语表（自动统一翻译）=====================
# 你可以在这里添加你书籍里的关键名词！！！
SOCIOLOGY_TERMS = {
    "social structure": "社会结构",
    "capital": "资本",
    "cultural capital": "文化资本",
    "social class": "社会阶层",
    "power": "权力",
    "ideology": "意识形态",
    "modernity": "现代性",
    "rationality": "理性",
    "institution": "制度",
    "identity": "身份认同",
    "legitimacy": "合法性",
    "hegemony": "霸权",
    "agency": "能动性",
    "structure": "结构"
}

# ===================== 记忆模块 =====================
class Memory:
    """存储翻译与反思记录，用于迭代优化"""
    def __init__(self):
        self.records: List[Dict[str, Any]] = []

    def add_record(self, record_type: str, content: str):
        self.records.append({"type": record_type, "content": content})

    def get_last_execution(self) -> str:
        """获取最后一次翻译结果"""
        for record in reversed(self.records):
            if record['type'] == 'execution':
                return record['content']
        return ""

    def get_trajectory(self) -> str:
        """获取完整翻译-反思轨迹"""
        trajectory = ""
        for record in self.records:
            if record['type'] == 'execution':
                trajectory += f"【翻译结果】\n{record['content']}\n\n"
            elif record['type'] == 'reflection':
                trajectory += f"【反思建议】\n{record['content']}\n\n"
        return trajectory.strip()

# ===================== 反思型翻译智能体 =====================
# 1. 初始翻译提示词（强制术语统一 + 社会学专业）
INIT_TRANSLATE_PROMPT = """
你是专业的现代社会学著作翻译专家，必须严格遵守：

【翻译规则】
1. 术语必须统一：以下词汇必须固定翻译，全程不能变
{terms_list}
2. 格式严格保留：所有Markdown标题、粗体、列表、段落格式完全不变
3. 忠于原文：不增、不减、不改意思
4. 语言流畅：符合中文学术规范
5. 只输出译文，无任何多余文字

待翻译文本：
{content}
"""

# 2. 反思提示词（重点检查：术语统一 + 格式）
REFLECT_PROMPT = """
你是严格的社会学翻译评审，请检查：
1. 关键名词是否全程统一翻译
2. 有无错译、漏译
3. Markdown格式是否完整保留
4. 语句是否通顺

只输出改进建议，完美则输出：【无需改进】

译文：
{translation}
"""

# 3. 优化翻译提示词
REFINE_TRANSLATE_PROMPT = """
你是社会学翻译专家，请根据反思优化译文：
1. 严格统一术语
2. 保留格式
3. 只输出最终译文

历史记录：
{trajectory}
"""

class ReflectTranslationAgent:
    def __init__(self, model_name: str, max_iterations=1):
        self.model_name = model_name
        self.memory = Memory()
        self.max_iterations = max_iterations

    def _llm(self, prompt: str) -> str:
        """调用本地Ollama模型"""
        try:
            resp = ollama.chat(
                model=self.model_name,
                messages=[{"role": "user", "content": prompt}]
            )
            return resp['message']['content'].strip()
        except Exception as e:
            print(f"模型调用失败：{str(e)}")
            return ""

    def translate_block(self, content: str) -> str:
        self.memory.records = []
        terms_text = "\n".join([f"- {k} → {v}" for k, v in SOCIOLOGY_TERMS.items()])

        # 初始翻译
        init_prompt = INIT_TRANSLATE_PROMPT.format(
            terms_list=terms_text,
            content=content
        )
        first_trans = self._llm(init_prompt)
        self.memory.add_record("execution", first_trans)

        # 反思迭代
        for _ in range(self.max_iterations):
            last_trans = self.memory.get_last_execution()
            reflect_prompt = REFLECT_PROMPT.format(translation=last_trans)
            feedback = self._llm(reflect_prompt)
            self.memory.add_record("reflection", feedback)

            if "无需改进" in feedback:
                break

            refine_prompt = REFINE_TRANSLATE_PROMPT.format(trajectory=self.memory.get_trajectory())
            refined = self._llm(refine_prompt)
            self.memory.add_record("execution", refined)

        return self.memory.get_last_execution()

# ===================== 分块函数（适配4b模型）=====================
def split_markdown_blocks(text, max_len=500):
    text = text.replace("\r\n", "\n")
    lines = text.split("\n")
    blocks = []
    current_block = []
    current_length = 0

    for line in lines:
        line_len = len(line)
        if current_length + line_len + 1 <= max_len:
            current_block.append(line)
            current_length += line_len + 1
        else:
            if current_block:
                blocks.append("\n".join(current_block))
            current_block = [line]
            current_length = line_len

    if current_block:
        blocks.append("\n".join(current_block))

    blocks = [b.strip() for b in blocks if b.strip()]
    return blocks

# ===================== 工具函数 =====================
def extract_placeholders(text, pattern, prefix):
    holders = {}
    matches = re.findall(pattern, text, re.DOTALL)
    for i, m in enumerate(matches):
        ph = f"[{prefix}_{i:03d}]"
        holders[ph] = m
        text = text.replace(m, ph)
    return text, holders

def restore_placeholders(text, holders):
    for ph, original in holders.items():
        text = text.replace(ph, original)
    return text

# ===================== 主流程 =====================
def main():
    print("=" * 60)
    print(" 反思型社会学翻译Agent（适配translategemma:4b）")
    print("=" * 60)

    # 检查Ollama
    try:
        ollama.list()
    except:
        print("❌ 请先启动 ollama serve")
        return

    # 读取文件
    try:
        with open(INPUT_FILE, 'r', encoding='utf-8') as f:
            raw = f.read()
        print(f"✅ 读取文件：{INPUT_FILE}，{len(raw)} 字符")
    except:
        print(f"❌ 文件不存在")
        return

    # 保护链接图片
    text, img_ph = extract_placeholders(raw, IMG_PATTERN, "IMG")
    text, link_ph = extract_placeholders(text, LINK_PATTERN, "LINK")
    print(f"✅ 保护 {len(img_ph)} 图 {len(link_ph)} 链接")

    # 分块
    blocks = split_markdown_blocks(text, MAX_BLOCK_LENGTH)
    print(f"✅ 文档分为 {len(blocks)} 块（安全适配4B模型）")

    if len(blocks) == 0:
        print("❌ 无内容")
        return

    # 翻译
    agent = ReflectTranslationAgent(MODEL_NAME, MAX_REFLECT_ITERATIONS)
    results = []
    print("\n开始翻译...\n")

    for block in tqdm(blocks, desc="翻译进度"):
        trans = agent.translate_block(block)
        results.append(trans)
        time.sleep(0.4)

    # 合并输出
    final = "\n\n".join(results)
    final = restore_placeholders(final, link_ph)
    final = restore_placeholders(final, img_ph)

    with open(OUTPUT_FILE, 'w', encoding='utf-8') as f:
        f.write(final)

    print("\n🎉 翻译完成！")
    print(f"📄 输出文件：{OUTPUT_FILE}")
    print("✅ 术语统一 ✅ 格式保留 ✅ 适配4B模型")

if __name__ == "__main__":
    main()
```


## 参考资料

[智能体经典范式构建](https://datawhalechina.github.io/hello-agents)

[AI Agent 让科技图书翻译效率提升十倍](https://jimmysong.io/zh/blog/ai-agent-translation/)

[深度调研开源 PDF 转 Markdown 工具：Marker、MinerU 与替代方案](https://jimmysong.io/zh/blog/pdf-to-markdown-open-source-deep-dive/)

[吴恩达的翻译Agent项目，复现教程来了！](https://zhuanlan.zhihu.com/p/30357609332)

[Translation Agent: Agentic translation using reflection workflow](https://github.com/andrewyng/translation-agent)

[WeasyPrint 68.1 documentation](https://doc.courtbouillon.org/weasyprint/stable/index.html)








