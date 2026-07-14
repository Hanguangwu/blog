---
title: 用 Python 同步 JIRA 数据到本地：从入门到实战
description: 本文介绍用 Python 同步 JIRA 数据到本地的基本过程。
date: 2026-07-14T12:34:25-08:00
draft: false
categories:
- 编程
tags:
- Python
- JIRA
---

# 用 Python 同步 JIRA 数据到本地：从入门到实战

## 一、JIRA 是什么？

**JIRA** 是 Atlassian 公司开发的一款**项目管理和问题追踪工具**，广泛应用于软件团队的敏捷开发、Bug 跟踪、需求管理和任务协作中。

简单来说，JIRA 就是一个"一切皆 Issue"的工作流引擎：

- **需求**（Story）是一个 Issue
- **Bug** 是一个 Issue
- **子任务**（Sub-task）也是一个 Issue
- 每个 Issue 有自己的**类型、状态、优先级、经办人、时间估算、自定义字段**等

JIRA 的核心价值在于：

| 能力           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| **工作流定制** | 每个 Issue 在不同状态间流转（待办→进行中→已完成），可自定义步骤和条件 |
| **敏捷看板**   | Scrum / Kanban 面板，拖拽管理任务                            |
| **自定义字段** | 不限于内置字段，可随意添加业务需要的字段                     |
| **强大的 JQL** | JIRA Query Language，类似 SQL 的灵活搜索语言                 |
| **权限体系**   | 颗粒度到字段级别的权限控制                                   |
| **集成生态**   | 与 Git、CI/CD、Slack、Confluence 深度集成                    |

## 二、为什么要把 JIRA 数据同步到本地？

虽然 JIRA 提供了 Web UI 和 REST API，但有些场景下把数据拉到本地更高效：

1. **离线备份** — 防止 JIRA 服务故障或配置丢失
2. **自定义分析** — 在本地用 Pandas / SQL 做更灵活的统计报表
3. **数据迁移** — 从 JIRA 迁移到其他平台前的数据整理
4. **自动化处理** — 批量修改、批量提取 Gherkin 用例、合规审计等
5. **性能考量** — 大规模查询时，本地处理比轮询 JIRA API 快得多

## 三、Python 调用 JIRA 的准备工作

### 安装依赖

```bash
pip install jira python-dotenv
```

- `jira` — Atlassian 官方推荐的 Python 客户端库
- `python-dotenv` — 从 `.env` 文件加载配置，避免硬编码凭据

### 配置 `.env` 文件

在项目目录下创建 `.env`，写入以下内容（**请使用你自己的实际值替换示例**）：

```bash
JIRA_SERVER=https://your-company.atlassian.net/
JIRA_USERNAME=your-email@example.com
JIRA_PASSWORD=your-api-token-or-password
```

> **安全提示：** 永远不要将包含真实凭据的 `.env` 文件提交到 Git。请在 `.gitignore` 中添加 `.env`。

### 什么是 API Token？

Atlassian 推荐使用 API Token 而非密码认证：

1. 登录 https://id.atlassian.com/manage-profile/security/api-tokens
2. 点击"Create API token"
3. 复制生成的 token 作为 `JIRA_PASSWORD` 的值

## 四、基础用法示例

### 4.1 连接并列出所有项目

```python
from jira import JIRA
import os
from dotenv import load_dotenv

load_dotenv()

jira_server = os.getenv("JIRA_SERVER")
jira_user = os.getenv("JIRA_USERNAME")
jira_pass = os.getenv("JIRA_PASSWORD")

jr = JIRA(server=jira_server, basic_auth=(jira_user, jira_pass))

projects = jr.projects()
print(f"共发现 {len(projects)} 个项目：")
for p in projects:
    print(f"  {p.key}: {p.name}")
```

**输出示例：**

```
共发现 12 个项目：
  PROJ: 我的项目
  BUGS: 缺陷追踪
  DOCS: 文档管理
  ...
```

### 4.2 搜索指定项目的 Issue

```python
# 搜索所有 JQL 都支持的功能
issues = jr.search_issues(
    'project = "PROJ" AND status = "进行中"',
    maxResults=50,
    fields="summary,status,assignee,created",
)
for issue in issues:
    print(f"{issue.key}: {issue.fields.summary}")
```

