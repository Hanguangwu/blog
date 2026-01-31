---
title: 最小的编程助手智能体nanocode
description: 本文介绍最小的编程助手智能体nanocode。
date: 2026-01-31T21:34:25-08:00
draft: false
categories:
- APP
- AI
- LLM
tags:
- GitHub
- nanocode
---


# 最小的编程助手智能体nanocode

## 前言

今天听了一场“function call is all you need”的AI 编程演讲，演讲者从一个大语言模型和一个Bash命令行工具开始增加文件读写等功能，然后搭建一个编程助手智能体。这让我觉得很神奇，后续也就知道了nanocode这个项目。这里详细解释一下nanocode的源码，方便我开发自己的编程助手智能体。

## 简介

[Minimal Claude Code alternative. Single Python file, zero dependencies, ~250 lines.](https://github.com/1rgs/nanocode)

Built using Claude Code, then used to build itself.

![](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20260131190600886.png)
## 特色

- Full agentic loop with tool use
- Tools: `read`, `write`, `edit`, `glob`, `grep`, `bash`
- Conversation history
- Colored terminal output

## 使用方式

```shell
export ANTHROPIC_API_KEY="your-key"
python nanocode.py
```

### 配置 OpenRouter

Use [OpenRouter](https://openrouter.ai/) to access any model:

```shell
export OPENROUTER_API_KEY="your-key"
python nanocode.py
```

To use a different model:

```shell
export OPENROUTER_API_KEY="your-key"
export MODEL="openai/gpt-5.2"
python nanocode.py
```

## 命令

- `/c` - Clear conversation
- `/q` or `exit` - Quit

## 工具

|Tool|Description|
|---|---|
|`read`|Read file with line numbers, offset/limit|
|`write`|Write content to file|
|`edit`|Replace string in file (must be unique)|
|`glob`|Find files by pattern, sorted by mtime|
|`grep`|Search files for regex|
|`bash`|Run shell command|

## 使用示例

```
────────────────────────────────────────
❯ what files are here?
────────────────────────────────────────

⏺ Glob(**/*.py)
  ⎿  nanocode.py

⏺ There's one Python file: nanocode.py
```

## 源码解析






# 轻量级编程助手智能体 nanocode

## 前言

今日我聆听了一场主题为“function call is all you need”（函数调用即是全部所需）的AI编程演讲。演讲者以一款大语言模型和一个Bash命令行工具为基础，逐步为其拓展文件读写等核心功能，最终搭建出一套完整的编程助手智能体。这场演讲令人深受启发，也让我后续了解到了nanocode这个优秀项目。在此，我将对nanocode的源码进行详细解析，以便为我后续开发专属编程助手智能体提供参考与借鉴。

## 简介

[一款轻量级 Claude Code 替代方案。仅含单个 Python 文件，零外部依赖，代码量约 250 行。](https://github.com/1rgs/nanocode)

该项目基于 Claude Code 构建开发，而后又借助其自身完成了进一步的迭代优化。

![](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20260131190600886.png)
## 特色

- 具备完整的工具调用智能体循环流程
- 内置工具：`read`（读取文件）、`write`（写入文件）、`edit`（编辑文件）、`glob`（文件匹配）、`grep`（内容检索）、`bash`（执行Shell命令）
- 支持对话历史记录
- 终端输出带有彩色高亮效果

## 使用方式

```shell
export ANTHROPIC_API_KEY="你的API密钥"
python nanocode.py
```

### 配置 OpenRouter

可使用 [OpenRouter](https://openrouter.ai/) 调用各类模型：

```shell
export OPENROUTER_API_KEY="你的API密钥"
python nanocode.py
```

指定其他模型使用：

```shell
export OPENROUTER_API_KEY="你的API密钥"
export MODEL="openai/gpt-5.2"
python nanocode.py
```

## 命令

- `/c`  - 清空当前对话历史
- `/q` 或 `exit`  - 退出程序

## 工具

| 工具 | 描述 |
|---|---|
|`read`| 读取文件内容并附带行号，支持设置偏移量/读取限制 |
|`write`| 向文件中写入指定内容 |
|`edit`| 替换文件中的指定字符串（该字符串必须具有唯一性） |
|`glob`| 按照指定模式查找文件，结果按文件修改时间排序 |
|`grep`| 按照正则表达式检索文件内容 |
|`bash`| 执行Shell命令 |


```
❯ how many files in this folder?
────────────────────────────────────────────────────────────────────────────────
⏺ Error: Expecting value: line 1 column 1 (char 0)
```
## 使用示例

```
────────────────────────────────────────
❯ 这里有哪些文件？
────────────────────────────────────────

⏺ 执行文件匹配(**/*.py)
  ⎿  nanocode.py

⏺ 当前目录下存在一个Python文件：nanocode.py
```

## 源码

```python
#!/usr/bin/env python3
"""nanocode - minimal claude code alternative"""

import glob as globlib, json, os, re, subprocess, urllib.request

OPENROUTER_KEY = os.environ.get("OPENROUTER_API_KEY")
API_URL = "https://openrouter.ai/api/v1/messages" if OPENROUTER_KEY else "https://api.anthropic.com/v1/messages"
MODEL = os.environ.get("MODEL", "anthropic/claude-opus-4.5" if OPENROUTER_KEY else "claude-opus-4-5")

# ANSI colors
RESET, BOLD, DIM = "\033[0m", "\033[1m", "\033[2m"
BLUE, CYAN, GREEN, YELLOW, RED = (
    "\033[34m",
    "\033[36m",
    "\033[32m",
    "\033[33m",
    "\033[31m",
)


# --- Tool implementations ---


def read(args):
    lines = open(args["path"]).readlines()
    offset = args.get("offset", 0)
    limit = args.get("limit", len(lines))
    selected = lines[offset : offset + limit]
    return "".join(f"{offset + idx + 1:4}| {line}" for idx, line in enumerate(selected))


def write(args):
    with open(args["path"], "w") as f:
        f.write(args["content"])
    return "ok"


def edit(args):
    text = open(args["path"]).read()
    old, new = args["old"], args["new"]
    if old not in text:
        return "error: old_string not found"
    count = text.count(old)
    if not args.get("all") and count > 1:
        return f"error: old_string appears {count} times, must be unique (use all=true)"
    replacement = (
        text.replace(old, new) if args.get("all") else text.replace(old, new, 1)
    )
    with open(args["path"], "w") as f:
        f.write(replacement)
    return "ok"


def glob(args):
    pattern = (args.get("path", ".") + "/" + args["pat"]).replace("//", "/")
    files = globlib.glob(pattern, recursive=True)
    files = sorted(
        files,
        key=lambda f: os.path.getmtime(f) if os.path.isfile(f) else 0,
        reverse=True,
    )
    return "\n".join(files) or "none"


def grep(args):
    pattern = re.compile(args["pat"])
    hits = []
    for filepath in globlib.glob(args.get("path", ".") + "/**", recursive=True):
        try:
            for line_num, line in enumerate(open(filepath), 1):
                if pattern.search(line):
                    hits.append(f"{filepath}:{line_num}:{line.rstrip()}")
        except Exception:
            pass
    return "\n".join(hits[:50]) or "none"


def bash(args):
    proc = subprocess.Popen(
        args["cmd"], shell=True,
        stdout=subprocess.PIPE, stderr=subprocess.STDOUT,
        text=True
    )
    output_lines = []
    try:
        while True:
            line = proc.stdout.readline()
            if not line and proc.poll() is not None:
                break
            if line:
                print(f"  {DIM}│ {line.rstrip()}{RESET}", flush=True)
                output_lines.append(line)
        proc.wait(timeout=30)
    except subprocess.TimeoutExpired:
        proc.kill()
        output_lines.append("\n(timed out after 30s)")
    return "".join(output_lines).strip() or "(empty)"


# --- Tool definitions: (description, schema, function) ---

TOOLS = {
    "read": (
        "Read file with line numbers (file path, not directory)",
        {"path": "string", "offset": "number?", "limit": "number?"},
        read,
    ),
    "write": (
        "Write content to file",
        {"path": "string", "content": "string"},
        write,
    ),
    "edit": (
        "Replace old with new in file (old must be unique unless all=true)",
        {"path": "string", "old": "string", "new": "string", "all": "boolean?"},
        edit,
    ),
    "glob": (
        "Find files by pattern, sorted by mtime",
        {"pat": "string", "path": "string?"},
        glob,
    ),
    "grep": (
        "Search files for regex pattern",
        {"pat": "string", "path": "string?"},
        grep,
    ),
    "bash": (
        "Run shell command",
        {"cmd": "string"},
        bash,
    ),
}


def run_tool(name, args):
    try:
        return TOOLS[name][2](args)
    except Exception as err:
        return f"error: {err}"


def make_schema():
    result = []
    for name, (description, params, _fn) in TOOLS.items():
        properties = {}
        required = []
        for param_name, param_type in params.items():
            is_optional = param_type.endswith("?")
            base_type = param_type.rstrip("?")
            properties[param_name] = {
                "type": "integer" if base_type == "number" else base_type
            }
            if not is_optional:
                required.append(param_name)
        result.append(
            {
                "name": name,
                "description": description,
                "input_schema": {
                    "type": "object",
                    "properties": properties,
                    "required": required,
                },
            }
        )
    return result


def call_api(messages, system_prompt):
    request = urllib.request.Request(
        API_URL,
        data=json.dumps(
            {
                "model": MODEL,
                "max_tokens": 8192,
                "system": system_prompt,
                "messages": messages,
                "tools": make_schema(),
            }
        ).encode(),
        headers={
            "Content-Type": "application/json",
            "anthropic-version": "2023-06-01",
            **({"Authorization": f"Bearer {OPENROUTER_KEY}"} if OPENROUTER_KEY else {"x-api-key": os.environ.get("ANTHROPIC_API_KEY", "")}),
        },
    )
    response = urllib.request.urlopen(request)
    return json.loads(response.read())


def separator():
    return f"{DIM}{'─' * min(os.get_terminal_size().columns, 80)}{RESET}"


def render_markdown(text):
    return re.sub(r"\*\*(.+?)\*\*", f"{BOLD}\\1{RESET}", text)


def main():
    print(f"{BOLD}nanocode{RESET} | {DIM}{MODEL} ({'OpenRouter' if OPENROUTER_KEY else 'Anthropic'}) | {os.getcwd()}{RESET}\n")
    messages = []
    system_prompt = f"Concise coding assistant. cwd: {os.getcwd()}"

    while True:
        try:
            print(separator())
            user_input = input(f"{BOLD}{BLUE}❯{RESET} ").strip()
            print(separator())
            if not user_input:
                continue
            if user_input in ("/q", "exit"):
                break
            if user_input == "/c":
                messages = []
                print(f"{GREEN}⏺ Cleared conversation{RESET}")
                continue

            messages.append({"role": "user", "content": user_input})

            # agentic loop: keep calling API until no more tool calls
            while True:
                response = call_api(messages, system_prompt)
                content_blocks = response.get("content", [])
                tool_results = []

                for block in content_blocks:
                    if block["type"] == "text":
                        print(f"\n{CYAN}⏺{RESET} {render_markdown(block['text'])}")

                    if block["type"] == "tool_use":
                        tool_name = block["name"]
                        tool_args = block["input"]
                        arg_preview = str(list(tool_args.values())[0])[:50]
                        print(
                            f"\n{GREEN}⏺ {tool_name.capitalize()}{RESET}({DIM}{arg_preview}{RESET})"
                        )

                        result = run_tool(tool_name, tool_args)
                        result_lines = result.split("\n")
                        preview = result_lines[0][:60]
                        if len(result_lines) > 1:
                            preview += f" ... +{len(result_lines) - 1} lines"
                        elif len(result_lines[0]) > 60:
                            preview += "..."
                        print(f"  {DIM}⎿  {preview}{RESET}")

                        tool_results.append(
                            {
                                "type": "tool_result",
                                "tool_use_id": block["id"],
                                "content": result,
                            }
                        )

                messages.append({"role": "assistant", "content": content_blocks})

                if not tool_results:
                    break
                messages.append({"role": "user", "content": tool_results})

            print()

        except (KeyboardInterrupt, EOFError):
            break
        except Exception as err:
            print(f"{RED}⏺ Error: {err}{RESET}")


if __name__ == "__main__":
    main()
```

## 一、代码整体功能总结

这份代码实现了一个**轻量级、零外部依赖（仅使用Python内置库）的AI编程助手智能体**，核心能力是通过调用Anthropic或OpenRouter的API，结合内置的6种实用工具（文件读写、查找、检索、Shell命令执行等），完成交互式的编程辅助任务。它的核心亮点是「完整的智能体工具调用循环」——AI会根据用户需求自动选择工具执行，再根据工具返回结果继续处理，直到完成任务，全程在终端以彩色输出提升可读性。

## 二、分模块详细解析

我们按照代码的逻辑结构，拆分为以下几个核心模块进行解释：

### 1. 头部配置与常量定义
这部分是程序的初始化准备，负责读取环境变量、配置API信息和终端彩色输出样式。
```python
#!/usr/bin/env python3
"""nanocode - minimal claude code alternative"""

# 导入内置依赖库，无需额外安装
import glob as globlib, json, os, re, subprocess, urllib.request

# 1. 读取环境变量，配置API相关参数
# 读取OpenRouter API密钥（优先使用OpenRouter，无则使用Anthropic）
OPENROUTER_KEY = os.environ.get("OPENROUTER_API_KEY")
# 确定API请求地址：有OpenRouter密钥则用OpenRouter，否则用Anthropic
API_URL = "https://openrouter.ai/api/v1/messages" if OPENROUTER_KEY else "https://api.anthropic.com/v1/messages"
# 确定使用的模型：优先从环境变量读取，无则默认对应平台的高性能模型
MODEL = os.environ.get("MODEL", "anthropic/claude-opus-4.5" if OPENROUTER_KEY else "claude-opus-4-5")

# 2. 定义ANSI终端转义序列，实现彩色输出和格式控制
RESET, BOLD, DIM = "\033[0m", "\033[1m", "\033[2m"  # 重置、加粗、暗淡
BLUE, CYAN, GREEN, YELLOW, RED = (
    "\033[34m",
    "\033[36m",
    "\033[32m",
    "\033[33m",
    "\033[31m",
)
```
- 关键说明：`os.environ.get()` 用于安全读取系统环境变量，避免直接访问不存在的环境变量抛出异常；ANSI转义序列是终端的通用格式，用于让输出内容有不同颜色和样式，提升用户体验。

### 2. 工具实现模块（核心工具函数）
这部分实现了6个核心工具的具体逻辑，对应`TOOLS`配置中的功能，是智能体能够操作文件和执行命令的核心。

#### （1）`read()`：读取文件内容（附带行号，支持偏移/限制）
```python
def read(args):
    # 打开文件并按行读取所有内容
    lines = open(args["path"]).readlines()
    # 获取偏移量（默认从第0行开始）和读取限制（默认读取全部）
    offset = args.get("offset", 0)
    limit = args.get("limit", len(lines))
    # 截取指定范围的行
    selected = lines[offset : offset + limit]
    # 拼接结果，附带行号（格式：4位数字| 行内容），提升可读性
    return "".join(f"{offset + idx + 1:4}| {line}" for idx, line in enumerate(selected))
```
- 入参：`args`是字典，必须包含`path`（文件路径），可选`offset`（起始行偏移）、`limit`（读取行数）。
- 返回：带行号的文件内容字符串。

#### （2）`write()`：向文件写入内容（覆盖写入）
```python
def write(args):
    # 以"w"（写入模式）打开文件，不存在则创建，存在则覆盖
    with open(args["path"], "w") as f:
        f.write(args["content"])
    return "ok"
```
- 入参：`args`必须包含`path`（文件路径）、`content`（要写入的内容）。
- 注意：使用`with`语句自动关闭文件，避免资源泄露；该方法是**覆盖写入**，会清空文件原有内容。

#### （3）`edit()`：替换文件中的指定字符串
```python
def edit(args):
    # 读取文件全部内容
    text = open(args["path"]).read()
    old, new = args["old"], args["new"]
    # 校验1：旧字符串是否存在
    if old not in text:
        return "error: old_string not found"
    count = text.count(old)
    # 校验2：未指定all=true时，旧字符串必须唯一（避免误替换）
    if not args.get("all") and count > 1:
        return f"error: old_string appears {count} times, must be unique (use all=true)"
    # 执行替换：all=true则替换所有，否则只替换第1次出现
    replacement = (
        text.replace(old, new) if args.get("all") else text.replace(old, new, 1)
    )
    # 写入替换后的内容（覆盖原文件）
    with open(args["path"], "w") as f:
        f.write(replacement)
    return "ok"
```
- 入参：必须包含`path`、`old`（待替换字符串）、`new`（新字符串），可选`all`（是否替换所有匹配项）。
- 核心逻辑：先做合法性校验，再执行替换，最后覆盖写入，保证替换操作的安全性。

#### （4）`glob()`：按模式查找文件（按修改时间倒序排序）
```python
def glob(args):
    # 拼接文件查找模式（处理路径中的双斜杠，转为单斜杠）
    pattern = (args.get("path", ".") + "/" + args["pat"]).replace("//", "/")
    # 递归查找符合模式的所有文件/目录
    files = globlib.glob(pattern, recursive=True)
    # 排序：按文件修改时间（mtime）倒序，非文件（目录）排最后
    files = sorted(
        files,
        key=lambda f: os.path.getmtime(f) if os.path.isfile(f) else 0,
        reverse=True,
    )
    # 返回结果：换行分隔文件路径，无结果返回"none"
    return "\n".join(files) or "none"
```
- 入参：必须包含`pat`（查找模式，如`**/*.py`），可选`path`（起始路径，默认当前目录）。
- 关键：`globlib.glob(recursive=True)`支持递归查找（`**`表示所有子目录），排序逻辑保证最新修改的文件优先展示。

#### （5）`grep()`：按正则表达式检索文件内容
```python
def grep(args):
    # 编译正则表达式模式，提升匹配效率
    pattern = re.compile(args["pat"])
    hits = []
    # 递归遍历所有文件/目录
    for filepath in globlib.glob(args.get("path", ".") + "/**", recursive=True):
        try:
            # 按行读取文件，检索匹配内容
            for line_num, line in enumerate(open(filepath), 1):
                if pattern.search(line):
                    # 记录：文件路径:行号:匹配行内容（去除末尾换行符）
                    hits.append(f"{filepath}:{line_num}:{line.rstrip()}")
        except Exception:
            pass  # 忽略无法读取的文件（如目录、权限不足文件）
    # 返回前50条匹配结果，避免结果过多溢出
    return "\n".join(hits[:50]) or "none"
```
- 入参：必须包含`pat`（正则表达式模式），可选`path`（起始路径）。
- 核心：结合`glob`遍历文件，用正则表达式匹配行内容，返回带位置信息的匹配结果，最多返回50条。

#### （6）`bash()`：执行Shell命令（带超时控制，实时输出）
```python
def bash(args):
    # 启动子进程执行Shell命令，捕获标准输出/错误输出
    proc = subprocess.Popen(
        args["cmd"], shell=True,
        stdout=subprocess.PIPE, stderr=subprocess.STDOUT,  # 标准错误重定向到标准输出
        text=True  # 以文本模式读取输出，而非字节流
    )
    output_lines = []
    try:
        # 实时读取子进程输出，并打印到终端
        while True:
            line = proc.stdout.readline()
            # 终止条件：无输出且进程已退出
            if not line and proc.poll() is not None:
                break
            if line:
                print(f"  {DIM}│ {line.rstrip()}{RESET}", flush=True)
                output_lines.append(line)
        # 等待进程执行完成，超时30秒
        proc.wait(timeout=30)
    except subprocess.TimeoutExpired:
        # 超时后杀死进程，记录超时信息
        proc.kill()
        output_lines.append("\n(timed out after 30s)")
    # 返回命令执行结果（去除首尾空白，无结果返回"(empty)"）
    return "".join(output_lines).strip() or "(empty)"
```
- 入参：必须包含`cmd`（要执行的Shell命令）。
- 关键：`subprocess.Popen`创建子进程，实时输出命令执行日志，30秒超时控制避免进程挂起，提升安全性。

### 3. 工具配置与辅助函数模块
这部分用于管理工具、生成工具Schema、执行工具调用，是连接工具函数与AI API的桥梁。

#### （1）`TOOLS`字典：工具元信息配置
```python
TOOLS = {
    "read": (
        "Read file with line numbers (file path, not directory)",
        {"path": "string", "offset": "number?", "limit": "number?"},
        read,
    ),
    # 其他工具配置省略...
}
```
- 结构：每个工具对应一个三元组 `(工具描述, 入参Schema, 工具函数)`。
- 入参Schema说明：`?`表示可选参数（如`offset: number?`），无`?`表示必选参数，用于生成AI可识别的工具Schema。

#### （2）`run_tool()`：工具执行封装（带异常捕获）
```python
def run_tool(name, args):
    try:
        # 从TOOLS中获取对应的工具函数并执行
        return TOOLS[name][2](args)
    except Exception as err:
        # 捕获工具执行中的所有异常，返回友好错误信息
        return f"error: {err}"
```
- 作用：统一封装工具调用逻辑，添加异常捕获，避免单个工具执行失败导致整个程序崩溃。

#### （3）`make_schema()`：生成AI API可识别的工具Schema
```python
def make_schema():
    result = []
    for name, (description, params, _fn) in TOOLS.items():
        properties = {}
        required = []
        for param_name, param_type in params.items():
            # 判断参数是否可选
            is_optional = param_type.endswith("?")
            # 获取参数基础类型（去除末尾的?）
            base_type = param_type.rstrip("?")
            # 构建参数属性（number类型转为integer，适配API要求）
            properties[param_name] = {
                "type": "integer" if base_type == "number" else base_type
            }
            # 必选参数加入required列表
            if not is_optional:
                required.append(param_name)
        # 拼接单个工具的Schema
        result.append(
            {
                "name": name,
                "description": description,
                "input_schema": {
                    "type": "object",
                    "properties": properties,
                    "required": required,
                },
            }
        )
    return result
```
- 作用：将`TOOLS`字典中的简易配置，转换为Anthropic/OpenRouter API要求的标准工具Schema格式，让AI能够理解并调用工具。
- 关键：处理可选参数和类型转换（`number`→`integer`），符合API的数据格式要求。

#### （4）`call_api()`：调用AI API（发送请求，获取响应）
```python
def call_api(messages, system_prompt):
    # 构建API请求体
    request = urllib.request.Request(
        API_URL,
        data=json.dumps(
            {
                "model": MODEL,
                "max_tokens": 8192,
                "system": system_prompt,
                "messages": messages,
                "tools": make_schema(),
            }
        ).encode(),  # 转为JSON字节流
        headers={
            "Content-Type": "application/json",
            "anthropic-version": "2023-06-01",
            # 动态构建授权头：区分OpenRouter和Anthropic
            **({"Authorization": f"Bearer {OPENROUTER_KEY}"} if OPENROUTER_KEY else {"x-api-key": os.environ.get("ANTHROPIC_API_KEY", "")}),
        },
    )
    # 发送请求并获取响应
    response = urllib.request.urlopen(request)
    # 解析JSON响应为Python字典并返回
    return json.loads(response.read())
```
- 入参：`messages`（对话历史）、`system_prompt`（系统提示词，定义AI的角色和行为）。
- 关键：使用Python内置`urllib.request`发送HTTP请求，无需额外安装`requests`库，符合「零依赖」要求；动态构建请求头，适配两种API平台。

### 4. 终端格式辅助函数
这部分函数用于优化终端输出格式，提升用户体验。
```python
def separator():
    # 生成终端宽度的分隔线（最多80个字符）
    return f"{DIM}{'─' * min(os.get_terminal_size().columns, 80)}{RESET}"

def render_markdown(text):
    # 简单渲染Markdown加粗格式（**内容** → 终端加粗样式）
    return re.sub(r"\*\*(.+?)\*\*", f"{BOLD}\\1{RESET}", text)
```

### 5. 主函数`main()`：程序入口与核心循环
这部分是程序的控制中心，实现了交互式对话、智能体工具调用循环、终端输出等核心逻辑。
```python
def main():
    # 打印程序初始化信息（名称、模型、当前工作目录）
    print(f"{BOLD}nanocode{RESET} | {DIM}{MODEL} ({'OpenRouter' if OPENROUTER_KEY else 'Anthropic'}) | {os.getcwd()}{RESET}\n")
    messages = []  # 存储对话历史
    system_prompt = f"Concise coding assistant. cwd: {os.getcwd()}"  # 系统提示词

    while True:
        try:
            # 1. 终端交互：获取用户输入
            print(separator())
            user_input = input(f"{BOLD}{BLUE}❯{RESET} ").strip()
            print(separator())
            if not user_input:
                continue
            # 退出命令
            if user_input in ("/q", "exit"):
                break
            # 清空对话历史命令
            if user_input == "/c":
                messages = []
                print(f"{GREEN}⏺ Cleared conversation{RESET}")
                continue

            # 2. 将用户输入加入对话历史
            messages.append({"role": "user", "content": user_input})

            # 3. 核心：智能体工具调用循环（直到无工具调用为止）
            while True:
                # 调用AI API获取响应
                response = call_api(messages, system_prompt)
                content_blocks = response.get("content", [])
                tool_results = []  # 存储工具执行结果

                # 解析API响应内容
                for block in content_blocks:
                    # 文本内容：直接打印到终端
                    if block["type"] == "text":
                        print(f"\n{CYAN}⏺{RESET} {render_markdown(block['text'])}")

                    # 工具调用：执行对应的工具，并收集结果
                    if block["type"] == "tool_use":
                        tool_name = block["name"]
                        tool_args = block["input"]
                        # 打印工具调用预览
                        arg_preview = str(list(tool_args.values())[0])[:50]
                        print(
                            f"\n{GREEN}⏺ {tool_name.capitalize()}{RESET}({DIM}{arg_preview}{RESET})"
                        )

                        # 执行工具并获取结果
                        result = run_tool(tool_name, tool_args)
                        # 处理结果预览，避免输出过长
                        result_lines = result.split("\n")
                        preview = result_lines[0][:60]
                        if len(result_lines) > 1:
                            preview += f" ... +{len(result_lines) - 1} lines"
                        elif len(result_lines[0]) > 60:
                            preview += "..."
                        print(f"  {DIM}⎿  {preview}{RESET}")

                        # 收集工具执行结果（用于反馈给AI）
                        tool_results.append(
                            {
                                "type": "tool_result",
                                "tool_use_id": block["id"],
                                "content": result,
                            }
                        )

                # 4. 更新对话历史：添加AI响应
                messages.append({"role": "assistant", "content": content_blocks})

                # 5. 终止循环条件：无工具调用结果（任务完成）
                if not tool_results:
                    break
                # 6. 有工具调用结果：将结果加入对话历史，继续调用AI
                messages.append({"role": "user", "content": tool_results})

            print()

        # 捕获用户中断（Ctrl+C）和EOF错误，优雅退出
        except (KeyboardInterrupt, EOFError):
            break
        # 捕获其他异常，打印友好错误信息
        except Exception as err:
            print(f"{RED}⏺ Error: {err}{RESET}")
```
- 核心逻辑流程：
  1.  初始化对话历史和系统提示词。
  2.  进入交互式循环，获取用户输入并处理命令（退出/清空历史）。
  3.  将用户输入加入对话历史，调用AI API。
  4.  解析API响应：如果是文本，直接输出；如果是工具调用，执行工具并收集结果。
  5.  更新对话历史，若有工具执行结果，将其反馈给AI，继续循环（让AI基于工具结果做下一步处理）。
  6.  无工具调用时，终止循环，等待用户下一次输入。

### 6. 程序入口
```python
if __name__ == "__main__":
    main()
```
- 作用：当直接运行该Python文件时，执行`main()`函数，启动程序；若被作为模块导入，不执行主逻辑，符合Python编程规范。

## 三、整体分析（核心亮点与设计思路）
### 1. 核心设计亮点
- **零外部依赖**：仅使用Python内置库，无需`pip install`任何包，可直接在有Python3的环境中运行，便携性极强。
- **完整的智能体循环**：实现了「用户输入→AI决策→工具执行→结果反馈→AI再决策」的闭环，符合智能体的核心特征。
- **简洁高效**：约250行代码实现完整功能，结构清晰，工具与核心逻辑解耦，易于扩展（新增工具只需在`TOOLS`中配置并实现对应函数）。
- **用户体验友好**：终端彩色输出、结果预览、命令行交互、异常友好提示，提升了使用便捷性。
- **多平台兼容**：支持Anthropic和OpenRouter两种API平台，可灵活切换模型。

### 2. 潜在局限性
- **文件操作风险**：`write()`、`edit()`为覆盖写入，无备份机制，可能误删文件内容。
- **Shell命令安全**：`bash()`支持执行任意Shell命令，存在安全风险（如`rm -rf /`），不适合在生产环境使用。
- **Markdown渲染有限**：仅支持加粗格式，不支持其他Markdown语法（如列表、代码块）。
- **无持久化存储**：对话历史仅存于内存，程序退出后丢失，无法恢复之前的对话。

### 3. 整体架构梳理
```
nanocode程序架构
├── 初始化层（环境变量、常量、依赖导入）
├── 工具层（6个核心工具函数，实现具体操作）
├── 桥梁层（工具配置、Schema生成、API调用，连接工具与AI）
├── 格式层（终端输出格式优化，提升用户体验）
└── 控制层（main函数，实现交互式循环与智能体闭环）
```

### 总结
1.  该代码是一个**轻量级、零依赖的AI编程智能体**，核心是通过工具调用闭环完成编程辅助任务，结构清晰且易于扩展。
2.  核心模块分为「工具实现」和「智能体循环」，工具负责具体操作，循环负责决策与反馈，解耦设计提升了可维护性。
3.  亮点是便携性和完整的智能体逻辑，局限性是缺乏安全防护和持久化存储，适合用于学习和个人轻量场景，不适合生产环境。










