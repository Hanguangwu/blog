---
title: 寻觅被遗忘的网络账号
description: 本文介绍一些网络账号泄露情况检测工具。
date: 2026-03-06T12:34:25-08:00
draft: false
categories:
- APP
- 网络安全
tags:
- 社工库
- OSINT
---


# 寻觅被遗忘的网络账号

## 前言

**开源情报（Open Source Intelligence，OSINT）** 是指通过公开渠道收集和分析信息的过程。它涉及从可公开访问的来源（如社交媒体、新闻报道、政府档案等）收集数据，以帮助决策、调查或研究。OSINT的应用广泛，包括政府监测、商业竞争情报和安全分析等领域。通过合法的方式收集信息，企业可以更深入地了解市场和竞争对手，从而做出明智的决策。 

[Hunt down social media accounts by username across social networks](https://github.com/sherlock-project/sherlock)

[WhatsMyName-Advanced whatsmyname OSINT tool for safe and comprehensive digital intelligence gathering. Free professional username analysis across 500+ platforms for security research, cyber investigations, and digital forensics.](https://github.com/WebBreacher/WhatsMyName)

缺点：几乎没有简中平台数据

这两个网站都是临时请求已收录的网站获知情况，而不是像Telegram中的「开盒」数据一样。

有空可以做一个中文版！

## 社工库 haveibeenpwned

网安：haveibeenpwned(社工库)查看信息是否被泄露

[Have I Been Pwned: Check if your email has been compromised in a data breach](https://haveibeenpwned.com/)

通过以上网址可以到达官网。

可以输入手机号或者邮箱查看信息是否被泄露，如果有被泄露的，查看怎么被泄露的。最后赶快修改关键信息，免得撞库。

撞库：比如你在优酷的信息泄露了，这个信息的密码是你qq账号的密码，攻击者通过这个优酷得到的密码进入了你的qq。这就叫撞库。

## 密码泄露检测工具 | 撞库风险即时查询

[密码泄露检测工具 | 撞库风险即时查询](https://www.freetoolhouse.com/secret-tool/pass-pwned)

通过HIBP官方数据库，检测您的密码是否出现在已知数据泄露事件中

k-Anonymity隐私保护：仅传输哈希前缀，完整密码永不离开设备

### 关于密码泄露检测与撞库攻击

#### 什么是密码泄露检测工具？

密码泄露检测工具（Pwned Password Checker）是一种在线安全工具，用于验证您的密码是否曾在公开的数据泄露事件中出现过。本工具通过调用Have I Been Pwned (HIBP) 官方API，实时查询超过7亿个已泄露密码的数据库，帮助您评估密码安全性，避免因使用已泄露密码而导致撞库攻击风险。

与传统的密码强度检测不同，泄露检测关注的是"实际已泄露"的密码，而非仅评估复杂度，因此能更真实地反映密码在当前网络安全环境下的实际风险等级。

#### 什么是撞库攻击？

撞库攻击（Credential Stuffing）是黑客常用的攻击手段之一。攻击者利用用户在多个平台重复使用同一密码的习惯，将从某网站泄露的账号密码组合，批量尝试登录其他网站。由于许多人习惯使用相同的密码，一旦某个平台的密码泄露，其他平台的账号安全也随之受到威胁。

根据网络安全机构的统计，超过65%的用户会在不同平台重复使用密码，这使得撞库攻击的成功率远高于传统暴力破解。这也是为什么即使您的密码足够复杂，一旦出现在泄露库中，就必须立即更换的根本原因。

#### k-Anonymity 隐私保护技术

为了解决隐私顾虑，本工具采用了k-Anonymity（k-匿名）技术，这是由安全专家Junade Ali设计的一种隐私保护模型。其工作原理是：

- 计算密码SHA-1哈希值：如"5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8"
- 分割哈希：取前5位"5baa6"发送给API服务器
- 服务器返回所有以"5baa6"开头的哈希后缀列表（通常包含数百个结果）
- 客户端在本地比对完整哈希是否存在于返回列表中

这种设计确保API服务器无法获知您查询的具体密码，因为哈希前缀对应着海量可能的密码组合，而完整哈希值始终保存在您本地。即使通信被截获，攻击者也只能看到哈希前缀，无法反向推导出原始密码。

#### HIBP数据库：全球最大的密码泄露库

Have I Been Pwned (HIBP) 由安全专家Troy Hunt创建，是目前全球最大的公开数据泄露查询平台。该数据库收录了来自数百起数据泄露事件的超过7亿个真实密码，涵盖Adobe、LinkedIn、MySpace、Yahoo等重大安全事件。HIBP被微软、美国政府等机构广泛采用，是公认的密码泄露权威数据源。

本工具通过官方API与HIBP数据库实时同步，确保查询结果的时效性和准确性。所有查询均遵循k-Anonymity协议，在保护用户隐私的同时提供专业的密码安全检测服务。

#### 密码安全最佳实践

使用密码管理器：通过1Password、Bitwarden等工具生成并存储高强度随机密码，避免重复使用和记忆负担

启用多因素认证：即使密码泄露，MFA也能阻止未授权访问，建议优先使用TOTP或硬件密钥

定期检测泄露：每季度使用本工具检测关键密码，发现泄露立即更换，并检查相关账号异常活动

#### 关于SHA-1哈希算法

本工具采用SHA-1算法对密码进行哈希处理，这是HIBP API的标准要求。虽然SHA-1在数字签名领域已被认为不够安全，但在密码泄露查询场景中，由于我们仅使用其单向哈希特性且配合k-Anonymity机制，即使SHA-1存在碰撞理论可能性，也不会影响密码查询的安全性。实际场景中，从哈希前缀反向破解原始密码的计算复杂度仍然极高，远超过实际攻击成本。

## Sherlock

[Hunt down social media accounts by username across social networks](https://github.com/sherlock-project/sherlock)
### 安装

This is an officially supported package, maintained by a member of the Sherlock Project itself.

[pipx](https://pipx.pypa.io/) is often recommended over pip, having more predictable behavior.

```
pipx install sherlock-project
```

For those who prefer classic pip, it’s very similar. Userspace is recommended.

```
pip install --user sherlock-project
```

That’s it! You can now run sherlock from anywhere.

```
sherlock --version
```

### 用法

Search for only one user:

```
sherlock user123
```

Search for multiples users:

```
sherlock user1 user2 user3
```

Accounts found will be stored in an individual text file with the corresponding username (e.g `user123.txt`).

### 代码解析

#### 跨越Web应用防火墙

```
		# As WAFs advance and evolve, they will occasionally block Sherlock and
        # lead to false positives and negatives. Fingerprints should be added
        # here to filter results that fail to bypass WAFs. Fingerprints should
        # be highly targetted. Comment at the end of each fingerprint to
        # indicate target and date fingerprinted.
        WAFHitMsgs = [
            r'.loading-spinner{visibility:hidden}body.no-js .challenge-running{display:none}body.dark{background-color:#222;color:#d9d9d9}body.dark a{color:#fff}body.dark a:hover{color:#ee730a;text-decoration:underline}body.dark .lds-ring div{border-color:#999 transparent transparent}body.dark .font-red{color:#b20f03}body.dark', # 2024-05-13 Cloudflare
            r'<span id="challenge-error-text">', # 2024-11-11 Cloudflare error page
            r'AwsWafIntegration.forceRefreshToken', # 2024-11-11 Cloudfront (AWS)
            r'{return l.onPageView}}),Object.defineProperty(r,"perimeterxIdentifiers",{enumerable:' # 2024-04-09 PerimeterX / Human Security
        ]
```

#### update-site-list.yml

```yml
name: Update Site List

# Trigger the workflow when changes are pushed to the main branch
# and the changes include the sherlock_project/resources/data.json file
on:
  push:
    branches:
      - master
    paths:
      - sherlock_project/resources/data.json

jobs:
  sync-json-data:
    # Use the latest version of Ubuntu as the runner environment
    runs-on: ubuntu-latest

    steps:
      # Check out the code at the specified pull request head commit
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
          fetch-depth: 0

      # Install Python 3
      - name: Install Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.x'

      # Execute the site_list.py Python script
      - name: Execute site-list.py
        run: python devel/site-list.py

      - name: Pushes to another repository
        uses: sdushantha/github-action-push-to-another-repository@main
        env:
          SSH_DEPLOY_KEY: ${{ secrets.SSH_DEPLOY_KEY }}
          API_TOKEN_GITHUB: ${{ secrets.API_TOKEN_GITHUB }}
        with:
          source-directory: 'output'
          destination-github-username: 'sherlock-project'
          commit-message: 'Updated site list'
          destination-repository-name: 'sherlockproject.xyz'
          user-email: siddharth.dushantha@gmail.com
          target-branch: master
```

提交到另一个仓库：`push-to-another-repository`

#### 增减数据——site-list.py

```python
#!/usr/bin/env python
# This module generates the listing of supported sites which can be found in
# sites.mdx. It also organizes all the sites in alphanumeric order
import json
import os

DATA_REL_URI: str = "sherlock_project/resources/data.json"

DEFAULT_ENCODING = "utf-8"

# Read the data.json file
with open(DATA_REL_URI, "r", encoding=DEFAULT_ENCODING) as data_file:
    data: dict = json.load(data_file)

# Removes schema-specific keywords for proper processing
social_networks = data.copy()
social_networks.pop('$schema', None)

# Sort the social networks in alphanumeric order
social_networks = sorted(social_networks.items())

# Make output dir where the site list will be written
os.mkdir("output")

# Write the list of supported sites to sites.mdx
with open("output/sites.mdx", "w", encoding=DEFAULT_ENCODING) as site_file:
    site_file.write("---\n")
    site_file.write("title: 'List of supported sites'\n")
    site_file.write("sidebarTitle: 'Supported sites'\n")
    site_file.write("icon: 'globe'\n")
    site_file.write("description: 'Sherlock currently supports **400+** sites'\n")
    site_file.write("---\n\n")

    for social_network, info in social_networks:
        url_main = info["urlMain"]
        is_nsfw = "**(NSFW)**" if info.get("isNSFW") else ""
        site_file.write(f"1. [{social_network}]({url_main}) {is_nsfw}\n")

# Overwrite the data.json file with sorted data
with open(DATA_REL_URI, "w", encoding=DEFAULT_ENCODING) as data_file:
    sorted_data = json.dumps(data, indent=2, sort_keys=True)
    data_file.write(sorted_data)
    data_file.write("\n")  # Keep the newline after writing data

print("Finished updating supported site listing!")
```

先复制数据再写入
## WhatsMyName

### WhatsMyName 是什么？

WhatsMyName 是 Micah "WebBreacher" Hoffman 在 2015 年创建的一个项目，其目标是在给定网站上检测用户名是否被使用。 他对当时用户名检查器的误报感到沮丧，因此他创建了自己的版本。 随着时间的推移，许多人帮助这个开源项目发展，使其成为现在。

如果您是进行此操作的 OSINT 专家，那么您现在可能会感到有些失望。 在 2023 年 5 月，我们从项目中移除了所有检查脚本，并专注于该项目的核心：其数据文件 (wmn-dat.json)。

因此，我们将继续发现网站并添加它们，您可以自由尝试以下使用我们数据的检查网站和脚本。

[WhatsMyName-Advanced whatsmyname OSINT tool for safe and comprehensive digital intelligence gathering. Free professional username analysis across 500+ platforms for security research, cyber investigations, and digital forensics.](https://github.com/WebBreacher/WhatsMyName)
### 它是如何工作的？

WhatsMyName (WMN) 包含一个 JSON 文件，其中包含检测结果。 包含来自世界各地的人的提交。 当一个工具（如下一部分中的工具）从这些网站上发起请求时，服务器会返回与我们检测结果相匹配的数据。 这将告诉检查脚本，该网站上是否存在我们指定的用户名，或者不是。

为了让一个网站包含在 WMN 中，它必须：

- 可访问。 我们无法检查需要付费或进行用户身份验证的网站。
- 在 URL 中包含用户名。 如果查看用户个人资料的 URL 中不包含该用户名，则此工具将不起作用。
- 不要在 URL 中修改用户名。 包含用户 ID 编号在用户名的 URL 将无法在 WMN 中使用。 此外，将您的用户名映射到用户 ID 编号并将其放在 URL 中的网站也无法使用。

### 使用 WhatsMyName 的工具/网站

- [https://whatsmyname.app/](https://whatsmyname.app/) - Chris Poulter 创建了这个网站，它将项目的 JSON 文件转换为易于使用的 Web 界面。
    - 按类别和搜索结果过滤。
    - 导出到 CSV 和其他格式。
    - 在运行时，获取项目的最新 JSON 文件。
    - 使用 [https://whatsmyname.app/?q=USERNAME](https://whatsmyname.app/?q=USERNAME) (例如，[https://whatsmyname.app/?q=john](https://whatsmyname.app/?q=john)) 在 URL 中提交用户名。
- Who Am I Google/Brave 扩展，不仅集成 WhatsMyName 的数据，还包括 Sherlock 和 Maigret 用户名检查器。 由 OSINT Liar 创建。
- Naminter - 专为 Whats My Name 列表而设计的，具有漂亮的控制台界面、浏览器冒充功能、能够绕过 Cloudflare 和其他基本保护、并发检查以及广泛的配置选项。
- Blackbird - 在其搜索中使用 WhatsMyName 列表。
- K2OSINT Bookmarks - Bookmarks，允许您在弹出窗口中输入用户名，然后在新标签页中打开 WMN 结果。
- LinkScope - 在 "在线身份" 类别下使用 WhatsMyName 进行解析。
- Maltego WhatsMyName 转换 - Maltego Local 转换，利用 JSON 文件并实时检查用户名。
- Reveal My Name - 由 @yooper 创建，是与此项目捆绑的 Python 检查器工具。
- sn0int - 下载并使用 kpcyrd/whatsmyname 模块中的 JSON 文件，详情和说明请参考 [https://twitter.com/sn0int/status/1228046880459907073。](https://twitter.com/sn0int/status/1228046880459907073%E3%80%82)
- Spiderfoot - 在 sfp_account 模块中使用。 还有一段视频演示如何使用 Spiderfoot 命令行界面 (CLI) 使用该项目。
- WhatsMyName-Python - 由 @C3n7ral051nt4g3ncy 创建的简单的 Python 脚本。
- WMN_screenshooter - 一个辅助脚本，使用 Selenium 尝试抓取识别出的个人资料页面截图。
- WhatsMyName-Client - 由 @grabowskiadrian 创建的简单的 Python 脚本，具有“请求头”和“POST 请求”支持。 该脚本还允许您测试 wmn-data.json 文件的配置。
- WhatsMyName-Web - 由 @AXRoux 创建的 WhatsMyName-Web 是 WhatsMyName 的一个简单的 Flask Web 应用程序。
- WhatsMyName Docker - 这是一个 Docker API 包装器，用于 WhatsMyName 工具。 Docker 化由 @kodamaChameleon 完成。
- NameSeeker - 是一款强大的跨平台桌面应用程序，可以搜索数百个网站上的用户名和电子邮件地址，帮助您快速发现您的数字足迹。 它基于 WhatsMyName 项目的数据，支持将搜索结果导出到 PDF、CSV、JSON 和其他格式。

### 代码解析

#### 排序Python代码

```python
import json

data_path = 'wmn-data.json'
schema_path = 'wmn-data-schema.json'

def sort_array_alphabetically(arr):
    return sorted(arr, key=str.lower)

def reorder_object_keys(obj, key_order):
    reordered = {k: obj[k] for k in key_order if k in obj}
    for k in obj:
        if k not in key_order:
            reordered[k] = obj[k]
    return reordered

def sort_headers(site):
    headers = site.get("headers")
    if isinstance(headers, dict):
        site["headers"] = dict(sorted(headers.items(), key=lambda item: item[0].lower()))

def load_and_format_json(path):
    with open(path, 'r', encoding='utf-8') as f:
        raw_content = f.read()
        data = json.loads(raw_content)
    formatted = json.dumps(data, indent=2, ensure_ascii=False)
    return data, raw_content, formatted

data, data_raw, data_formatted = load_and_format_json(data_path)
schema, schema_raw, schema_formatted = load_and_format_json(schema_path)

changed = False

# Sort authors and categories
if isinstance(data.get('authors'), list):
    data['authors'] = sort_array_alphabetically(data['authors'])

if isinstance(data.get('categories'), list):
    data['categories'] = sort_array_alphabetically(data['categories'])

# Sort and reorder sites
site_schema = schema.get('properties', {}).get('sites', {}).get('items', {})
key_order = list(site_schema.get('properties', {}).keys())

if isinstance(data.get('sites'), list):
    data['sites'].sort(key=lambda site: site.get('name', '').lower())
    for site in data['sites']:
        sort_headers(site)
    data['sites'] = [reorder_object_keys(site, key_order) for site in data['sites']]

updated_data_formatted = json.dumps(data, indent=2, ensure_ascii=False)

# Write wmn-data.json if changed
if data_raw.strip() != updated_data_formatted.strip():
    with open(data_path, 'w', encoding='utf-8') as f:
        f.write(updated_data_formatted)
    print("Updated and sorted wmn-data.json.")
    changed = True
else:
    print("wmn-data.json already formatted.")

# Write formatted wmn-data-schema.json if changed
if schema_raw.strip() != schema_formatted.strip():
    with open(schema_path, 'w', encoding='utf-8') as f:
        f.write(schema_formatted)
    print("Formatted wmn-data-schema.json.")
    changed = True
else:
    print("wmn-data-schema.json already formatted.")

if not changed:
    print("No changes made.")

```

#### 排序工作流

```yml
name: Sort and Format JSON Files

on:
  push:
    paths:
      - 'wmn-data.json'
      - 'wmn-data-schema.json'

jobs:
  sort-and-format-json:
    name: Sort and Format JSON Files
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Run Python sorting script
        run: python scripts/sort_format_json.py

      - name: Commit if changed
        run: |
          git config --global user.name "github-actions"
          git config --global user.email "github-actions@github.com"
          git add wmn-data.json wmn-data-schema.json
          git diff --cached --quiet || git commit -m "chore: auto-sort and format JSON files"
          git push

```

#### Docker镜像验证JSON

```yml
name: Validate JSON
on: [pull_request]
jobs:
  verify-json-validation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Validate JSON
        uses: docker://orrosenblatt/validate-json-action:latest
        env:
          INPUT_SCHEMA: /wmn-data-schema.json
          INPUT_JSONS: /wmn-data.json

```

还能这么玩，哈！

## 失效的中文OSINT FRAMEWORK

[osint-framework](https://github.com/B1gM8c/osint)

Osint Framework（开源情报查询框架）即通过在互联网中公开的信息进行一些事件的调查，它的优势显而易见就是不会与目标主体产生接触、关联，意味着你的调查将不会被他发现，这是不是听起来蛮有意思的！

### 功能介绍

功能大纲包括

- 用户名搜索
- 邮箱账户搜索
- 域名搜索
- IP地址搜索
- 图片/视频/文档搜索
- 社交网站搜索
- 即时通讯
- 身份信息核实
- 约会记录查询
- 电话号码查询
- 公开档案
- 商业档案
- 交通运输
- 高精度定位/地图
- 搜索引擎玩法
- 论坛/博客/IRC
- 档案资料
- 语言翻译
- 元数据
- 手机模拟器
- 数字货币
- 分类信息
- 编码/解码
- 工具
- 恶意文件分析
- 漏洞EXP & 修复建议
- 威胁情报
- OpSec
- 文档资料
- 训练

## 附录

### 开源情报分析（OSINT）实战指南：从基础到高级的信息收集技术

https://xz.aliyun.com/news/17607

在数字化时代，公开信息中隐藏着大量有价值的情报，而开源情报分析（OSINT）正是挖掘这些信息的关键技术。无论是网络安全竞赛（CTF）、渗透测试，还是企业安全审计，掌握OSINT技能都能让你更高效地收集目标数据。本文是一份全面且实用的OSINT技术指南，涵盖从基础到高级的信息收集方法，包括：

- 搜索引擎高级语法（Google、Bing、DuckDuckGo）
- 反向图片搜索（Google Images、Yandex、TinEye）
- EXIF数据解析（地理位置、拍摄设备信息）
- 社交媒体情报分析（Twitter、Facebook、Instagram、Snapchat）
- 电子邮件与用户名追踪（Hunter.io、Phonebook.cz、HaveIBeenPwned）
- 密码泄露检测（Dehashed、WeLeakInfo）
- 匿名身份（马甲）构建（虚拟电话、加密邮箱、VPN、虚拟机）

此外，文章还提供了数十个实用工具和网站，帮助你在合法合规的前提下高效完成情报收集任务。无论你是安全研究员、OSINT爱好者，还是CTF选手，这篇指南都能助你提升信息挖掘能力。

在文章最后，会实战分析一道OSINT类型的题目。