**常用 JQL 示例：**

| JQL                               | 含义                |
| --------------------------------- | ------------------- |
| `project = PROJ`                  | PROJ 项目所有 Issue |
| `assignee = currentUser()`        | 指派给我的          |
| `updated >= -7d`                  | 最近一周更新的      |
| `status in (Open, "In Progress")` | 未关闭的            |
| `ORDER BY created DESC`           | 按创建时间倒序      |

### 4.3 获取 Issue 完整详情

```python
issue = jr.issue("PROJ-123", fields="*all", expand="changelog")
print(issue.raw)  # 完整的 JSON 数据
```

`fields="*all"` 返回所有字段（包括自定义字段），`expand="changelog"` 返回变更历史。

## 五、实战：把整个 JIRA 同步到本地

下面是一个完整的脚本，它会遍历所有项目、分页拉取全部 Issue、保存为本地 JSON 文件。

### 完整代码

```python
from jira import JIRA
import os
import json
from datetime import datetime
from dotenv import load_dotenv

# 1. 加载 .env 配置
load_dotenv()

JIRA_SERVER = os.getenv("JIRA_SERVER")
JIRA_USERNAME = os.getenv("JIRA_USERNAME")
JIRA_PASSWORD = os.getenv("JIRA_PASSWORD")

if not all([JIRA_SERVER, JIRA_USERNAME, JIRA_PASSWORD]):
    raise ValueError("请在 .env 中配置 JIRA_SERVER、JIRA_USERNAME、JIRA_PASSWORD")

# 2. 连接 JIRA
jr = JIRA(server=JIRA_SERVER, basic_auth=(JIRA_USERNAME, JIRA_PASSWORD))
print(f"已连接 JIRA: {JIRA_SERVER}")

# 3. 获取所有项目
projects = jr.projects()
print(f"\n共发现 {len(projects)} 个项目:\n")
for p in projects:
    print(f"  - {p.key}: {p.name}")

# 创建输出目录
output_dir = os.path.join(os.path.dirname(__file__), "jira_backup")
os.makedirs(output_dir, exist_ok=True)

# 4. 遍历每个项目，获取全部 Issue
for project in projects:
    project_key = project.key
    print(f"\n正在同步项目: {project_key} - {project.name}")

    all_issues = []
    start_at = 0
    page_size = 100

    while True:
        issues = jr.search_issues(
            f'project = "{project_key}"',
            startAt=start_at,
            maxResults=page_size,
            fields="*all",
            expand="changelog",
        )
        if not issues:
            break

        for issue in issues:
            # 逐个获取完整详情
            full_issue = jr.issue(
                issue.key,
                fields="*all",
                expand="changelog,renderedFields",
            )
            all_issues.append(full_issue.raw)
            print(f"  ✓ {issue.key}: {issue.fields.summary[:60]}")

        start_at += len(issues)
        if len(issues) < page_size:
            break

    # 保存到 JSON 文件
    output_file = os.path.join(output_dir, f"{project_key}.json")
    with open(output_file, "w", encoding="utf-8") as f:
        json.dump({
            "project_key": project_key,
            "project_name": project.name,
            "synced_at": datetime.now().isoformat(),
            "total_issues": len(all_issues),
            "issues": all_issues,
        }, f, ensure_ascii=False, indent=2)

    print(f"  已保存 {len(all_issues)} 个 Issue → {output_file}")

print(f"\n全部同步完成！数据保存在: {output_dir}")
```

### 运行效果

```
已连接 JIRA: https://your-company.atlassian.net/

共发现 12 个项目:

  - PROJ: 我的项目
  - BUGS: 缺陷追踪
  ...

正在同步项目: PROJ - 我的项目
  ✓ PROJ-1: 用户登录功能开发
  ✓ PROJ-2: 修复首页白屏问题
  ✓ PROJ-3: 数据导出性能优化
  ...
  已保存 128 个 Issue → jira_backup/PROJ.json

全部同步完成！数据保存在: jira_backup/
```

### 输出的 JSON 结构

