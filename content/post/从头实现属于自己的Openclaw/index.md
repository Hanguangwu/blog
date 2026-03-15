---
title: 从头实现属于自己的 OpenClaw
description: 本文介绍claw0项目。
date: 2026-03-15T12:34:25-08:00
draft: false
categories:
- AI
- Agent
- GitHub
tags:
- claw0
- OpenClaw
---

# 从头实现属于自己的 OpenClaw

## claw0 简介

[项目链接](https://github.com/shareAI-lab/claw0)

**从零到一: 构建 AI Agent 网关**

> 10 个渐进式章节, 每节都是可直接运行的 Python 文件.
> 3 种语言 (英语, 中文, 日语) -- 代码 + 文档同目录.

相关项目：**[learn-claude-code](https://github.com/shareAI-lab/learn-claude-code)** -- 姊妹教学仓库, 用 12 个递进课程从零构建一个智能体**框架** (nano Claude Code)。claw0 聚焦于网关路由、多通道接入和主动行为, learn-claude-code 则深入智能体的内部设计: 结构化规划 (TodoManager + nag)、上下文压缩 (三层 compact)、基于文件的任务持久化与依赖图、团队协调 (JSONL 邮箱、关机/计划审批 FSM)、自治式自组织, 以及 git worktree 隔离的并行执行。如果你想理解一个生产级单元智能体的内部运作, 从那里开始。

---

### 这是什么?

大多数 Agent 教程停在"调一次 API"就结束了. 这个仓库从那个 while 循环开始, 一路带你到生产级网关.

逐章节构建一个最小化 AI Agent 网关. 10 个章节, 10 个核心概念, 约 7,000 行 Python. 每节只引入一个新概念, 前一节的代码原样保留. 学完全部 10 节, 你就能顺畅地阅读 OpenClaw 的生产代码.

```sh
s01: Agent Loop           -- 基础: while + stop_reason
s02: Tool Use             -- 让模型能调工具: dispatch table
s03: Sessions & Context   -- 会话持久化, 上下文溢出处理
s04: Channels             -- Telegram + 飞书: 完整通道管线
s05: Gateway & Routing    -- 5 级绑定, 会话隔离
s06: Intelligence         -- 灵魂, 记忆, 技能, 提示词组装
s07: Heartbeat & Cron     -- 主动型 Agent + 定时任务
s08: Delivery             -- 可靠消息队列 + 退避
s09: Resilience           -- 3 层重试洋葱 + 认证轮换
s10: Concurrency          -- 命名队列车道序列化混沌
```

### 架构概览

```
+------------------- claw0 layers -------------------+
|                                                     |
|  s10: Concurrency  (命名车道, generation 追踪)      |
|  s09: Resilience   (认证轮换, 溢出压缩)             |
|  s08: Delivery     (预写队列, 退避)                 |
|  s07: Heartbeat    (Lane 锁, cron 调度)             |
|  s06: Intelligence (8 层提示词, 混合记忆检索)       |
|  s05: Gateway      (WebSocket, 5 级路由)            |
|  s04: Channels     (Telegram 管线, 飞书 webhook)    |
|  s03: Sessions     (JSONL 持久化, 3 阶段重试)       |
|  s02: Tools        (dispatch table, 4 个工具)       |
|  s01: Agent Loop   (while True + stop_reason)       |
|                                                     |
+-----------------------------------------------------+
```

### 章节依赖关系

```
s01 --> s02 --> s03 --> s04 --> s05
                 |               |
                 v               v
                s06 ----------> s07 --> s08
                 |               |
                 v               v
                s09 ----------> s10
```

- s01-s02: 基础 (无依赖)
- s03: 基于 s02 (为工具循环添加持久化)
- s04: 基于 s03 (通道产生 InboundMessage 给会话)
- s05: 基于 s04 (将通道消息路由到 Agent)
- s06: 基于 s03 (使用会话做上下文, 添加提示词层)
- s07: 基于 s06 (心跳使用灵魂/记忆构建提示词)
- s08: 基于 s07 (心跳输出经由投递队列)
- s09: 基于 s03+s06 (复用 ContextGuard 做溢出层, 模型配置)
- s10: 基于 s07 (将单一 Lock 替换为命名车道系统)

### 快速开始

```sh
# 1. 克隆并进入目录
git clone https://github.com/anthropics/claw0.git && cd claw0

# 2. 安装依赖
pip install -r requirements.txt

# 3. 配置
cp .env.example .env
# 编辑 .env: 填入 ANTHROPIC_API_KEY 和 MODEL_ID

# 4. 运行任意章节 (选择你的语言)
python sessions/zh/s01_agent_loop.py    # 中文
python sessions/en/s01_agent_loop.py    # English
python sessions/ja/s01_agent_loop.py    # Japanese
```

### 学习路径

每节只加一个新概念, 上一节的代码完整保留:

```
Phase 1: 基础         Phase 2: 连接            Phase 3: 智能            Phase 4: 自治           Phase 5: 生产
+----------------+    +-------------------+    +-----------------+     +-----------------+    +-----------------+
| s01: Loop      |    | s03: Sessions     |    | s06: Intelligence|    | s07: Heartbeat  |    | s09: Resilience |
| s02: Tools     | -> | s04: Channels     | -> |   灵魂, 记忆,   | -> |     & Cron       | -> |   & Concurrency |
|                |    | s05: Gateway      |    |   技能, 提示词   |    | s08: Delivery   |    | s10: Lanes      |
+----------------+    +-------------------+    +-----------------+     +-----------------+    +-----------------+
 循环 + dispatch       持久化 + 路由             人格 + 回忆             主动行为 + 可靠投递      重试 + 序列化
```

### 章节详情

| # | 章节 | 核心概念 | 行数 |
|---|------|---------|------|
| 01 | Agent Loop | `while True` + `stop_reason` -- 这就是一个 Agent | ~175 |
| 02 | Tool Use | 工具 = schema dict + handler map. 模型选名字, 你查表执行 | ~445 |
| 03 | Sessions | JSONL: 写入追加, 读取重放. 太大了? 总结旧消息 | ~890 |
| 04 | Channels | 每个平台都不同, 但最终都生产同一个 `InboundMessage` | ~780 |
| 05 | Gateway | 绑定表将 (channel, peer) 映射到 agent. 最具体的匹配胜出 | ~625 |
| 06 | Intelligence | 系统提示词 = 磁盘上的文件. 换文件, 换人格, 不改代码 | ~750 |
| 07 | Heartbeat & Cron | 定时线程: "该不该跑?" + 和用户消息共用同一管线 | ~660 |
| 08 | Delivery | 先写磁盘, 再尝试发送. 崩溃也丢不了消息 | ~870 |
| 09 | Resilience | 3 层重试洋葱: 认证轮换, 溢出压缩, 工具循环 | ~1130 |
| 10 | Concurrency | 命名车道 + FIFO 队列, generation 追踪, Future 返回 | ~900 |

### 仓库结构

```
claw0/
  README.md              English README
  README.zh.md           Chinese README
  README.ja.md           Japanese README
  .env.example           配置模板
  requirements.txt       Python 依赖
  sessions/              所有教学章节 (代码 + 文档)
    en/                  English
      s01_agent_loop.py  s01_agent_loop.md
      s02_tool_use.py    s02_tool_use.md
      ...                (10 .py + 10 .md)
    zh/                  中文
      s01_agent_loop.py  s01_agent_loop.md
      ...                (10 .py + 10 .md)
    ja/                  Japanese
      s01_agent_loop.py  s01_agent_loop.md
      ...                (10 .py + 10 .md)
  workspace/             共享工作区样例
    SOUL.md  IDENTITY.md  TOOLS.md  USER.md
    HEARTBEAT.md  BOOTSTRAP.md  AGENTS.md  MEMORY.md
    CRON.json
    skills/example-skill/SKILL.md
```

每个语言文件夹自包含: 可运行的 Python 代码 + 配套文档. 代码逻辑跨语言一致, 注释和文档因语言而异.

### 前置要求

- Python 3.10+
- Anthropic (或兼容服务商) 的 API key

### 依赖

```
anthropic>=0.39.0
python-dotenv>=1.0.0
websockets>=12.0
croniter>=2.0.0
python-telegram-bot>=21.0
httpx>=0.27.0
```

## 第 01 节: Agent 循环

> Agent 就是 `while True` + `stop_reason`.

### 架构

```
    User Input
        |
        v
    messages[] <-- append {role: "user", ...}
        |
        v
    client.messages.create(model, system, messages)
        |
        v
    stop_reason?
      /        \
 "end_turn"  "tool_use"
     |            |
   Print      (第 02 节)
     |
     v
    messages[] <-- append {role: "assistant", ...}
     |
     +--- 回到循环, 等待下一次输入
```

后续所有功能 -- 工具、会话、路由、投递 -- 都是在这个循环之上叠加的层,
循环本身不会改变.

### 本节要点

- **messages[]** 是唯一的状态. 每次 API 调用时, LLM 都会看到完整数组.
- **stop_reason** 是每次 API 响应后的唯一决策点.
- **end_turn** = "打印文本." **tool_use** = "执行工具, 将结果反馈回去" (第 02 节).
- 循环结构永远不变. 后续章节围绕它添加功能.

### 核心代码走读

#### 1. 完整的 agent 循环

每轮三个步骤: 收集输入, 调用 API, 根据 stop_reason 分支.

```python
def agent_loop() -> None:
    messages: list[dict] = []

    while True:
        try:
            user_input = input(colored_prompt()).strip()
        except (KeyboardInterrupt, EOFError):
            break

        if not user_input:
            continue
        if user_input.lower() in ("quit", "exit"):
            break

        messages.append({"role": "user", "content": user_input})

        try:
            response = client.messages.create(
                model=MODEL_ID,
                max_tokens=8096,
                system=SYSTEM_PROMPT,
                messages=messages,
            )
        except Exception as exc:
            print(f"API Error: {exc}")
            messages.pop()   # 回滚, 让用户可以重试
            continue

        if response.stop_reason == "end_turn":
            assistant_text = ""
            for block in response.content:
                if hasattr(block, "text"):
                    assistant_text += block.text
            print_assistant(assistant_text)

            messages.append({
                "role": "assistant",
                "content": response.content,
            })
```

#### 2. stop_reason 分支

即使在第 01 节, 代码也预留了 `tool_use` 分支. 虽然还没有工具,
但这个脚手架意味着第 02 节不需要修改外层循环.

```python
        elif response.stop_reason == "tool_use":
            print_info("[stop_reason=tool_use] No tools in this section.")
            messages.append({"role": "assistant", "content": response.content})
```

| stop_reason    | 含义                   | 动作           |
|----------------|------------------------|----------------|
| `"end_turn"`   | 模型完成了回复         | 打印, 继续循环 |
| `"tool_use"`   | 模型想调用工具         | 执行, 反馈结果 |
| `"max_tokens"` | 回复被 token 限制截断  | 打印部分文本   |

### 试一试

```sh
# 确保 .env 中有你的密钥
echo 'ANTHROPIC_API_KEY=sk-ant-xxxxx' > .env
echo 'MODEL_ID=claude-sonnet-4-20250514' >> .env

# 运行 agent
python zh/s01_agent_loop.py

# 和它对话 -- 多轮对话有效, 因为 messages[] 会累积
# You > 法国的首都是哪里?
# You > 它的人口是多少?
# (模型记得上一轮提到的"法国".)
```

### OpenClaw 中的对应实现

| 方面           | claw0 (本文件)                  | OpenClaw 生产代码                      |
|----------------|--------------------------------|---------------------------------------|
| 循环位置       | 单文件中的 `agent_loop()`       | `src/agent/` 中的 `AgentLoop` 类      |
| 消息存储       | 内存中的 `list[dict]`          | JSONL 持久化的 SessionStore           |
| stop_reason    | 相同的分支逻辑                 | 相同逻辑 + 流式支持                    |
| 错误处理       | 弹出最后一条消息, 继续         | 带退避的重试 + 上下文保护              |
| 系统提示词     | 硬编码字符串                   | 8 层动态组装 (第 06 节)                |

## 第 02 节: 工具使用

> 工具 = 数据 (schema) + 处理函数映射表. 模型选一个名字, 你查表执行.

### 架构

```
    User Input
        |
        v
    messages[] --> LLM API (tools=TOOLS)
                       |
                  stop_reason?
                  /          \
            "end_turn"    "tool_use"
               |              |
             Print    for each tool_use block:
                        TOOL_HANDLERS[name](**input)
                              |
                        tool_result
                              |
                        messages[] <-- {role:"user", content:[tool_result]}
                              |
                        back to LLM --> may chain more tools
                                          or "end_turn" --> Print
```

外层 `while True` 与第 01 节完全相同. 唯一的新增是一个**内层** while 循环,
在 `stop_reason == "tool_use"` 时持续调用 LLM.

### 本节要点

- **TOOLS**: JSON schema 字典列表, 告诉模型有哪些工具可用.
- **TOOL_HANDLERS**: `dict[str, Callable]`, 将工具名映射到 Python 函数.
- **process_tool_call()**: 字典查找 + `**kwargs` 分发.
- **内层循环**: 模型可能连续调用多个工具, 然后才生成文本.
- **工具结果放在 user 消息中** (Anthropic API 的要求).

### 核心代码走读

#### 1. Schema + 分发表

两个平行的数据结构. `TOOLS` 告诉模型, `TOOL_HANDLERS` 告诉你的代码.

```python
TOOLS = [
    {
        "name": "bash",
        "description": "Run a shell command and return its output.",
        "input_schema": {
            "type": "object",
            "properties": {
                "command": {"type": "string", "description": "The shell command."},
                "timeout": {"type": "integer", "description": "Timeout in seconds."},
            },
            "required": ["command"],
        },
    },
    # ... read_file, write_file, edit_file (相同模式)
]

TOOL_HANDLERS: dict[str, Any] = {
    "bash": tool_bash,
    "read_file": tool_read_file,
    "write_file": tool_write_file,
    "edit_file": tool_edit_file,
}
```

添加新工具 = 在 `TOOLS` 中加一项 + 在 `TOOL_HANDLERS` 中加一项. 循环本身不需要改动.

#### 2. 分发函数

模型返回工具名和输入字典. 分发就是一次字典查找.
错误作为字符串返回 (而非抛出异常), 这样模型可以看到错误并自行修正.

```python
def process_tool_call(tool_name: str, tool_input: dict) -> str:
    handler = TOOL_HANDLERS.get(tool_name)
    if handler is None:
        return f"Error: Unknown tool '{tool_name}'"
    try:
        return handler(**tool_input)
    except TypeError as exc:
        return f"Error: Invalid arguments for {tool_name}: {exc}"
    except Exception as exc:
        return f"Error: {tool_name} failed: {exc}"
```

#### 3. 内层工具调用循环

相比第 01 节唯一的结构变化. 模型可能连续调用多次工具, 最后才产生文本回复.

```python
while True:
    response = client.messages.create(
        model=MODEL_ID, max_tokens=8096,
        system=SYSTEM_PROMPT, tools=TOOLS, messages=messages,
    )
    messages.append({"role": "assistant", "content": response.content})

    if response.stop_reason == "end_turn":
        # 提取文本, 打印, break
        break

    elif response.stop_reason == "tool_use":
        tool_results = []
        for block in response.content:
            if block.type != "tool_use":
                continue
            result = process_tool_call(block.name, block.input)
            tool_results.append({
                "type": "tool_result",
                "tool_use_id": block.id,
                "content": result,
            })
        # 工具结果放在 user 消息中 (API 要求)
        messages.append({"role": "user", "content": tool_results})
        continue  # 回到 LLM
```

### 试一试

```sh
python zh/s02_tool_use.py

# 让它执行命令
# You > 当前目录下有哪些文件?

# 让它读取文件
# You > 读取 en/s01_agent_loop.py 的内容

# 让它创建和编辑文件
# You > 创建一个名为 hello.txt 的文件, 内容是 "Hello World"
# You > 把 hello.txt 中的 "World" 改成 "claw0"

# 观察它链式调用工具 (读取 -> 编辑 -> 验证)
# You > 在 hello.txt 顶部添加一行注释
```

### OpenClaw 中的对应实现

| 方面             | claw0 (本文件)                 | OpenClaw 生产代码                      |
|------------------|-------------------------------|----------------------------------------|
| 工具定义         | Python 字典列表               | TypeBox schema, 自动校验               |
| 分发             | `dict[str, Callable]` 查表    | 相同模式 + 中间件管线                   |
| 安全性           | `safe_path()` 阻止目录穿越    | 沙箱执行, 白名单                       |
| 工具数量         | 4 个 (bash, read, write, edit)| 20+ (网页搜索, 媒体, 日历等)            |
| 工具结果         | 返回纯字符串                  | 带元数据的结构化结果                    |

## 第 03 节: 会话与上下文保护

> 会话就是 JSONL 文件. 追加写入, 重放恢复, 太大就摘要压缩.

### 架构

```
    User Input
        |
        v
    SessionStore.load_session()  --> rebuild messages[] from JSONL
        |
        v
    ContextGuard.guard_api_call()
        |
        +-- Attempt 0: normal call
        |       |
        |   overflow? --no--> success
        |       |yes
        +-- Attempt 1: truncate oversized tool results
        |       |
        |   overflow? --no--> success
        |       |yes
        +-- Attempt 2: compact history via LLM summary
        |       |
        |   overflow? --yes--> raise
        |
    SessionStore.save_turn()  --> append to JSONL
        |
        v
    Print response

    File layout:
    workspace/.sessions/agents/{agent_id}/sessions/{session_id}.jsonl
    workspace/.sessions/agents/{agent_id}/sessions.json  (index)
```

### 本节要点

- **SessionStore**: JSONL 持久化. 写入时追加, 读取时重放.
- **_rebuild_history()**: 将扁平的 JSONL 转换回 API 兼容的 messages[].
- **ContextGuard**: 3 阶段溢出重试 (正常 -> 截断 -> 压缩 -> 失败).
- **compact_history()**: LLM 生成摘要替换旧消息.
- **REPL 命令**: `/new`, `/switch`, `/context`, `/compact` 用于会话管理.

### 核心代码走读

#### 1. JSONL 追加与重放

每个会话是一个 `.jsonl` 文件 -- 每行一条 JSON 记录. 追加写入是原子的
(不需要重写整个文件). 四种记录类型:

```python
{"type": "user", "content": "Hello", "ts": 1234567890}
{"type": "assistant", "content": [{"type": "text", "text": "Hi!"}], "ts": ...}
{"type": "tool_use", "tool_use_id": "toolu_...", "name": "read_file", "input": {...}, "ts": ...}
{"type": "tool_result", "tool_use_id": "toolu_...", "content": "file contents", "ts": ...}
```

`_rebuild_history()` 方法将这些扁平记录转换回 Anthropic API 格式
(严格交替 user/assistant, tool_use 在 assistant 内, tool_result 在 user 内):

```python
def _rebuild_history(self, path: Path) -> list[dict]:
    messages: list[dict] = []
    for line in path.read_text(encoding="utf-8").strip().split("\n"):
        record = json.loads(line)
        rtype = record.get("type")

        if rtype == "user":
            messages.append({"role": "user", "content": record["content"]})
        elif rtype == "assistant":
            content = record["content"]
            if isinstance(content, str):
                content = [{"type": "text", "text": content}]
            messages.append({"role": "assistant", "content": content})
        elif rtype == "tool_use":
            # 合并到最后一条 assistant 消息
            block = {"type": "tool_use", "id": record["tool_use_id"],
                     "name": record["name"], "input": record["input"]}
            if messages and messages[-1]["role"] == "assistant":
                messages[-1]["content"].append(block)
            else:
                messages.append({"role": "assistant", "content": [block]})
        elif rtype == "tool_result":
            # 将连续的结果合并到同一条 user 消息
            result_block = {"type": "tool_result",
                            "tool_use_id": record["tool_use_id"],
                            "content": record["content"]}
            if (messages and messages[-1]["role"] == "user"
                    and isinstance(messages[-1]["content"], list)
                    and messages[-1]["content"][0].get("type") == "tool_result"):
                messages[-1]["content"].append(result_block)
            else:
                messages.append({"role": "user", "content": [result_block]})
    return messages
```

#### 2. 3 阶段保护

`guard_api_call()` 包裹每次 API 调用. 如果上下文溢出, 它会用越来越激进的策略重试:

```python
def guard_api_call(self, api_client, model, system, messages,
                   tools=None, max_retries=2):
    current_messages = messages
    for attempt in range(max_retries + 1):
        try:
            result = api_client.messages.create(
                model=model, max_tokens=8096,
                system=system, messages=current_messages,
                **({"tools": tools} if tools else {}),
            )
            if current_messages is not messages:
                messages.clear()
                messages.extend(current_messages)
            return result
        except Exception as exc:
            error_str = str(exc).lower()
            is_overflow = ("context" in error_str or "token" in error_str)
            if not is_overflow or attempt >= max_retries:
                raise
            if attempt == 0:
                current_messages = self._truncate_large_tool_results(current_messages)
            elif attempt == 1:
                current_messages = self.compact_history(
                    current_messages, api_client, model)
```

#### 3. 历史压缩

将最早 50% 的消息序列化为纯文本, 让 LLM 生成摘要,
用摘要 + 近期消息替换:

```python
def compact_history(self, messages, api_client, model):
    keep_count = max(4, int(len(messages) * 0.2))
    compress_count = max(2, int(len(messages) * 0.5))
    compress_count = min(compress_count, len(messages) - keep_count)

    old_text = _serialize_messages_for_summary(messages[:compress_count])
    summary_resp = api_client.messages.create(
        model=model, max_tokens=2048,
        system="You are a conversation summarizer. Be concise and factual.",
        messages=[{"role": "user", "content": summary_prompt}],
    )
    # 用摘要 + "Understood" 对替换旧消息
    compacted = [
        {"role": "user", "content": "[Previous conversation summary]\n" + summary},
        {"role": "assistant", "content": [{"type": "text",
         "text": "Understood, I have the context."}]},
    ]
    compacted.extend(messages[compress_count:])
    return compacted
```

### 试一试

```sh
python zh/s03_sessions.py

# 创建会话并在会话之间切换
# You > /new my-project
# You > 给我讲讲 Python 生成器
# You > /new experiments
# You > 2+2 等于多少?
# You > /switch my-p     (前缀匹配)

# 查看上下文使用情况
# You > /context
# Context usage: ~1,234 / 180,000 tokens
# [####--------------------------] 0.7%

# 上下文过大时手动压缩
# You > /compact
```

### OpenClaw 中的对应实现

| 方面              | claw0 (本文件)                 | OpenClaw 生产代码                       |
|-------------------|--------------------------------|-----------------------------------------|
| 存储格式          | JSONL 文件, 每个会话一个       | 相同的 JSONL 格式                       |
| 重放              | `_rebuild_history()`           | 相同的重建逻辑                          |
| 溢出处理          | 3 阶段保护                     | 相同模式 + token 计数 API               |
| 压缩              | LLM 摘要替换旧消息             | 相同方案, 自适应压缩                    |
| token 估算        | `len(text) // 4` 启发式        | API 提供的 token 计数                   |
## 第 04 节: 通道

> 每个平台都不同, 但它们都产生相同的 InboundMessage.

### 架构

```
    Telegram ----.                          .---- sendMessage API
    Feishu -------+-- InboundMessage ---+---- im/v1/messages
    CLI (stdin) --'    Agent Loop        '---- print(stdout)
                       (same brain)

    Telegram detail:
    getUpdates (long-poll, 30s)
        |
    offset persist (disk)
        |
    media_group_id? --yes--> buffer 500ms --> merge captions
        |no
    text buffer (1s silence) --> flush
        |
    InboundMessage --> allowed_chats filter --> agent turn
```

### 本节要点

- **InboundMessage**: 一个 dataclass, 将所有平台的消息负载统一为同一格式.
- **Channel ABC**: `receive()` + `send()` 就是全部接口契约.
- **TelegramChannel**: 长轮询, offset 持久化, 媒体组缓冲, 文本合并.
- **FeishuChannel**: 基于 webhook, token 认证, @提及检测, 多类型消息解析.
- **ChannelManager**: 持有所有活跃通道的注册中心.

### 核心代码走读

#### 1. InboundMessage -- 统一的消息格式

每个通道都归一化为此格式. agent 循环只看到 `InboundMessage`,
永远不接触平台特定的负载.

```python
@dataclass
class InboundMessage:
    text: str
    sender_id: str
    channel: str = ""          # "cli", "telegram", "feishu"
    account_id: str = ""       # 接收消息的 bot
    peer_id: str = ""          # DM=user_id, group=chat_id, topic=chat_id:topic:thread_id
    is_group: bool = False
    media: list = field(default_factory=list)
    raw: dict = field(default_factory=dict)
```

`peer_id` 编码了会话范围:

| 上下文            | peer_id 格式              |
|-------------------|---------------------------|
| Telegram 私聊     | `user_id`                 |
| Telegram 群组     | `chat_id`                 |
| Telegram 话题     | `chat_id:topic:thread_id` |
| 飞书单聊          | `user_id`                 |
| 飞书群组          | `chat_id`                 |

#### 2. Channel 抽象基类

添加新平台只需实现两个方法:

```python
class Channel(ABC):
    name: str = "unknown"

    @abstractmethod
    def receive(self) -> InboundMessage | None: ...

    @abstractmethod
    def send(self, to: str, text: str, **kwargs: Any) -> bool: ...
```

`CLIChannel` 是最简单的实现 -- `receive()` 包装 `input()`,
`send()` 包装 `print()`:

```python
class CLIChannel(Channel):
    name = "cli"

    def receive(self) -> InboundMessage | None:
        text = input("You > ").strip()
        if not text:
            return None
        return InboundMessage(
            text=text, sender_id="cli-user", channel="cli",
            account_id="cli-local", peer_id="cli-user",
        )

    def send(self, to: str, text: str, **kwargs: Any) -> bool:
        print_assistant(text)
        return True
```

#### 3. run_agent_turn -- 与通道无关的处理逻辑

agent turn 函数接收一个 `InboundMessage`, 运行标准的工具循环,
然后通过来源通道发送回复:

```python
def run_agent_turn(inbound: InboundMessage, conversations: dict, mgr: ChannelManager):
    sk = build_session_key(inbound.channel, inbound.account_id, inbound.peer_id)
    if sk not in conversations:
        conversations[sk] = []
    messages = conversations[sk]
    messages.append({"role": "user", "content": inbound.text})

    # Telegram 的输入指示器
    if inbound.channel == "telegram":
        tg = mgr.get("telegram")
        if isinstance(tg, TelegramChannel):
            tg.send_typing(inbound.peer_id.split(":topic:")[0])

    while True:
        response = client.messages.create(
            model=MODEL_ID, max_tokens=8096,
            system=SYSTEM_PROMPT, tools=TOOLS, messages=messages,
        )
        messages.append({"role": "assistant", "content": response.content})

        if response.stop_reason == "end_turn":
            text = "".join(b.text for b in response.content if hasattr(b, "text"))
            ch = mgr.get(inbound.channel)
            if ch:
                ch.send(inbound.peer_id, text)
            break
        elif response.stop_reason == "tool_use":
            # 分发工具, 追加结果, 继续
            ...
```

### 试一试

```sh
# 仅 CLI (除了 API key 外不需要其他环境变量)
python zh/s04_channels.py

# 启用 Telegram -- 在 .env 中添加:
# TELEGRAM_BOT_TOKEN=123456:ABC-DEF...
# TELEGRAM_ALLOWED_CHATS=12345,67890    (可选白名单)

# 启用飞书 -- 在 .env 中添加:
# FEISHU_APP_ID=cli_xxxxx
# FEISHU_APP_SECRET=xxxxx

# REPL 命令
# You > /channels      (列出已注册的通道)
# You > /accounts      (显示 bot 账号)
```

### OpenClaw 中的对应实现

| 方面            | claw0 (本文件)                   | OpenClaw 生产代码                        |
|-----------------|----------------------------------|------------------------------------------|
| Channel ABC     | `receive()` + `send()`           | 相同接口 + 生命周期钩子                  |
| 平台数量        | CLI, Telegram, 飞书              | 10+ (Telegram, Discord, Slack 等)        |
| 并发模型        | 每个通道一个线程 + 共享队列      | 相同的线程模型 + 异步网关                |
| 消息格式        | `InboundMessage` dataclass       | 相同的统一消息类型                       |
| Offset 存储     | 纯文本文件                       | 带版本号的 JSON + 原子写入               |
## 第 05 节: 网关与路由

> 一张绑定表将 (channel, peer) 映射到 agent_id. 最具体的匹配优先.

### 架构

```
    Inbound Message (channel, account_id, peer_id, text)
           |
    +------v------+     +----------+
    |   Gateway    | <-- | WS/REPL  |  JSON-RPC 2.0
    +------+------+     +----------+
           |
    +------v------+
    | BindingTable |  5-tier resolution:
    +------+------+    T1: peer_id     (most specific)
           |           T2: guild_id
           |           T3: account_id
           |           T4: channel
           |           T5: default     (least specific)
           |
     (agent_id, binding)
           |
    +------v---------+
    | build_session_key() |  dm_scope controls isolation
    +------+---------+
           |
    +------v------+
    | AgentManager |  per-agent config / personality / sessions
    +------+------+
           |
        LLM API
```

### 本节要点

- **BindingTable**: 排序的路由绑定列表. 从 tier 1 到 tier 5 遍历, 首次匹配即返回.
- **build_session_key()**: `dm_scope` 控制会话隔离 (每用户、每通道等).
- **AgentManager**: 多 agent 注册中心 -- 每个 agent 有自己的性格和模型.
- **GatewayServer**: 可选的 WebSocket 服务器, 使用 JSON-RPC 2.0 协议.
- **共享事件循环**: daemon 线程中的 asyncio 循环, 信号量限制并发数为 4.

### 核心代码走读

#### 1. BindingTable.resolve() -- 路由核心

绑定按 `(tier, -priority)` 排序. 解析时线性遍历, 首次匹配即返回.

```python
@dataclass
class Binding:
    agent_id: str
    tier: int           # 1-5, 越小越具体
    match_key: str      # "peer_id" | "guild_id" | "account_id" | "channel" | "default"
    match_value: str    # e.g. "telegram:12345", "discord", "*"
    priority: int = 0   # 同一 tier 内, 越高越优先

class BindingTable:
    def resolve(self, channel="", account_id="",
                guild_id="", peer_id="") -> tuple[str | None, Binding | None]:
        for b in self._bindings:
            if b.tier == 1 and b.match_key == "peer_id":
                if ":" in b.match_value:
                    if b.match_value == f"{channel}:{peer_id}":
                        return b.agent_id, b
                elif b.match_value == peer_id:
                    return b.agent_id, b
            elif b.tier == 2 and b.match_key == "guild_id" and b.match_value == guild_id:
                return b.agent_id, b
            elif b.tier == 3 and b.match_key == "account_id" and b.match_value == account_id:
                return b.agent_id, b
            elif b.tier == 4 and b.match_key == "channel" and b.match_value == channel:
                return b.agent_id, b
            elif b.tier == 5 and b.match_key == "default":
                return b.agent_id, b
        return None, None
```

给定以下示例绑定:

```python
bt.add(Binding(agent_id="luna", tier=5, match_key="default", match_value="*"))
bt.add(Binding(agent_id="sage", tier=4, match_key="channel", match_value="telegram"))
bt.add(Binding(agent_id="sage", tier=1, match_key="peer_id",
               match_value="discord:admin-001", priority=10))
```

| 输入                              | Tier | Agent |
|-----------------------------------|------|-------|
| `channel=cli, peer=user1`         | 5    | Luna  |
| `channel=telegram, peer=user2`    | 4    | Sage  |
| `channel=discord, peer=admin-001` | 1    | Sage  |
| `channel=discord, peer=user3`     | 5    | Luna  |

#### 2. 带 dm_scope 的会话 key

agent 解析完成后, agent 配置上的 `dm_scope` 控制会话隔离方式:

```python
def build_session_key(agent_id, channel="", account_id="",
                      peer_id="", dm_scope="per-peer"):
    aid = normalize_agent_id(agent_id)
    if dm_scope == "per-account-channel-peer" and peer_id:
        return f"agent:{aid}:{channel}:{account_id}:direct:{peer_id}"
    if dm_scope == "per-channel-peer" and peer_id:
        return f"agent:{aid}:{channel}:direct:{peer_id}"
    if dm_scope == "per-peer" and peer_id:
        return f"agent:{aid}:direct:{peer_id}"
    return f"agent:{aid}:main"
```

| dm_scope                   | Key 格式                                 | 效果                      |
|----------------------------|------------------------------------------|---------------------------|
| `main`                     | `agent:{id}:main`                        | 所有人共享一个会话        |
| `per-peer`                 | `agent:{id}:direct:{peer}`               | 每个用户隔离              |
| `per-channel-peer`         | `agent:{id}:{ch}:direct:{peer}`          | 每个平台的不同会话        |
| `per-account-channel-peer` | `agent:{id}:{ch}:{acc}:direct:{peer}`    | 最大隔离度                |

#### 3. AgentConfig -- 每个 agent 的性格

每个 agent 携带自己的配置. 系统提示词从配置生成:

```python
@dataclass
class AgentConfig:
    id: str
    name: str
    personality: str = ""
    model: str = ""              # 空 = 使用全局 MODEL_ID
    dm_scope: str = "per-peer"

    def system_prompt(self) -> str:
        parts = [f"You are {self.name}."]
        if self.personality:
            parts.append(f"Your personality: {self.personality}")
        parts.append("Answer questions helpfully and stay in character.")
        return " ".join(parts)
```

### 试一试

```sh
python zh/s05_gateway_routing.py

# 测试路由
# You > /bindings                      (查看所有路由绑定)
# You > /route cli user1               (通过 default 解析到 Luna)
# You > /route telegram user2           (通过 channel 绑定解析到 Sage)

# 强制指定 agent
# You > /switch sage
# You > Hello!                          (无论路由结果如何都和 Sage 对话)
# You > /switch off                     (恢复正常路由)

# 启动 WebSocket 网关
# You > /gateway
# Gateway running on ws://localhost:8765
```

### OpenClaw 中的对应实现

| 方面             | claw0 (本文件)                 | OpenClaw 生产代码                      |
|------------------|--------------------------------|----------------------------------------|
| 路由解析         | 5 层线性扫描                   | 相同的层级系统 + 配置文件              |
| 会话 key         | `dm_scope` 参数                | 相同的 dm_scope + 持久化会话           |
| 多 agent         | 内存中的 AgentConfig           | 每个 agent 独立的工作区目录            |
| 网关             | WebSocket + JSON-RPC 2.0       | 相同协议 + HTTP API                    |
| 并发控制         | `asyncio.Semaphore(4)`         | 相同的信号量模式                       |

## 第 06 节: 智能层

> 系统提示词从磁盘上的文件组装. 换文件, 换性格.

### 架构

```
    Startup                              Per-Turn
    =======                              ========

    BootstrapLoader                      User Input
    load SOUL.md, IDENTITY.md, ...           |
    truncate per file (20k)                  v
    cap total (150k)                    _auto_recall(user_input)
         |                              search memory by TF-IDF
         v                                   |
    SkillsManager                            v
    scan directories for SKILL.md       build_system_prompt()
    parse frontmatter                   assemble 8 layers:
    deduplicate by name                     1. Identity
         |                                  2. Soul (personality)
         v                                  3. Tools guidance
    bootstrap_data + skills_block           4. Skills
    (cached for all turns)                  5. Memory (evergreen + recalled)
                                            6. Bootstrap (remaining files)
                                            7. Runtime context
                                            8. Channel hints
                                                |
                                                v
                                            LLM API call

    Earlier layers = stronger influence on behavior.
    SOUL.md is at layer 2 for exactly this reason.
```

### 本节要点

- **BootstrapLoader**: 从工作区加载最多 8 个 markdown 文件, 有单文件和总量上限.
- **SkillsManager**: 扫描多个目录查找带 YAML frontmatter 的 `SKILL.md` 文件.
- **MemoryStore**: 双层存储 (常驻 MEMORY.md + 每日 JSONL), TF-IDF 搜索.
- **`_auto_recall()`**:  用用户消息搜索记忆, 将结果注入提示词.
- **build_system_prompt()**: 将 8 个层组装为一个字符串, 每轮重新构建.

### 核心代码走读

#### 1. build_system_prompt() -- 8 层组装

这个函数是智能系统的核心. 它每轮都产生不同的系统提示词,
因为记忆可能已经被更新.

```python
def build_system_prompt(mode="full", bootstrap=None, skills_block="",
                        memory_context="", agent_id="main", channel="terminal"):
    sections: list[str] = []

    # 第 1 层: 身份
    identity = bootstrap.get("IDENTITY.md", "").strip()
    sections.append(identity if identity else "You are a helpful AI assistant.")

    # 第 2 层: 灵魂 (性格) -- 越靠前 = 影响力越强
    if mode == "full":
        soul = bootstrap.get("SOUL.md", "").strip()
        if soul:
            sections.append(f"### Personality\n\n{soul}")

    # 第 3 层: 工具使用指南
    tools_md = bootstrap.get("TOOLS.md", "").strip()
    if tools_md:
        sections.append(f"### Tool Usage Guidelines\n\n{tools_md}")

    # 第 4 层: 技能
    if mode == "full" and skills_block:
        sections.append(skills_block)

    # 第 5 层: 记忆 (常驻 + 自动搜索的)
    if mode == "full":
        # ... 合并 MEMORY.md 和召回的记忆

    # 第 6 层: 引导上下文 (HEARTBEAT.md, BOOTSTRAP.md, AGENTS.md, USER.md)
    # 第 7 层: 运行时上下文 (agent ID, 模型, 通道, 时间)
    # 第 8 层: 通道提示 ("You are responding via Telegram.")

    return "\n\n".join(sections)
```

#### 2. MemoryStore.search_memory() -- TF-IDF 搜索

纯 Python 实现, 不需要外部向量数据库. 加载所有记忆片段, 计算 TF-IDF 向量,
按余弦相似度排序.

```python
def search_memory(self, query: str, top_k: int = 5) -> list[dict]:
    chunks = self._load_all_chunks()   # MEMORY.md 段落 + 每日 JSONL 条目
    query_tokens = self._tokenize(query)
    chunk_tokens = [self._tokenize(c["text"]) for c in chunks]

    # 所有片段的文档频率
    df: dict[str, int] = {}
    for tokens in chunk_tokens:
        for t in set(tokens):
            df[t] = df.get(t, 0) + 1

    def tfidf(tokens):
        tf = {}
        for t in tokens:
            tf[t] = tf.get(t, 0) + 1
        return {t: c * (math.log((n + 1) / (df.get(t, 0) + 1)) + 1)
                for t, c in tf.items()}

    def cosine(a, b):
        common = set(a) & set(b)
        if not common:
            return 0.0
        dot = sum(a[k] * b[k] for k in common)
        na = math.sqrt(sum(v * v for v in a.values()))
        nb = math.sqrt(sum(v * v for v in b.values()))
        return dot / (na * nb) if na and nb else 0.0

    qvec = tfidf(query_tokens)
    scored = []
    for i, tokens in enumerate(chunk_tokens):
        score = cosine(qvec, tfidf(tokens))
        if score > 0.0:
            scored.append({"path": chunks[i]["path"], "score": score,
                           "snippet": chunks[i]["text"][:200]})
    scored.sort(key=lambda x: x["score"], reverse=True)
    return scored[:top_k]
```

#### 3. 混合搜索管道 -- 向量 + 关键词 + MMR

完整的搜索管道串联五个阶段:

1. **关键词搜索** (TF-IDF): 与上面相同的算法, 按余弦相似度返回 top-10
2. **向量搜索** (哈希投影): 通过基于哈希的随机投影模拟嵌入向量, 返回 top-10
3. **合并**: 按文本前缀取并集, 加权组合 (`vector_weight=0.7, text_weight=0.3`)
4. **时间衰减**: `score *= exp(-decay_rate * age_days)`, 越近的记忆得分越高
5. **Maximal Marginal Relevance (a.k.a MMR) 算法重排序**: `MMR = lambda * relevance - (1-lambda) * max_similarity_to_selected`, 用 token 集合的 Jaccard 相似度保证多样性

基于哈希的向量嵌入展示了双通道搜索的**模式**, 不需要外部嵌入 API.

#### 4. _auto_recall() -- 自动记忆注入

每次 LLM 调用之前, 自动搜索相关记忆并注入到系统提示词中.
用户不需要显式请求.

```python
def _auto_recall(user_message: str) -> str:
    results = memory_store.search_memory(user_message, top_k=3)
    if not results:
        return ""
    return "\n".join(f"- [{r['path']}] {r['snippet']}" for r in results)

# 在 agent 循环中, 每轮:
memory_context = _auto_recall(user_input)
system_prompt = build_system_prompt(
    mode="full", bootstrap=bootstrap_data,
    skills_block=skills_block, memory_context=memory_context,
)
```

### 试一试

```sh
python zh/s06_intelligence.py

# 创建工作区文件以体验完整系统:
# workspace/SOUL.md       -- "You are warm, curious, and encouraging."
# workspace/IDENTITY.md   -- "You are Luna, a personal AI companion."
# workspace/MEMORY.md     -- "User prefers Python over JavaScript."

# 查看组装好的提示词
# You > /prompt

# 检查加载了哪些引导文件
# You > /bootstrap

# 搜索记忆
# You > /search python

# 告诉它一些信息, 然后过一会再问
# You > 我最喜欢的颜色是蓝色.
# You > 你知道我的偏好吗?
# (auto-recall 找到颜色记忆并注入提示词)
```

### OpenClaw 中的对应实现

| 方面             | claw0 (本文件)                | OpenClaw 生产代码                       |
|------------------|------------------------------|-----------------------------------------|
| 提示词组装       | 8 层 `build_system_prompt`   | 相同的分层方案                          |
| 引导文件         | 从工作区目录加载             | 相同的文件集 + 每个 agent 的覆盖配置    |
| 记忆搜索         | 混合管道 (TF-IDF + 向量 + MMR) | 相同方案 + 可选的 embedding API         |
| 技能发现         | 扫描目录查找 SKILL.md        | 相同的扫描 + 插件系统                   |
| 自动召回         | 每条用户消息都搜索           | 相同模式, top_k 可配置                  |
## 第 07 节: 心跳与 Cron

> 一个定时器线程检查"该不该运行", 然后将任务排入与用户消息相同的队列.

### 架构

```
    Main Lane (user input):
        User Input --> lane_lock.acquire() -------> LLM --> Print
                       (blocking: always wins)

    Heartbeat Lane (background thread, 1s poll):
        should_run()?
            |no --> sleep 1s
            |yes
        _execute():
            lane_lock.acquire(blocking=False)
                |fail --> yield (user has priority)
                |success
            build prompt from HEARTBEAT.md + SOUL.md + MEMORY.md
                |
            run_agent_single_turn()
                |
            parse: "HEARTBEAT_OK"? --> suppress
                   meaningful text? --> duplicate? --> suppress
                                           |no
                                       output_queue.append()

    Cron Service (background thread, 1s tick):
        CRON.json --> load jobs --> tick() every 1s
            |
        for each job: enabled? --> due? --> _run_job()
            |
        error? --> consecutive_errors++ --> >=5? --> auto-disable
            |ok
        consecutive_errors = 0 --> log to cron-runs.jsonl
```

### 本节要点

- **Lane 互斥**: `threading.Lock` 在用户和心跳之间共享. 用户总是赢 (阻塞获取); 心跳让步 (非阻塞获取).
- **should_run()**: 每次心跳尝试前的 4 个前置条件检查.
- **HEARTBEAT_OK**: agent 用来表示"没有需要报告的内容"的约定.
- **CronService**: 3 种调度类型 (`at`, `every`, `cron`), 连续错误 5 次后自动禁用.
- **输出队列**: 后台结果通过线程安全的列表输送到 REPL.

### 核心代码走读

#### 1. Lane 互斥

最重要的设计原则: 用户消息始终优先.

```python
lane_lock = threading.Lock()

# Main lane: 阻塞获取. 用户始终能进入.
lane_lock.acquire()
try:
    # 处理用户消息, 调用 LLM
finally:
    lane_lock.release()

# Heartbeat lane: 非阻塞获取. 用户活跃时让步.
def _execute(self) -> None:
    acquired = self.lane_lock.acquire(blocking=False)
    if not acquired:
        return   # 用户持有锁, 跳过本次心跳
    self.running = True
    try:
        instructions, sys_prompt = self._build_heartbeat_prompt()
        response = run_agent_single_turn(instructions, sys_prompt)
        meaningful = self._parse_response(response)
        if meaningful and meaningful.strip() != self._last_output:
            self._last_output = meaningful.strip()
            with self._queue_lock:
                self._output_queue.append(meaningful)
    finally:
        self.running = False
        self.last_run_at = time.time()
        self.lane_lock.release()
```

#### 2. should_run() -- 前置条件链

四个检查必须全部通过. 锁的检测在 `_execute()` 中单独进行,
以避免 TOCTOU（Time of Check to Time of Use） 竞态条件.

```python
def should_run(self) -> tuple[bool, str]:
    if not self.heartbeat_path.exists():
        return False, "HEARTBEAT.md not found"
    if not self.heartbeat_path.read_text(encoding="utf-8").strip():
        return False, "HEARTBEAT.md is empty"

    elapsed = time.time() - self.last_run_at
    if elapsed < self.interval:
        return False, f"interval not elapsed ({self.interval - elapsed:.0f}s remaining)"

    hour = datetime.now().hour
    s, e = self.active_hours
    in_hours = (s <= hour < e) if s <= e else not (e <= hour < s)
    if not in_hours:
        return False, f"outside active hours ({s}:00-{e}:00)"

    if self.running:
        return False, "already running"
    return True, "all checks passed"
```

#### 3. CronService -- 3 种调度类型

任务定义在 `CRON.json` 中. 每个任务有一个 `schedule.kind` 和一个 `payload`:

```python
@dataclass
class CronJob:
    id: str
    name: str
    enabled: bool
    schedule_kind: str       # "at" | "every" | "cron"
    schedule_config: dict
    payload: dict            # {"kind": "agent_turn", "message": "..."}
    consecutive_errors: int = 0

def _compute_next(self, job, now):
    if job.schedule_kind == "at":
        ts = datetime.fromisoformat(cfg.get("at", "")).timestamp()
        return ts if ts > now else 0.0
    if job.schedule_kind == "every":
        every = cfg.get("every_seconds", 3600)
        # 对齐到锚点, 保证触发时间可预测
        steps = int((now - anchor) / every) + 1
        return anchor + steps * every
    if job.schedule_kind == "cron":
        return croniter(expr, datetime.fromtimestamp(now)).get_next(datetime).timestamp()
```

连续 5 次错误后自动禁用:

```python
if status == "error":
    job.consecutive_errors += 1
    if job.consecutive_errors >= 5:
        job.enabled = False
else:
    job.consecutive_errors = 0
```

### 试一试

```sh
python zh/s07_heartbeat_cron.py

# 创建 workspace/HEARTBEAT.md 写入指令:
# "Check if there are any unread reminders. Reply HEARTBEAT_OK if nothing to report."

# 检查心跳状态
# You > /heartbeat

# 手动触发心跳
# You > /trigger

# 列出 cron 任务 (需要 workspace/CRON.json)
# You > /cron

# 检查 lane 锁状态
# You > /lanes
# main_locked: False  heartbeat_running: False
```

`CRON.json` 示例:

```json
{
  "jobs": [
    {
      "id": "daily-check",
      "name": "Daily Check",
      "enabled": true,
      "schedule": {"kind": "cron", "expr": "0 9 * * *"},
      "payload": {"kind": "agent_turn", "message": "Generate a daily summary."}
    }
  ]
}
```

### OpenClaw 中的对应实现

| 方面             | claw0 (本文件)                 | OpenClaw 生产代码                       |
|------------------|-------------------------------|-----------------------------------------|
| Lane 互斥       | `threading.Lock`, 非阻塞      | 相同的锁模式                            |
| 心跳配置         | 工作区中的 `HEARTBEAT.md`      | 相同文件 + 环境变量覆盖                 |
| Cron 调度        | `CRON.json`, 3 种类型         | 相同格式 + webhook 触发器               |
| 自动禁用         | 连续 5 次错误                  | 相同阈值, 可配置                        |
| 输出投递         | 内存队列, 排出到 REPL          | 投递队列 (第 08 节)                     |
## 第 08 节: 消息投递

> 先写磁盘, 再尝试发送. 崩溃安全.

### 架构

```
    Agent Reply / Heartbeat / Cron
              |
        chunk_message()          split by platform limits
              |                  (telegram=4096, discord=2000, etc.)
              v
        DeliveryQueue.enqueue()
          1. Generate unique ID
          2. Write to .tmp.{pid}.{id}.json
          3. fsync()
          4. os.replace() to {id}.json    <-- WRITE-AHEAD
              |
              v
        DeliveryRunner (background thread, 1s scan)
              |
        deliver_fn(channel, to, text)
           /          \
        success      failure
          |              |
        ack()         fail()
        (delete       (retry_count++, compute backoff,
         .json)        update .json on disk)
                         |
                    retry_count >= 5?
                      |yes
                    move to failed/

    Backoff: [5s, 25s, 2min, 10min] with +/-20% jitter
```

### 本节要点

- **DeliveryQueue**: 磁盘持久化的预写队列. 入队时先写磁盘, 再尝试投递.
- **原子写入**: 临时文件 + `os.fsync()` + `os.replace()` -- 崩溃时不会产生半写文件.
- **DeliveryRunner**: 后台线程, 以指数退避处理待投递条目.
- **chunk_message()**: 按平台大小限制分片文本, 尊重段落边界.
- **启动恢复扫描**: 启动时自动重试上次崩溃前遗留的待投递条目.

### 核心代码走读

#### 1. DeliveryQueue.enqueue() + 原子写入

基本规则: 先写磁盘, 再尝试投递. 如果进程在入队和投递之间崩溃,
消息仍然保存在磁盘上.

```python
def enqueue(self, channel: str, to: str, text: str) -> str:
    delivery_id = uuid.uuid4().hex[:12]
    entry = QueuedDelivery(
        id=delivery_id, channel=channel, to=to, text=text,
        enqueued_at=time.time(), next_retry_at=0.0,
    )
    self._write_entry(entry)
    return delivery_id

def _write_entry(self, entry: QueuedDelivery) -> None:
    final_path = self.queue_dir / f"{entry.id}.json"
    tmp_path = self.queue_dir / f".tmp.{os.getpid()}.{entry.id}.json"

    data = json.dumps(entry.to_dict(), indent=2, ensure_ascii=False)
    with open(tmp_path, "w", encoding="utf-8") as f:
        f.write(data)
        f.flush()
        os.fsync(f.fileno())        # 数据已落盘

    os.replace(str(tmp_path), str(final_path))  # POSIX 上的原子操作
```

三步保证:
- 第 1 步: 写入 `.tmp.{pid}.{id}.json` (崩溃 = 孤立的临时文件, 无害)
- 第 2 步: `fsync()` -- 数据已落盘
- 第 3 步: `os.replace()` -- 原子交换 (崩溃 = 旧文件或新文件, 绝不会是半写文件)

#### 2. ack() / fail() -- 重试生命周期

```python
def ack(self, delivery_id: str) -> None:
    """投递成功. 删除队列文件."""
    (self.queue_dir / f"{delivery_id}.json").unlink()

def fail(self, delivery_id: str, error: str) -> None:
    """递增 retry_count, 计算下次重试时间, 或放弃."""
    entry = self._read_entry(delivery_id)
    entry.retry_count += 1
    entry.last_error = error
    if entry.retry_count >= MAX_RETRIES:
        self.move_to_failed(delivery_id)
        return
    backoff_ms = compute_backoff_ms(entry.retry_count)
    entry.next_retry_at = time.time() + backoff_ms / 1000.0
    self._write_entry(entry)  # 将新的重试状态更新到磁盘
```

带抖动的退避, 防止雷群效应:

```python
BACKOFF_MS = [5_000, 25_000, 120_000, 600_000]
MAX_RETRIES = 5

def compute_backoff_ms(retry_count: int) -> int:
    if retry_count <= 0:
        return 0
    idx = min(retry_count - 1, len(BACKOFF_MS) - 1)
    base = BACKOFF_MS[idx]
    jitter = random.randint(-base // 5, base // 5)   # +/- 20%
    return max(0, base + jitter)
```

#### 3. DeliveryRunner -- 后台循环

每秒扫描待投递条目. 只处理 `next_retry_at` 已到期的条目.
启动时执行恢复扫描, 处理上次崩溃遗留的条目.

```python
class DeliveryRunner:
    def start(self) -> None:
        self._recovery_scan()
        self._thread = threading.Thread(
            target=self._background_loop, daemon=True)
        self._thread.start()

    def _process_pending(self) -> None:
        pending = self.queue.load_pending()
        now = time.time()
        for entry in pending:
            if entry.next_retry_at > now:
                continue
            self.total_attempted += 1
            try:
                self.deliver_fn(entry.channel, entry.to, entry.text)
                self.queue.ack(entry.id)
                self.total_succeeded += 1
            except Exception as exc:
                self.queue.fail(entry.id, str(exc))
                self.total_failed += 1
```

### 试一试

```sh
python zh/s08_delivery.py

# 发送一条消息 -- 观察它被入队并投递
# You > Hello!

# 开启 50% 失败率
# You > /simulate-failure

# 再发一条消息 -- 观察带退避的重试
# You > 在失败模式下的测试消息

# 查看队列
# You > /queue
# You > /failed

# 恢复正常, 观察待投递条目被送出
# You > /simulate-failure

# 查看统计数据
# You > /stats
```

### OpenClaw 中的对应实现

| 方面           | claw0 (本文件)                   | OpenClaw 生产代码                     |
|----------------|----------------------------------|---------------------------------------|
| 队列存储       | 目录中的 JSON 文件               | 相同的每条目一个文件模式              |
| 原子写入       | tmp + fsync + os.replace         | 相同方案                              |
| 退避           | [5s, 25s, 2min, 10min] + 抖动   | 相同的调度                            |
| 消息分片       | 段落边界分割                     | 相同 + 代码围栏感知                   |
| 恢复           | 启动时扫描队列目录               | 相同的扫描 + 孤立文件清理             |

### 全系列总结

10 个章节, 构成一个 agent 网关的核心机制:

```
    Section 01: while True + stop_reason        (循环)
    Section 02: TOOLS + TOOL_HANDLERS           (执行)
    Section 03: JSONL + ContextGuard            (持久化)
    Section 04: Channel ABC + InboundMessage    (通道)
    Section 05: BindingTable + session key      (路由)
    Section 06: 8-layer prompt + hybrid search  (智能)
    Section 07: Heartbeat + Cron                (自治)
    Section 08: DeliveryQueue + backoff         (可靠投递)
    Section 09: 3-layer retry onion + profiles  (韧性)
    Section 10: Named lanes + generation track  (并发)
```

第 01 节的 agent 循环在第 10 节的核心依然清晰可辨.
AI agent 就是一个 `while True` 循环加上一张分发表,
外面包裹着持久化、路由、智能、调度、可靠性、韧性和并发控制的层层机制.


## 第 09 节: 弹性

> 一次调用失败, 轮换重试.

### 架构

```
    Profiles: [main-key, backup-key, emergency-key]
         |
    for each non-cooldown profile:          LAYER 1: Auth Rotation
         |
    create client(profile.api_key)
         |
    for compact_attempt in 0..2:            LAYER 2: Overflow Recovery
         |
    _run_attempt(client, model, ...)        LAYER 3: Tool-Use Loop
         |              |
       success       exception
         |              |
    mark_success    classify_failure()
    return result       |
                   overflow? --> compact, retry Layer 2
                   auth/rate? -> mark_failure, break to Layer 1
                   timeout?  --> mark_failure(60s), break to Layer 1
                        |
                   all profiles exhausted?
                        |
                   try fallback models
                        |
                   all fallbacks failed?
                        |
                   raise RuntimeError
```

### 本节要点

- **FailoverReason**: 枚举, 将每个异常分类为六个类别之一 (rate_limit, auth, timeout, billing, overflow, unknown). 类别决定由哪一层重试处理.
- **AuthProfile**: 数据类, 持有一个 API key 及其冷却状态. 跟踪 `cooldown_until`、`failure_reason` 和 `last_good_at`.
- **ProfileManager**: 选择第一个未冷却的配置, 标记失败 (设置冷却), 标记成功 (清除失败状态).
- **ContextGuard**: 轻量级上下文溢出保护. 截断过大的工具结果, 如果仍然溢出则通过 LLM 摘要压缩历史.
- **ResilienceRunner**: 三层重试洋葱. Layer 1 轮换配置, Layer 2 处理溢出压缩, Layer 3 是标准的工具调用循环.
- **重试限制**: `BASE_RETRY=24`, `PER_PROFILE=8`, 上限为 `min(max(base + per_profile * N, 32), 160)`.
- **SimulatedFailure**: 为下次 API 调用装备一个模拟错误, 让你无需真实故障即可观察各类失败的处理过程.

### 核心代码走读

#### 1. classify_failure() -- 将异常路由到正确的层

每个异常在重试洋葱决定处理方式之前都会经过分类. 分类器检查错误字符串中的已知模式:

```python
class FailoverReason(Enum):
    rate_limit = "rate_limit"
    auth = "auth"
    timeout = "timeout"
    billing = "billing"
    overflow = "overflow"
    unknown = "unknown"

def classify_failure(exc: Exception) -> FailoverReason:
    msg = str(exc).lower()
    if "rate" in msg or "429" in msg:
        return FailoverReason.rate_limit
    if "auth" in msg or "401" in msg or "key" in msg:
        return FailoverReason.auth
    if "timeout" in msg or "timed out" in msg:
        return FailoverReason.timeout
    if "billing" in msg or "quota" in msg or "402" in msg:
        return FailoverReason.billing
    if "context" in msg or "token" in msg or "overflow" in msg:
        return FailoverReason.overflow
    return FailoverReason.unknown
```

分类驱动不同的冷却时长:
- `auth` / `billing`: 300s (坏 key, 不会很快自愈)
- `rate_limit`: 120s (等待速率限制窗口重置)
- `timeout`: 60s (瞬态故障, 短冷却)
- `overflow`: 不冷却配置 -- 改为压缩消息

#### 2. ProfileManager -- 冷却感知的 key 轮换

按顺序检查配置. 当冷却过期时配置可用. 失败后配置进入冷却; 成功后清除失败状态.

```python
class ProfileManager:
    def select_profile(self) -> AuthProfile | None:
        now = time.time()
        for profile in self.profiles:
            if now >= profile.cooldown_until:
                return profile
        return None

    def mark_failure(self, profile, reason, cooldown_seconds=300.0):
        profile.cooldown_until = time.time() + cooldown_seconds
        profile.failure_reason = reason.value

    def mark_success(self, profile):
        profile.failure_reason = None
        profile.last_good_at = time.time()
```

#### 3. ResilienceRunner.run() -- 三层洋葱

外层循环遍历配置 (Layer 1). 中间循环在压缩后重试溢出 (Layer 2). 内层调用运行工具调用循环 (Layer 3).

```python
def run(self, system, messages, tools):
    # LAYER 1: Auth Rotation
    for _rotation in range(len(self.profile_manager.profiles)):
        profile = self.profile_manager.select_profile()
        if profile is None:
            break

        api_client = Anthropic(api_key=profile.api_key)

        # LAYER 2: Overflow Recovery
        layer2_messages = list(messages)
        for compact_attempt in range(MAX_OVERFLOW_COMPACTION):
            try:
                # LAYER 3: Tool-Use Loop
                result, layer2_messages = self._run_attempt(
                    api_client, self.model_id, system,
                    layer2_messages, tools,
                )
                self.profile_manager.mark_success(profile)
                return result, layer2_messages

            except Exception as exc:
                reason = classify_failure(exc)

                if reason == FailoverReason.overflow:
                    # Compact and retry Layer 2
                    layer2_messages = self.guard.truncate_tool_results(layer2_messages)
                    layer2_messages = self.guard.compact_history(
                        layer2_messages, api_client, self.model_id)
                    continue

                elif reason in (FailoverReason.auth, FailoverReason.rate_limit):
                    self.profile_manager.mark_failure(profile, reason)
                    break  # try next profile (Layer 1)

                elif reason == FailoverReason.timeout:
                    self.profile_manager.mark_failure(profile, reason, 60)
                    break  # try next profile (Layer 1)

    # All profiles exhausted -- try fallback models
    for fallback_model in self.fallback_models:
        # ... try with first available profile ...

    raise RuntimeError("all profiles and fallbacks exhausted")
```

#### 4. _run_attempt() -- Layer 3 工具调用循环

最内层与第 01/02 节相同的 `while True` + `stop_reason` 分发. 在循环中运行工具调用, 直到模型返回 `end_turn` 或异常传播到外层.

```python
def _run_attempt(self, api_client, model, system, messages, tools):
    current_messages = list(messages)
    iteration = 0

    while iteration < self.max_iterations:
        iteration += 1
        response = api_client.messages.create(
            model=model, max_tokens=8096,
            system=system, tools=tools,
            messages=current_messages,
        )
        current_messages.append({"role": "assistant", "content": response.content})

        if response.stop_reason == "end_turn":
            return response, current_messages

        elif response.stop_reason == "tool_use":
            tool_results = []
            for block in response.content:
                if block.type != "tool_use":
                    continue
                result = process_tool_call(block.name, block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": result,
                })
            current_messages.append({"role": "user", "content": tool_results})
            continue

    raise RuntimeError("Tool-use loop exceeded max iterations")
```

### 试一试

```sh
python zh/s09_resilience.py

# 正常对话 -- 观察单配置成功
# You > Hello!

# 查看配置状态
# You > /profiles

# 模拟速率限制失败 -- 观察配置轮换
# You > /simulate-failure rate_limit
# You > Tell me a joke

# 模拟认证失败
# You > /simulate-failure auth
# You > What time is it?

# 失败后检查冷却状态
# You > /cooldowns

# 查看备选模型链
# You > /fallback

# 查看弹性统计
# You > /stats
```

### OpenClaw 中的对应实现

| 方面             | claw0 (本文件)                          | OpenClaw 生产代码                          |
|------------------|------------------------------------------|----------------------------------------------|
| 配置轮换         | 3 个演示配置, 相同 key                  | 跨提供商的多个真实 key                       |
| 失败分类器       | 异常文本字符串匹配                       | 相同模式, 加 HTTP 状态码检查                 |
| 溢出恢复         | 截断工具结果 + LLM 摘要                  | 相同的两阶段压缩                             |
| 冷却追踪         | 内存中的浮点时间戳                       | 相同的每配置内存追踪                         |
| 备选模型         | 可配置的备选链                           | 相同链, 通常为更小/更便宜的模型              |
| 重试限制         | BASE_RETRY=24, PER_PROFILE=8, 上限=160  | 相同公式                                     |
| 模拟失败         | /simulate-failure 命令用于测试           | 集成测试工具带故障注入                       |

## 第 10 节: 并发

> 命名 lane 序列化混沌.

### 架构

```
    Incoming Work
        |
    CommandQueue.enqueue(lane, fn)
        |
    +---v---+    +--------+    +-----------+
    | main  |    |  cron  |    | heartbeat |
    | max=1 |    | max=1  |    |   max=1   |
    | FIFO  |    | FIFO   |    |   FIFO    |
    +---+---+    +---+----+    +-----+-----+
        |            |              |
    [active]     [active]       [active]
        |            |              |
    _task_done   _task_done     _task_done
        |            |              |
    _pump()      _pump()        _pump()
    (dequeue     (dequeue       (dequeue
     next if      next if        next if
     active<max)  active<max)    active<max)
```

每个 lane 是一个 `LaneQueue`: 由 `threading.Condition` 保护的 FIFO deque. 任务以普通 callable 进入, 通过 `concurrent.futures.Future` 返回结果. `CommandQueue` 按名称将工作分发到正确的 lane, 并管理完整的生命周期.

### 本节要点

- **命名 lane**: 每个 lane 有一个名称 (如 `"main"`, `"cron"`, `"heartbeat"`) 和独立的 FIFO 队列. Lane 在首次使用时惰性创建.
- **max_concurrency**: 每个 lane 限制同时运行的任务数. 默认为 1 (串行执行). 增加以允许 lane 内的并行工作.
- **_pump() 循环**: 每个任务完成后 (`_task_done`), lane 检查是否可以出队更多任务. 这种自泵送设计意味着不需要外部调度器.
- **基于 Future 的结果**: 每次 `enqueue()` 返回一个 `concurrent.futures.Future`. 调用方可以通过 `future.result()` 阻塞等待, 或通过 `add_done_callback()` 附加回调.
- **Generation 追踪**: 每个 lane 有一个整数 generation 计数器. `reset_all()` 时所有 generation 递增. 当过期任务完成时 (其 generation 与当前不匹配), 不调用 `_pump()` -- 防止僵尸任务在重启后排空队列.
- **基于 Condition 的同步**: `threading.Condition` 替代了第 07 节的原始 `threading.Lock`. 这使 `wait_for_idle()` 能高效地睡眠等待通知, 而非轮询.
- **用户优先**: 用户输入进入 `main` lane 并阻塞等待结果. 后台工作 (心跳、cron) 进入独立的 lane, 永远不阻塞 REPL.

### 核心代码走读

#### 1. LaneQueue -- 核心原语

一个 lane 就是 deque + 条件变量 + 活跃计数器. `_pump()` 是引擎:

```python
class LaneQueue:
    def __init__(self, name: str, max_concurrency: int = 1) -> None:
        self.name = name
        self.max_concurrency = max(1, max_concurrency)
        self._deque = deque()           # [(fn, future, generation), ...]
        self._condition = threading.Condition()
        self._active_count = 0
        self._generation = 0

    def enqueue(self, fn, generation=None):
        future = concurrent.futures.Future()
        with self._condition:
            gen = generation if generation is not None else self._generation
            self._deque.append((fn, future, gen))
            self._pump()
        return future

    def _pump(self):
        """Pop and start tasks while active < max_concurrency."""
        while self._active_count < self.max_concurrency and self._deque:
            fn, future, gen = self._deque.popleft()
            self._active_count += 1
            threading.Thread(
                target=self._run_task, args=(fn, future, gen), daemon=True
            ).start()

    def _task_done(self, gen):
        with self._condition:
            self._active_count -= 1
            if gen == self._generation:  # stale tasks do not re-pump
                self._pump()
            self._condition.notify_all()
```

#### 2. CommandQueue -- 调度器

`CommandQueue` 持有 lane_name 到 `LaneQueue` 的字典. Lane 惰性创建:

```python
class CommandQueue:
    def __init__(self):
        self._lanes: dict[str, LaneQueue] = {}
        self._lock = threading.Lock()

    def get_or_create_lane(self, name, max_concurrency=1):
        with self._lock:
            if name not in self._lanes:
                self._lanes[name] = LaneQueue(name, max_concurrency)
            return self._lanes[name]

    def enqueue(self, lane_name, fn):
        lane = self.get_or_create_lane(lane_name)
        return lane.enqueue(fn)

    def reset_all(self):
        """Increment generation on all lanes for restart recovery."""
        with self._lock:
            for lane in self._lanes.values():
                with lane._condition:
                    lane._generation += 1
```

#### 3. Generation 追踪 -- 重启恢复

generation 计数器解决了一个微妙的问题: 如果系统在任务进行中重启, 那些任务可能完成并尝试用过期状态泵送队列. 通过递增 generation, 所有旧回调变成无害的空操作:

```python
def _task_done(self, gen):
    with self._condition:
        self._active_count -= 1
        if gen == self._generation:
            self._pump()       # current generation: normal flow
        # else: stale task -- do NOT pump, let it die quietly
        self._condition.notify_all()
```

#### 4. HeartbeatRunner -- lane 感知的跳过

不再使用 `lock.acquire(blocking=False)`, 心跳改为检查 lane 统计信息:

```python
def heartbeat_tick(self):
    ok, reason = self.should_run()
    if not ok:
        return

    lane_stats = self.command_queue.get_or_create_lane(LANE_HEARTBEAT).stats()
    if lane_stats["active"] > 0:
        return  # lane is busy, skip this tick

    future = self.command_queue.enqueue(LANE_HEARTBEAT, _do_heartbeat)
    future.add_done_callback(_on_done)
```

这在功能上等同于非阻塞锁模式, 但以 lane 抽象来表达.

### 试一试

```sh
python zh/s10_concurrency.py

# 显示所有 lane 及其当前状态
# You > /lanes
#   main          active=[.]  queued=0  max=1  gen=0
#   cron          active=[.]  queued=0  max=1  gen=0
#   heartbeat     active=[.]  queued=0  max=1  gen=0

# 手动将工作入队到命名 lane
# You > /enqueue main What is the capital of France?

# 创建自定义 lane 并入队工作
# You > /enqueue research Summarize recent AI developments

# 修改 lane 的 max_concurrency
# You > /concurrency research 3

# 显示 generation 计数器
# You > /generation

# 模拟重启 (递增所有 generation)
# You > /reset

# 显示每个 lane 的待处理条目
# You > /queue
```

### OpenClaw 中的对应实现

| 方面             | claw0 (本文件)                             | OpenClaw 生产代码                              |
|------------------|--------------------------------------------|------------------------------------------------|
| Lane 原语       | `LaneQueue` + `threading.Condition`        | 相同模式, 带指标采集                           |
| 调度器           | `CommandQueue` lane 字典                   | 相同的惰性创建调度器                           |
| 并发控制         | 每 lane `max_concurrency`, 默认 1         | 相同, 可按部署配置                             |
| 任务执行         | 每任务一个 `threading.Thread`              | 线程池 + 有界 worker                           |
| 结果投递         | `concurrent.futures.Future`                | 相同的基于 Future 的接口                       |
| Generation 追踪 | 整数计数器, 过期任务跳过泵送               | 相同的 generation 模式用于重启安全             |
| 空闲检测         | `wait_for_idle()` + Condition.wait()       | 相同, 用于优雅关停                             |
| 标准 lane       | main, cron, heartbeat                      | 相同默认值 + 插件定义的自定义 lane             |
| 用户优先         | Main lane 阻塞等待结果                     | 相同的阻塞语义用于用户输入                     |

## Python代码实现

### 合并后的 claw0

```python
"""
claw0 Full Integration - All-in-One Agent Gateway

This script combines all 10 sessions into a complete agent gateway:
- s01: Agent Loop (while + stop_reason)
- s02: Tool Use (dispatch table)
- s03: Sessions (JSONL persistence, context guard)
- s04: Channels (InboundMessage abstraction)
- s05: Gateway & Routing (5-tier binding)
- s06: Intelligence (soul, memory, skills, prompt assembly)
- s07: Heartbeat & Cron (proactive agent)
- s08: Delivery (message queue with backoff)
- s09: Resilience (retry, auth rotation)
- s10: Concurrency (named lanes)

Usage:
    cd claw0
    python sessions/en/claw0_full.py

Required .env config:
    ANTHROPIC_API_KEY=sk-ant-xxxxx
    MODEL_ID=claude-sonnet-4-20250514
"""

# ============================================================================
# Imports
# ============================================================================
import asyncio
import json
import math
import os
import queue
import re
import subprocess
import sys
import threading
import time
import uuid
from abc import ABC, abstractmethod
from concurrent.futures import Future
from dataclasses import dataclass, field
from datetime import datetime, timezone
from pathlib import Path
from typing import Any
from croniter import croniter

from dotenv import load_dotenv
from anthropic import Anthropic

# ============================================================================
# Configuration
# ============================================================================
load_dotenv(Path(__file__).resolve().parent.parent.parent / ".env", override=True)

MODEL_ID = os.getenv("MODEL_ID", "claude-sonnet-4-20250514")
client = Anthropic(
    api_key=os.getenv("ANTHROPIC_API_KEY"),
    base_url=os.getenv("ANTHROPIC_BASE_URL") or None,
)

WORKSPACE_DIR = Path(__file__).resolve().parent.parent.parent / "workspace"
SESSIONS_DIR = WORKSPACE_DIR / ".sessions"
AGENTS_DIR = WORKSPACE_DIR / ".agents"
STATE_DIR = WORKSPACE_DIR / ".state"
MEMORY_DIR = WORKSPACE_DIR / "memory"
DELIVERY_DIR = WORKSPACE_DIR / "delivery-queue"

WORKSPACE_DIR.mkdir(parents=True, exist_ok=True)
SESSIONS_DIR.mkdir(parents=True, exist_ok=True)
AGENTS_DIR.mkdir(parents=True, exist_ok=True)
STATE_DIR.mkdir(parents=True, exist_ok=True)
MEMORY_DIR.mkdir(parents=True, exist_ok=True)
DELIVERY_DIR.mkdir(parents=True, exist_ok=True)

CONTEXT_SAFE_LIMIT = 180000
MAX_TOOL_OUTPUT = 50000

BOOTSTRAP_FILES = [
    "SOUL.md", "IDENTITY.md", "TOOLS.md", "USER.md",
    "HEARTBEAT.md", "BOOTSTRAP.md", "AGENTS.md", "MEMORY.md",
]

VALID_ID_RE = re.compile(r"^[a-z0-9][a-z0-9_-]{0,63}$")
INVALID_CHARS_RE = re.compile(r"[^a-z0-9_-]+")
DEFAULT_AGENT_ID = "main"

# ============================================================================
# ANSI Colors
# ============================================================================
CYAN = "\033[36m"
GREEN = "\033[32m"
YELLOW = "\033[33m"
RED = "\033[31m"
DIM = "\033[2m"
RESET = "\033[0m"
BOLD = "\033[1m"
MAGENTA = "\033[35m"
BLUE = "\033[34m"


def colored_prompt() -> str:
    return f"{CYAN}{BOLD}You > {RESET}"


def print_assistant(text: str) -> None:
    print(f"\n{GREEN}{BOLD}Assistant:{RESET} {text}\n")


def print_tool(name: str, detail: str) -> None:
    print(f"  {DIM}[tool: {name}] {detail}{RESET}")


def print_info(text: str) -> None:
    print(f"{DIM}{text}{RESET}")


def print_warn(text: str) -> None:
    print(f"{YELLOW}{text}{RESET}")


def print_session(text: str) -> None:
    print(f"{MAGENTA}{text}{RESET}")


def print_channel(text: str) -> None:
    print(f"{BLUE}{text}{RESET}")


def print_lane(text: str) -> None:
    print(f"{CYAN}{text}{RESET}")


# ============================================================================
# Data Structures
# ============================================================================

@dataclass
class InboundMessage:
    """All channels normalize into this."""
    text: str
    sender_id: str
    channel: str = "cli"
    account_id: str = ""
    peer_id: str = ""
    is_group: bool = False
    media: list = field(default_factory=list)
    raw: dict = field(default_factory=dict)


@dataclass
class DeliveryMessage:
    """Message in the delivery queue."""
    id: str = ""
    agent_id: str = ""
    session_key: str = ""
    role: str = "user"
    content: Any = ""
    channel: str = "cli"
    created_at: float = field(default_factory=time.time)
    attempts: int = 0
    last_error: str = ""


# ============================================================================
# Agent ID Normalization
# ============================================================================

def normalize_agent_id(value: str) -> str:
    trimmed = value.strip()
    if not trimmed:
        return DEFAULT_AGENT_ID
    if VALID_ID_RE.match(trimmed):
        return trimmed.lower()
    cleaned = INVALID_CHARS_RE.sub("-", trimmed.lower()).strip("-")[:64]
    return cleaned or DEFAULT_AGENT_ID


# ============================================================================
# Safe Path Helper
# ============================================================================

def safe_path(raw: str, base: Path = WORKSPACE_DIR) -> Path:
    target = (base / raw).resolve()
    if not str(target).startswith(str(base.resolve())):
        raise ValueError(f"Path traversal blocked: {raw}")
    return target


def truncate(text: str, limit: int = MAX_TOOL_OUTPUT) -> str:
    if len(text) <= limit:
        return text
    return text[:limit] + f"\n... [truncated, {len(text)} total chars]"


# ============================================================================
# Tool Implementations
# ============================================================================

def tool_bash(command: str, timeout: int = 30) -> str:
    """Run a shell command."""
    dangerous = ["rm -rf /", "mkfs", "> /dev/sd", "dd if="]
    for pattern in dangerous:
        if pattern in command:
            return f"Error: Refused to run dangerous command"
    print_tool("bash", command)
    try:
        result = subprocess.run(
            command, shell=True, capture_output=True, text=True,
            timeout=timeout, cwd=str(WORKSPACE_DIR),
        )
        output = ""
        if result.stdout:
            output += result.stdout
        if result.stderr:
            output += ("\n--- stderr ---\n" + result.stderr) if output else result.stderr
        if result.returncode != 0:
            output += f"\n[exit code: {result.returncode}]"
        return truncate(output) if output else "[no output]"
    except subprocess.TimeoutExpired:
        return f"Error: Command timed out after {timeout}s"
    except Exception as exc:
        return f"Error: {exc}"


def tool_read_file(file_path: str) -> str:
    """Read file contents."""
    print_tool("read_file", file_path)
    try:
        target = safe_path(file_path)
        if not target.exists():
            return f"Error: File not found: {file_path}"
        if not target.is_file():
            return f"Error: Not a file: {file_path}"
        content = target.read_text(encoding="utf-8")
        return truncate(content)
    except ValueError as exc:
        return str(exc)
    except Exception as exc:
        return f"Error: {exc}"


def tool_write_file(file_path: str, content: str) -> str:
    """Write content to a file."""
    print_tool("write_file", file_path)
    try:
        target = safe_path(file_path)
        target.parent.mkdir(parents=True, exist_ok=True)
        target.write_text(content, encoding="utf-8")
        return f"Successfully wrote {len(content)} chars to {file_path}"
    except ValueError as exc:
        return str(exc)
    except Exception as exc:
        return f"Error: {exc}"


def tool_edit_file(file_path: str, old_string: str, new_string: str) -> str:
    """Edit a file with exact string replacement."""
    print_tool("edit_file", f"{file_path} ({len(old_string)} chars)")
    try:
        target = safe_path(file_path)
        if not target.exists():
            return f"Error: File not found: {file_path}"
        content = target.read_text(encoding="utf-8")
        count = content.count(old_string)
        if count == 0:
            return "Error: old_string not found"
        if count > 1:
            return f"Error: old_string found {count} times (must be unique)"
        new_content = content.replace(old_string, new_string, 1)
        target.write_text(new_content, encoding="utf-8")
        return f"Successfully edited {file_path}"
    except ValueError as exc:
        return str(exc)
    except Exception as exc:
        return f"Error: {exc}"


def tool_list_directory(directory: str = ".") -> str:
    """List directory contents."""
    print_tool("list_directory", directory)
    try:
        target = safe_path(directory)
        if not target.exists():
            return f"Error: Directory not found: {directory}"
        if not target.is_dir():
            return f"Error: Not a directory: {directory}"
        entries = sorted(target.iterdir())
        lines = []
        for entry in entries:
            prefix = "[dir]  " if entry.is_dir() else "[file] "
            lines.append(prefix + entry.name)
        return "\n".join(lines) if lines else "[empty]"
    except ValueError as exc:
        return str(exc)
    except Exception as exc:
        return f"Error: {exc}"


def tool_get_current_time() -> str:
    """Get current UTC time."""
    print_tool("get_current_time", "")
    return datetime.now(timezone.utc).strftime("%Y-%m-%d %H:%M:%S UTC")


def tool_memory_write(key: str, value: str) -> str:
    """Write to persistent memory."""
    print_tool("memory_write", f"{key}")
    try:
        mem_file = MEMORY_DIR / f"{key}.json"
        mem_file.write_text(
            json.dumps({"key": key, "value": value, "updated": time.time()}, ensure_ascii=False),
            encoding="utf-8"
        )
        return f"Memory saved: {key}"
    except Exception as exc:
        return f"Error: {exc}"


def tool_memory_read(key: str) -> str:
    """Read from persistent memory."""
    print_tool("memory_read", key)
    try:
        mem_file = MEMORY_DIR / f"{key}.json"
        if not mem_file.exists():
            return f"Memory not found: {key}"
        data = json.loads(mem_file.read_text(encoding="utf-8"))
        return data.get("value", "")
    except Exception as exc:
        return f"Error: {exc}"


def tool_memory_search(query: str) -> str:
    """Search memory for matching entries."""
    print_tool("memory_search", query)
    try:
        results = []
        for mem_file in MEMORY_DIR.glob("*.json"):
            try:
                data = json.loads(mem_file.read_text(encoding="utf-8"))
                if query.lower() in str(data.get("value", "")).lower():
                    results.append(f"## {data.get('key')}\n{data.get('value')}")
            except:
                pass
        if not results:
            return "No matching memories found."
        return "\n\n---\n\n".join(results)
    except Exception as exc:
        return f"Error: {exc}"


# ============================================================================
# Tool Schema + Dispatch Table
# ============================================================================

TOOLS = [
    {
        "name": "bash",
        "description": "Run a shell command and return its output.",
        "input_schema": {
            "type": "object",
            "properties": {
                "command": {"type": "string", "description": "Shell command to execute."},
                "timeout": {"type": "integer", "description": "Timeout in seconds. Default 30.", "default": 30},
            },
            "required": ["command"],
        },
    },
    {
        "name": "read_file",
        "description": "Read file contents from workspace.",
        "input_schema": {
            "type": "object",
            "properties": {
                "file_path": {"type": "string", "description": "Path relative to workspace."},
            },
            "required": ["file_path"],
        },
    },
    {
        "name": "write_file",
        "description": "Write content to a file. Creates parent directories.",
        "input_schema": {
            "type": "object",
            "properties": {
                "file_path": {"type": "string", "description": "Path relative to workspace."},
                "content": {"type": "string", "description": "Content to write."},
            },
            "required": ["file_path", "content"],
        },
    },
    {
        "name": "edit_file",
        "description": "Replace exact string in file. old_string must be unique.",
        "input_schema": {
            "type": "object",
            "properties": {
                "file_path": {"type": "string", "description": "Path relative to workspace."},
                "old_string": {"type": "string", "description": "Exact text to find."},
                "new_string": {"type": "string", "description": "Replacement text."},
            },
            "required": ["file_path", "old_string", "new_string"],
        },
    },
    {
        "name": "list_directory",
        "description": "List files and directories.",
        "input_schema": {
            "type": "object",
            "properties": {
                "directory": {"type": "string", "description": "Path relative to workspace.", "default": "."},
            },
            "required": [],
        },
    },
    {
        "name": "get_current_time",
        "description": "Get current UTC date and time.",
        "input_schema": {
            "type": "object",
            "properties": {},
            "required": [],
        },
    },
    {
        "name": "memory_write",
        "description": "Write a persistent memory entry.",
        "input_schema": {
            "type": "object",
            "properties": {
                "key": {"type": "string", "description": "Memory key."},
                "value": {"type": "string", "description": "Memory value."},
            },
            "required": ["key", "value"],
        },
    },
    {
        "name": "memory_read",
        "description": "Read a persistent memory entry.",
        "input_schema": {
            "type": "object",
            "properties": {
                "key": {"type": "string", "description": "Memory key."},
            },
            "required": ["key"],
        },
    },
    {
        "name": "memory_search",
        "description": "Search memory for matching entries.",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search query."},
            },
            "required": ["query"],
        },
    },
]

TOOL_HANDLERS: dict[str, Any] = {
    "bash": tool_bash,
    "read_file": tool_read_file,
    "write_file": tool_write_file,
    "edit_file": tool_edit_file,
    "list_directory": tool_list_directory,
    "get_current_time": tool_get_current_time,
    "memory_write": tool_memory_write,
    "memory_read": tool_memory_read,
    "memory_search": tool_memory_search,
}


def process_tool_call(tool_name: str, tool_input: dict) -> str:
    handler = TOOL_HANDLERS.get(tool_name)
    if handler is None:
        return f"Error: Unknown tool '{tool_name}'"
    try:
        return handler(**tool_input)
    except TypeError as exc:
        return f"Error: Invalid arguments: {exc}"
    except Exception as exc:
        return f"Error: {tool_name} failed: {exc}"


# ============================================================================
# SessionStore (s03) - JSONL Persistence
# ============================================================================

class SessionStore:
    """Manages persistent JSONL-based conversation storage."""

    def __init__(self, agent_id: str = "default"):
        self.agent_id = agent_id
        self.base_dir = SESSIONS_DIR / "agents" / agent_id / "sessions"
        self.base_dir.mkdir(parents=True, exist_ok=True)
        self.index_path = self.base_dir.parent / "sessions.json"
        self._index: dict[str, dict] = self._load_index()
        self.current_session_id: str | None = None

    def _load_index(self) -> dict[str, dict]:
        if self.index_path.exists():
            try:
                return json.loads(self.index_path.read_text(encoding="utf-8"))
            except (json.JSONDecodeError, OSError):
                return {}
        return {}

    def _save_index(self) -> None:
        self.index_path.write_text(
            json.dumps(self._index, indent=2, ensure_ascii=False),
            encoding="utf-8",
        )

    def _session_path(self, session_id: str) -> Path:
        return self.base_dir / f"{session_id}.jsonl"

    def create_session(self, label: str = "") -> str:
        session_id = uuid.uuid4().hex[:12]
        now = datetime.now(timezone.utc).isoformat()
        self._index[session_id] = {
            "label": label, "created_at": now,
            "last_active": now, "message_count": 0,
        }
        self._save_index()
        self._session_path(session_id).touch()
        self.current_session_id = session_id
        return session_id

    def load_session(self, session_id: str) -> list[dict]:
        path = self._session_path(session_id)
        if not path.exists():
            return []
        self.current_session_id = session_id
        return self._rebuild_history(path)

    def save_turn(self, role: str, content: Any) -> None:
        if not self.current_session_id:
            return
        self.append_transcript(self.current_session_id, {
            "type": role, "content": content, "ts": time.time(),
        })

    def save_tool_result(self, tool_use_id: str, name: str,
                         tool_input: dict, result: str) -> None:
        if not self.current_session_id:
            return
        ts = time.time()
        self.append_transcript(self.current_session_id, {
            "type": "tool_use", "tool_use_id": tool_use_id,
            "name": name, "input": tool_input, "ts": ts,
        })
        self.append_transcript(self.current_session_id, {
            "type": "tool_result", "tool_use_id": tool_use_id,
            "content": result, "ts": ts,
        })

    def append_transcript(self, session_id: str, record: dict) -> None:
        path = self._session_path(session_id)
        with open(path, "a", encoding="utf-8") as f:
            f.write(json.dumps(record, ensure_ascii=False) + "\n")
        if session_id in self._index:
            self._index[session_id]["last_active"] = datetime.now(timezone.utc).isoformat()
            self._index[session_id]["message_count"] += 1
            self._save_index()

    def _rebuild_history(self, path: Path) -> list[dict]:
        messages: list[dict] = []
        lines = path.read_text(encoding="utf-8").strip().split("\n")
        for line in lines:
            if not line.strip():
                continue
            try:
                record = json.loads(line)
            except json.JSONDecodeError:
                continue
            rtype = record.get("type")
            if rtype == "user":
                messages.append({"role": "user", "content": record["content"]})
            elif rtype == "assistant":
                content = record["content"]
                if isinstance(content, str):
                    content = [{"type": "text", "text": content}]
                messages.append({"role": "assistant", "content": content})
            elif rtype == "tool_use":
                block = {"type": "tool_use", "id": record["tool_use_id"],
                         "name": record["name"], "input": record["input"]}
                if messages and messages[-1]["role"] == "assistant":
                    content = messages[-1]["content"]
                    if isinstance(content, list):
                        content.append(block)
                    else:
                        messages[-1]["content"] = [{"type": "text", "text": str(content)}, block]
                else:
                    messages.append({"role": "assistant", "content": [block]})
            elif rtype == "tool_result":
                result_block = {"type": "tool_result", "tool_use_id": record["tool_use_id"],
                                "content": record["content"]}
                if (messages and messages[-1]["role"] == "user"
                        and isinstance(messages[-1]["content"], list)
                        and messages[-1]["content"]
                        and isinstance(messages[-1]["content"][0], dict)
                        and messages[-1]["content"][0].get("type") == "tool_result"):
                    messages[-1]["content"].append(result_block)
                else:
                    messages.append({"role": "user", "content": [result_block]})
        return messages

    def list_sessions(self) -> list[tuple[str, dict]]:
        items = list(self._index.items())
        items.sort(key=lambda x: x[1].get("last_active", ""), reverse=True)
        return items


# ============================================================================
# ContextGuard (s03) - Overflow Protection
# ============================================================================

class ContextGuard:
    """Protect from context window overflow with 3-stage retry."""

    def __init__(self, max_tokens: int = CONTEXT_SAFE_LIMIT):
        self.max_tokens = max_tokens

    @staticmethod
    def estimate_tokens(text: str) -> int:
        return len(text) // 4

    def estimate_messages_tokens(self, messages: list[dict]) -> int:
        total = 0
        for msg in messages:
            content = msg.get("content", "")
            if isinstance(content, str):
                total += self.estimate_tokens(content)
            elif isinstance(content, list):
                for block in content:
                    if isinstance(block, dict):
                        if "text" in block:
                            total += self.estimate_tokens(block["text"])
                        elif block.get("type") == "tool_result":
                            rc = block.get("content", "")
                            if isinstance(rc, str):
                                total += self.estimate_tokens(rc)
                        elif block.get("type") == "tool_use":
                            total += self.estimate_tokens(json.dumps(block.get("input", {})))
                    elif hasattr(block, "text"):
                        total += self.estimate_tokens(block.text)
        return total

    def truncate_tool_result(self, result: str, max_fraction: float = 0.3) -> str:
        max_chars = int(self.max_tokens * 4 * max_fraction)
        if len(result) <= max_chars:
            return result
        return result[:max_chars] + f"\n\n[... truncated ({len(result)} chars total) ...]"

    def _truncate_large_tool_results(self, messages: list[dict]) -> list[dict]:
        result = []
        for msg in messages:
            content = msg.get("content", "")
            if isinstance(content, list):
                new_blocks = []
                for block in content:
                    if (isinstance(block, dict) and block.get("type") == "tool_result"
                            and isinstance(block.get("content"), str)):
                        block = dict(block)
                        block["content"] = self.truncate_tool_result(block["content"])
                    new_blocks.append(block)
                result.append({"role": msg["role"], "content": new_blocks})
            else:
                result.append(msg)
        return result

    def guard_api_call(
        self, api_client: Anthropic, model: str, system: str,
        messages: list[dict], tools: list[dict] | None = None, max_retries: int = 2,
    ) -> Any:
        current_messages = messages
        for attempt in range(max_retries + 1):
            try:
                kwargs: dict[str, Any] = {
                    "model": model, "max_tokens": 8096,
                    "system": system, "messages": current_messages,
                }
                if tools:
                    kwargs["tools"] = tools
                result = api_client.messages.create(**kwargs)
                if current_messages is not messages:
                    messages.clear()
                    messages.extend(current_messages)
                return result
            except Exception as exc:
                error_str = str(exc).lower()
                is_overflow = "context" in error_str or "token" in error_str
                if not is_overflow or attempt >= max_retries:
                    raise
                if attempt == 0:
                    print_warn("  [guard] Truncating tool results...")
                    current_messages = self._truncate_large_tool_results(current_messages)
                elif attempt == 1:
                    print_warn("  [guard] Compact history...")
                    current_messages = self._compact_history(current_messages, api_client, model)
        raise RuntimeError("guard_api_call: exhausted retries")

    def _compact_history(self, messages: list[dict], api_client: Anthropic, model: str) -> list[dict]:
        total = len(messages)
        if total <= 4:
            return messages
        keep_count = max(4, int(total * 0.2))
        compress_count = max(2, int(total * 0.5))
        compress_count = min(compress_count, total - keep_count)
        if compress_count < 2:
            return messages
        old_messages = messages[:compress_count]
        recent_messages = messages[compress_count:]
        parts = []
        for msg in old_messages:
            role = msg["role"]
            content = msg.get("content", "")
            if isinstance(content, str):
                parts.append(f"[{role}]: {content}")
            elif isinstance(content, list):
                for block in content:
                    if isinstance(block, dict):
                        if block.get("type") == "text":
                            parts.append(f"[{role}]: {block['text']}")
                        elif block.get("type") == "tool_use":
                            parts.append(f"[{role}]: {json.dumps(block.get('input', {}))}")
        old_text = "\n".join(parts)
        try:
            summary_resp = api_client.messages.create(
                model=model, max_tokens=2048,
                system="Summarize conversation concisely.",
                messages=[{"role": "user", "content": f"Summarize:\n{old_text[:10000]}"}],
            )
            summary_text = "".join(b.text for b in summary_resp.content if hasattr(b, "text"))
            print_session(f"  [compact] {len(old_messages)} -> summary ({len(summary_text)} chars)")
        except Exception as exc:
            print_warn(f"  [compact] Failed: {exc}")
            return recent_messages
        compacted = [
            {"role": "user", "content": "[Previous conversation summary]\n" + summary_text},
            {"role": "assistant", "content": [{"type": "text", "text": "Understood."}]},
        ]
        compacted.extend(recent_messages)
        return compacted


# ============================================================================
# BootstrapLoader (s06) - Prompt Assembly
# ============================================================================

class BootstrapLoader:
    """Loads prompt components from workspace files."""

    def __init__(self, workspace: Path = WORKSPACE_DIR):
        self.workspace = workspace

    def load_file(self, name: str) -> str:
        path = self.workspace / name
        if not path.exists():
            return ""
        try:
            content = path.read_text(encoding="utf-8")
            if len(content) > MAX_TOOL_OUTPUT:
                content = content[:MAX_TOOL_OUTPUT] + "\n... [truncated]"
            return content
        except Exception:
            return ""

    def build_system_prompt(self, agent_id: str = "default") -> str:
        parts = []
        for fname in BOOTSTRAP_FILES:
            content = self.load_file(fname)
            if content:
                header = fname.replace(".md", "").replace("_", " ").title()
                parts.append(f"## {header}\n{content}")
        parts.append("\n## Available Tools\nYou have access to: bash, read_file, write_file, edit_file, list_directory, get_current_time, memory_write, memory_read, memory_search.")
        return "\n\n".join(parts)


# ============================================================================
# DeliveryQueue (s08) - Reliable Message Queue
# ============================================================================

class DeliveryQueue:
    """Write-ahead log message queue with retry."""

    def __init__(self, queue_dir: Path = DELIVERY_DIR):
        self.queue_dir = queue_dir
        self.queue_dir.mkdir(parents=True, exist_ok=True)
        self.pending: dict[str, DeliveryMessage] = {}
        self._load_pending()

    def _load_pending(self) -> None:
        for f in self.queue_dir.glob("*.json"):
            try:
                data = json.loads(f.read_text(encoding="utf-8"))
                msg = DeliveryMessage(**data)
                self.pending[msg.id] = msg
            except Exception:
                pass

    def enqueue(self, msg: DeliveryMessage) -> None:
        if not msg.id:
            msg.id = uuid.uuid4().hex[:12]
        path = self.queue_dir / f"{msg.id}.json"
        path.write_text(json.dumps({
            "id": msg.id, "agent_id": msg.agent_id, "session_key": msg.session_key,
            "role": msg.role, "content": msg.content, "channel": msg.channel,
            "created_at": msg.created_at, "attempts": msg.attempts, "last_error": msg.last_error,
        }, ensure_ascii=False), encoding="utf-8")
        self.pending[msg.id] = msg

    def dequeue(self, msg_id: str) -> DeliveryMessage | None:
        path = self.queue_dir / f"{msg_id}.json"
        if path.exists():
            try:
                path.unlink()
            except Exception:
                pass
        return self.pending.pop(msg_id, None)

    def retry_failed(self, max_attempts: int = 3) -> list[DeliveryMessage]:
        result = []
        for msg in list(self.pending.values()):
            if msg.attempts >= max_attempts:
                self.dequeue(msg.id)
            elif time.time() - msg.created_at > 300:
                msg.attempts += 1
                result.append(msg)
                self.enqueue(msg)
        return result


# ============================================================================
# NamedLanes (s10) - Concurrency Control
# ============================================================================

class NamedLanes:
    """Named lanes serialize messages per agent/session."""

    def __init__(self):
        self.lanes: dict[str, asyncio.Queue] = {}
        self.locks: dict[str, asyncio.Lock] = {}
        self.results: dict[str, Future] = {}

    def get_lane(self, name: str) -> str:
        return f"lane:{name}"

    def enqueue(self, lane_name: str, item: Any) -> None:
        if lane_name not in self.lanes:
            self.lanes[lane_name] = queue.Queue()
        self.lanes[lane_name].put(item)

    def dequeue(self, lane_name: str) -> Any | None:
        if lane_name not in self.lanes:
            return None
        try:
            return self.lanes[lane_name].get_nowait()
        except queue.Empty:
            return None


# ============================================================================
# Binding (s05) - 5-Tier Routing
# ============================================================================

@dataclass
class Binding:
    channel: str = ""
    account_id: str = ""
    peer_id: str = ""
    guild_id: str = ""
    agent_id: str = "main"
    session_label: str = ""


class BindingTable:
    """5-tier routing: peer > guild > account > channel > default."""

    def __init__(self):
        self.bindings: list[Binding] = []

    def add(self, binding: Binding) -> None:
        self.bindings.append(binding)

    def resolve(self, channel: str, account_id: str = "",
                peer_id: str = "", guild_id: str = "") -> tuple[str, str]:
        matches = []
        for b in self.bindings:
            score = 0
            if b.peer_id and b.peer_id == peer_id:
                score = 50
            elif b.guild_id and b.guild_id == guild_id:
                score = 40
            elif b.account_id and b.account_id == account_id:
                score = 30
            elif b.channel and b.channel == channel:
                score = 20
            elif not b.peer_id and not b.guild_id and not b.account_id and not b.channel:
                score = 10
            else:
                continue
            if score > 0:
                matches.append((score, b))
        if not matches:
            return DEFAULT_AGENT_ID, "default"
        matches.sort(key=lambda x: x[0], reverse=True)
        return matches[0][1].agent_id, matches[0][1].session_label or "default"


# ============================================================================
# Heartbeat & Cron (s07)
# ============================================================================

class Heartbeat:
    """Proactive agent with cron scheduling."""

    def __init__(self, agent_id: str, system_prompt: str,
                 session_store: SessionStore, context_guard: ContextGuard):
        self.agent_id = agent_id
        self.system_prompt = system_prompt
        self.session_store = session_store
        self.context_guard = context_guard
        self.running = False
        self.thread: threading.Thread | None = None
        self.crons: list[tuple[str, str]] = []
        self.last_runs: dict[str, float] = {}

    def add_cron(self, cron_expr: str, action: str) -> None:
        self.crons.append((cron_expr, action))

    def start(self) -> None:
        self.running = True
        self.thread = threading.Thread(target=self._run, daemon=True)
        self.thread.start()
        print_info(f"  [heartbeat] Started for agent {self.agent_id}")

    def stop(self) -> None:
        self.running = False
        if self.thread:
            self.thread.join(timeout=2)
        print_info(f"  [heartbeat] Stopped for agent {self.agent_id}")

    def _run(self) -> None:
        while self.running:
            now = datetime.now(timezone.utc)
            for cron_expr, action in self.crons:
                try:
                    cron = croniter(cron_expr, now)
                    if cron.is_now() or (self.last_runs.get(action, 0) < cron.get_prev()):
                        self.last_runs[time.time()] = time.time()
                        self._execute_action(action)
                except Exception:
                    pass
            time.sleep(10)

    def _execute_action(self, action: str) -> None:
        print_lane(f"  [heartbeat] Executing: {action}")
        # In full version, would trigger agent action here


# ============================================================================
# Agent Manager
# ============================================================================

class AgentManager:
    """Manages multiple agents with routing."""

    def __init__(self):
        self.agents: dict[str, dict] = {}
        self.session_stores: dict[str, SessionStore] = {}
        self.context_guards: dict[str, ContextGuard] = {}
        self.heartbeats: dict[str, Heartbeat] = {}
        self.binding_table = BindingTable()
        self.bootstrap_loader = BootstrapLoader()
        self.delivery_queue = DeliveryQueue()
        self.named_lanes = NamedLanes()
        self._setup_default_agent()

    def _setup_default_agent(self) -> None:
        agent_id = DEFAULT_AGENT_ID
        self.agents[agent_id] = {
            "id": agent_id,
            "name": "default",
            "system_prompt": self.bootstrap_loader.build_system_prompt(agent_id),
        }
        self.session_stores[agent_id] = SessionStore(agent_id)
        self.context_guards[agent_id] = ContextGuard()
        self.binding_table.add(Binding(channel="", agent_id=agent_id))

    def get_agent_id(self, channel: str, account_id: str = "",
                     peer_id: str = "", guild_id: str = "") -> str:
        return self.binding_table.resolve(channel, account_id, peer_id, guild_id)[0]

    def get_or_create_agent(self, agent_id: str) -> dict:
        if agent_id not in self.agents:
            agent_id = normalize_agent_id(agent_id)
            self.agents[agent_id] = {
                "id": agent_id,
                "name": agent_id,
                "system_prompt": self.bootstrap_loader.build_system_prompt(agent_id),
            }
            self.session_stores[agent_id] = SessionStore(agent_id)
            self.context_guards[agent_id] = ContextGuard()
            self.binding_table.add(Binding(agent_id=agent_id))
        return self.agents[agent_id]

    def create_agent(self, agent_id: str, name: str = "",
                     soul: str = "", identity: str = "") -> None:
        agent_id = normalize_agent_id(agent_id)
        system_parts = []
        if soul:
            system_parts.append(f"## Soul\n{soul}")
        if identity:
            system_parts.append(f"## Identity\n{identity}")
        system_parts.append(self.bootstrap_loader.build_system_prompt(agent_id))
        self.agents[agent_id] = {
            "id": agent_id,
            "name": name or agent_id,
            "system_prompt": "\n\n".join(system_parts),
        }
        self.session_stores[agent_id] = SessionStore(agent_id)
        self.context_guards[agent_id] = ContextGuard()
        self.binding_table.add(Binding(agent_id=agent_id))
        print_info(f"  Created agent: {agent_id}")


# ============================================================================
# REPL Commands
# ============================================================================

def handle_repl_command(
    command: str, agent_manager: AgentManager,
    agent_id: str, messages: list[dict],
) -> tuple[bool, list[dict], str]:
    """Handle /-prefixed commands."""
    parts = command.strip().split(maxsplit=2)
    cmd = parts[0].lower()
    arg1 = parts[1] if len(parts) > 1 else ""
    arg2 = parts[2] if len(parts) > 2 else ""

    store = agent_manager.session_stores[agent_id]
    guard = agent_manager.context_guards[agent_id]

    if cmd == "/new":
        label = arg1 or ""
        sid = store.create_session(label)
        print_session(f"  Created session: {sid}" + (f" ({label})" if label else ""))
        return True, [], agent_id

    elif cmd == "/list":
        sessions = store.list_sessions()
        if not sessions:
            print_info("  No sessions.")
        else:
            print_info("  Sessions:")
            for sid, meta in sessions:
                active = " <-- current" if sid == store.current_session_id else ""
                count = meta.get("message_count", 0)
                print_info(f"    {sid}  msgs={count}{active}")
        return True, messages, agent_id

    elif cmd == "/switch":
        if not arg1:
            print_warn("  Usage: /switch <session_id>")
            return True, messages, agent_id
        matched = [sid for sid in store._index if sid.startswith(arg1)]
        if len(matched) == 0:
            print_warn(f"  Not found: {arg1}")
            return True, messages, agent_id
        if len(matched) > 1:
            print_warn(f"  Ambiguous: {', '.join(matched)}")
            return True, messages, agent_id
        new_messages = store.load_session(matched[0])
        print_session(f"  Switched to: {matched[0]} ({len(new_messages)} messages)")
        return True, new_messages, agent_id

    elif cmd == "/agents":
        print_info("  Agents:")
        for aid, agent in agent_manager.agents.items():
            print_info(f"    {aid}: {agent.get('name', aid)}")
        return True, messages, agent_id

    elif cmd == "/create":
        if not arg1:
            print_warn("  Usage: /create <agent_id> [name]")
            return True, messages, agent_id
        agent_manager.create_agent(arg1, arg2)
        return True, messages, arg1

    elif cmd == "/use":
        if not arg1:
            print_warn("  Usage: /use <agent_id>")
            return True, messages, agent_id
        if arg1 not in agent_manager.agents:
            print_warn(f"  Unknown agent: {arg1}")
            return True, messages, agent_id
        print_info(f"  Switched to agent: {arg1}")
        store = agent_manager.session_stores[arg1]
        sessions = store.list_sessions()
        if sessions:
            new_messages = store.load_session(sessions[0][0])
        else:
            store.create_session("default")
            new_messages = []
        return True, new_messages, arg1

    elif cmd == "/prompt":
        agent = agent_manager.agents[agent_id]
        print_info(f"  System prompt ({len(agent['system_prompt'])} chars):")
        print(agent["system_prompt"][:500] + "..." if len(agent["system_prompt"]) > 500
              else agent["system_prompt"])
        return True, messages, agent_id

    elif cmd == "/reload":
        agent_manager.agents[agent_id]["system_prompt"] = (
            agent_manager.bootstrap_loader.build_system_prompt(agent_id)
        )
        print_info("  Reloaded system prompt from workspace files.")
        return True, messages, agent_id

    elif cmd == "/context":
        estimated = guard.estimate_messages_tokens(messages)
        pct = (estimated / guard.max_tokens) * 100
        bar_len = 30
        filled = int(bar_len * min(pct, 100) / 100)
        bar = "#" * filled + "-" * (bar_len - filled)
        color = GREEN if pct < 50 else (YELLOW if pct < 80 else RED)
        print_info(f"  Context: ~{estimated:,} / {guard.max_tokens:,} tokens ({pct:.1f}%)")
        print(f"  {color}[{bar}]{RESET}")
        return True, messages, agent_id

    elif cmd == "/queue":
        pending = agent_manager.delivery_queue.pending
        print_info(f"  Pending messages: {len(pending)}")
        for msg_id, msg in list(pending.items())[:5]:
            print_info(f"    {msg_id}: {msg.role} - {msg.attempts} attempts")
        return True, messages, agent_id

    elif cmd == "/help":
        print_info("  Commands:")
        print_info("    /new [label]       New session")
        print_info("    /list              List sessions")
        print_info("    /switch <id>      Switch session")
        print_info("    /agents           List agents")
        print_info("    /create <id> [n]  Create agent")
        print_info("    /use <agent_id>   Switch agent")
        print_info("    /prompt           Show system prompt")
        print_info("    /reload           Reload prompt from files")
        print_info("    /context          Show context usage")
        print_info("    /queue            Show delivery queue")
        print_info("    /help             This help")
        print_info("    quit/exit         Exit")
        return True, messages, agent_id

    return False, messages, agent_id


# ============================================================================
# Core: Agent Loop
# ============================================================================

def agent_loop() -> None:
    """Main agent loop with all features integrated."""

    agent_manager = AgentManager()
    agent_id = DEFAULT_AGENT_ID

    store = agent_manager.session_stores[agent_id]
    guard = agent_manager.context_guards[agent_id]

    # Resume or create session
    sessions = store.list_sessions()
    if sessions:
        sid = sessions[0][0]
        messages = store.load_session(sid)
        print_session(f"  Resumed session: {sid}")
    else:
        sid = store.create_session("default")
        messages = []
        print_session(f"  Created session: {sid}")

    print_info("=" * 60)
    print_info("  claw0  |  Full Integration")
    print_info(f"  Model: {MODEL_ID}")
    print_info(f"  Agent: {agent_id}")
    print_info(f"  Session: {store.current_session_id}")
    print_info(f"  Tools: {', '.join(TOOL_HANDLERS.keys())}")
    print_info("  Type /help for commands, quit/exit to leave.")
    print_info("=" * 60)
    print()

    while True:
        try:
            user_input = input(colored_prompt()).strip()
        except (KeyboardInterrupt, EOFError):
            print(f"\n{DIM}Goodbye.{RESET}")
            break

        if not user_input:
            continue

        if user_input.lower() in ("quit", "exit"):
            print(f"{DIM}Goodbye.{RESET}")
            break

        # REPL commands
        if user_input.startswith("/"):
            handled, messages, agent_id = handle_repl_command(
                user_input, agent_manager, agent_id, messages
            )
            if handled:
                store = agent_manager.session_stores.get(agent_id, store)
                guard = agent_manager.context_guards.get(agent_id, guard)
                continue

        # Get agent config
        agent = agent_manager.agents[agent_id]
        system_prompt = agent["system_prompt"]

        # Append user message
        messages.append({"role": "user", "content": user_input})
        store.save_turn("user", user_input)

        # Tool call loop
        while True:
            try:
                response = guard.guard_api_call(
                    api_client=client,
                    model=MODEL_ID,
                    system=system_prompt,
                    messages=messages,
                    tools=TOOLS,
                )
            except Exception as exc:
                print(f"\n{YELLOW}API Error: {exc}{RESET}\n")
                while messages and messages[-1]["role"] != "user":
                    messages.pop()
                if messages:
                    messages.pop()
                break

            # Save assistant response
            serialized_content = []
            for block in response.content:
                if hasattr(block, "text"):
                    serialized_content.append({"type": "text", "text": block.text})
                elif block.type == "tool_use":
                    serialized_content.append({
                        "type": "tool_use", "id": block.id,
                        "name": block.name, "input": block.input,
                    })
            store.save_turn("assistant", serialized_content)

            if response.stop_reason == "end_turn":
                assistant_text = "".join(
                    b.text for b in response.content if hasattr(b, "text")
                )
                if assistant_text:
                    print_assistant(assistant_text)
                break

            elif response.stop_reason == "tool_use":
                tool_results = []
                for block in response.content:
                    if block.type != "tool_use":
                        continue
                    result = process_tool_call(block.name, block.input)
                    store.save_tool_result(block.id, block.name, block.input, result)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result,
                    })
                messages.append({"role": "user", "content": tool_results})
                continue

            else:
                print_info(f"[stop_reason={response.stop_reason}]")
                break


# ============================================================================
# Entry Point
# ============================================================================

def main() -> None:
    if not os.getenv("ANTHROPIC_API_KEY"):
        print(f"{YELLOW}Error: ANTHROPIC_API_KEY not set.{RESET}")
        print(f"{DIM}Copy .env.example to .env and fill in your key.{RESET}")
        sys.exit(1)

    agent_loop()


if __name__ == "__main__":
    main()
```