```json
{
  "project_key": "PROJ",
  "project_name": "我的项目",
  "synced_at": "2026-07-14T11:00:00",
  "total_issues": 128,
  "issues": [
    {
      "id": "10001",
      "key": "PROJ-1",
      "self": "https://your-company.atlassian.net/rest/api/2/issue/10001",
      "fields": {
        "summary": "用户登录功能开发",
        "issuetype": { "name": "故事" },
        "status": { "name": "进行中" },
        "assignee": { "displayName": "张三" },
        "created": "2026-06-01T10:00:00.000+0800",
        "updated": "2026-07-10T15:30:00.000+0800",
        "customfield_10602": "Gherkin 用例文本...",
        "description": "作为用户，我想要...",
        "comment": { "comments": [...] },
        ...
      }
    }
  ]
}
```

## 六、进阶技巧

### 6.1 只获取变更的 Issue（增量同步）

如果每天同步一次，没必要每次都全量拉取。用 JQL 按更新日期过滤：

```python
from datetime import datetime, timedelta

since = (datetime.now() - timedelta(days=1)).strftime("%Y-%m-%d %H:%M")
jql = f'project = "PROJ" AND updated >= "{since}"'
issues = jr.search_issues(jql, fields="*all")
```

### 6.2 导出为 CSV 而非 JSON

```python
import csv

with open("jira_export.csv", "w", newline="", encoding="utf-8-sig") as f:
    writer = csv.writer(f)
    writer.writerow(["Issue Key", "Summary", "Status", "Assignee", "Created"])
    for issue in issues:
        writer.writerow([
            issue.key,
            issue.fields.summary,
            issue.fields.status.name,
            getattr(issue.fields.assignee, "displayName", ""),
            issue.fields.created,
        ])
```

### 6.3 处理大项目（分批导出）

超过 1000 条 Issue 时务必分页，JIRA API 默认单页最多 100 条：

```python
start_at = 0
page_size = 100
all_issues = []

while True:
    batch = jr.search_issues(
        'project = "PROJ"',
        startAt=start_at,
        maxResults=page_size,
    )
    if not batch:
        break
    all_issues.extend(batch)
    start_at += len(batch)
    print(f"已获取 {start_at} 条...")
```

### 6.4 从本地 JSON 还原数据到新 JIRA

```python
import json

with open("jira_backup/PROJ.json", "r", encoding="utf-8") as f:
    data = json.load(f)

for issue_data in data["issues"]:
    fields = issue_data["fields"]
    new_issue = jr.create_issue(
        project={"key": "PROJ"},
        summary=fields["summary"],
        description=fields.get("description", ""),
        issuetype={"name": fields["issuetype"]["name"]},
    )
    print(f"已创建: {new_issue.key}")
```

## 七、常见问题

### Q: JQL 报错 "The character '*' is a reserved JQL character"

`*` 是 JQL 的保留通配符，必须用引号括起来。错误写法：

```python
jr.search_issues("PROJ:*")        # ❌ 会报 400 错误
```

正确写法：

```python
jr.search_issues('project = "PROJ"')  # ✅
```

### Q: `Project` object has no attribute 'lead'

`jr.projects()` 返回的是轻量级 Project 对象，不包含 `lead` 属性。如果确实需要 lead 信息：

```python
full_project = jr.project("PROJ")  # 单独获取完整项目信息
print(full_project.lead.displayName)
```

### Q: 如何处理 JIRA 的 rate limit？

JIRA Cloud 对 API 有速率限制。建议：

- 每页获取 100 条（最大值），减少请求次数
- 在每次请求之间添加 `time.sleep(0.5)`
- 考虑使用 JIRA Software Data Center（自托管）免除限制

## 八、总结

JIRA 的 Python API 非常成熟，用 `jira` 库几十行代码就能完成数据同步。核心要点：

1. **凭据管理** — 用 `.env` + `python-dotenv`，绝不硬编码
2. **JQL 查询** — 注意 JQL 保留字符（`*`、`?`、`[` 等）需要引号
3. **分页** — 超过 100 条必须分页遍历
4. **字段选择** — 用 `fields="*all"` 获取全部字段，`expand="changelog"` 获取变更历史
5. **增量同步** — 按 `updated >= "日期"` 过滤，避免全量重复拉取

---

*本文中的代码示例均使用模拟域名和凭据，实际运行时请替换为你自己的 JIRA 配置。*










