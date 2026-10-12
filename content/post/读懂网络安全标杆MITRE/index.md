---
title: 从ATT&CK到ATLAS：读懂网络安全标杆MITRE，看懂攻防框架的进化
description: 本文介绍网络安全标杆MITRE。
date: 2026-10-12T22:34:25-08:00
draft: false
categories:
- 网络安全
- 计算机
tags:
- MITRE
- ATT&CK
- ATLAS
- AI安全
---


# 从ATT&CK到ATLAS：读懂网络安全标杆MITRE，看懂攻防框架的进化

在网络安全行业，如果你做威胁狩猎、红队评估、攻防演练，几乎一定会碰到MITRE ATT&CK；而随着大模型、AI应用快速落地，它的“AI安全兄弟”——MITRE ATLAS，正在成为AI红队、LLM风险治理的新标准。这两套框架都出自同一家非营利机构：MITRE。今天我们就一起梳理MITRE、ATT&CK与ATLAS的来龙去脉，重点拆解两大框架的内核、战术、技术与落地用法。

## 一、MITRE：不止是安全框架的“幕后推手”

很多人只知道ATT&CK，但并不了解MITRE本身。MITRE（The MITRE Corporation）是一家诞生于1958年的美国非营利组织，前身是麻省理工学院林肯实验室，起源可以追溯到冷战时期为美国空军打造SAGE半自动防空系统的项目。

它的定位很特殊：作为联邦资助研发中心（FFRDC），以公共利益为导向，不和商业公司争抢市场，专门承接政府机构的前沿系统工程研发，客户包括美国国防部、国土安全部、联邦航空管理局等。业务边界也远不止网络安全，覆盖航空、公共卫生、人工智能、量子科学、空间安全等众多方向。疫情期间它推出Sara Alert™工具辅助流行病追踪；2026年，MITRE也参与了英伟达牵头、面向6G的安全AI原生平台联合倡议。

但让MITRE在全球安全圈封神的，还是它输出的一系列通用标准：
- **CVE**：漏洞编号字典，给每一个公开漏洞分配唯一ID，是全球漏洞互通的基础；
- **CWE**：通用缺陷列表，用来梳理软件底层缺陷类型，是代码安全、SAST工具的参考基线；
- **MITRE ATT&CK**：传统网络攻防行为知识库；
- **MITRE ATLAS**：面向AI系统的对抗威胁知识库。

MITRE的核心价值，就是给分散的安全行业提供一套**统一语言**：让攻防双方、厂商、甲方、研究人员，对攻击者行为有一致的描述口径。

## 二、MITRE ATT&CK：传统网络攻防的通用“行为字典”
ATT&CK全称 **Adversarial Tactics, Techniques, and Common Knowledge（对抗战术、技术与常识）**。概念诞生于2013年，2015年正式对外发布，是一套**基于真实攻击观测、持续迭代维护的公开攻防知识库**，它跳出了传统“漏洞+利用”的单点视角，站在攻击者的完整杀伤链视角，把攻击行为标准化建模。

### 2.1 ATT&CK核心概念：Tactic / Technique / Sub-technique / Procedure
这是理解ATT&CK最核心的四层结构：
1. **Tactic（战术，TA开头）：代表攻击者的目标（为什么这么做）**
    战术是攻击阶段、攻击者想要达成的目的，是分类的顶层维度；
2. **Technique（技术，T开头）：攻击者实现目标的方法（做什么）**
    描述一类攻击行为；
3. **Sub-technique（子技术，T.xxx）：细分的具体实现方式**
    同一个技术下的不同变种；
4. **Procedure（流程）：真实威胁组织、APT团伙实际使用的操作步骤**
    把技术落地到真实攻击中的实例。

ATT&CK不是线性的杀伤链，而是**矩阵**——攻击者可以跳跃、回退、重复执行战术，更贴合真实入侵的非线性特点。

### 2.2 三大矩阵领域（Matrix）
- **Enterprise（企业域）**：使用最广泛，覆盖Windows/Linux/macOS终端、云、身份、服务器、SaaS等企业IT环境；
- **Mobile（移动域）**：针对安卓、iOS手机、移动终端的攻击行为；
- **ICS（工业控制系统域）**：面向工控、电力、油气等关键基础设施，是工控安全评估、工控红队的标准参考。

Enterprise ATT&CK主流14个战术，简要说明：
1. **侦察 Reconnaissance(TA0043)**：收集目标信息，比如域名、员工邮箱、系统版本；
2. **资源开发 Resource Development(TA0042)**：搭建攻击资源，C2服务器、钓鱼页面、恶意样本；
3. **初始访问 Initial Access(TA0001)**：拿下第一个入口，钓鱼邮件、漏洞利用、弱口令；
4. **执行 Execution(TA0002)**：在目标机器运行恶意代码；
5. **持久化 Persistence(TA0003)**：重启后依然保有访问权限，计划任务、注册表、服务；
6. **权限提升 Privilege Escalation(TA0004)**：从普通用户拿到管理员权限；
7. **防御规避 Defense Evasion(TA0005)**：绕过EDR、防火墙、日志审计，删日志、混淆代码；
8. **凭据访问 Credential Access(TA0006)**：窃取账号密码、Token、Hash；
9. **发现 Discovery(TA0007)**：在内网踩点，扫描网段、查看进程、用户列表；
10. **横向移动 Lateral Movement(TA0008)**：从一台机器跳到另一台；
11. **收集 Collection(TA0009)**：抓取文档、截图、数据库数据；
12. **命令与控制 Command and Control(TA0011)**：被控主机和攻击者C2通信；
13. **数据渗出 Exfiltration(TA0010)**：把窃取的数据外传；
14. **影响 Impact(TA0040)**：勒索加密、破坏系统、业务中断。

### 2.3 版本迭代与生态工具
ATT&CK是**社区驱动的活态知识库**，常规每半年发布新版本，吸纳全球安全社区上报的真实攻击案例：
- v14（2023）：新增社会工程、“人类漏洞”相关非技术攻击；
- 截至2025更新至v18；
- 2026年MITRE成立ATT&CK顾问委员会，长期治理框架演进。

配套工具生态：
- **ATT&CK Navigator**：最常用的可视化矩阵工具，可以标记检测覆盖、红队测试范围，导出矩阵图；
- **ATT&CK Workbench**：本地部署，自定义矩阵、维护内部威胁库；
- **ATT&CK Evaluations（ATT&CK评估）**：第三方厂商的安全产品能力测评，看EDR、SIEM能覆盖多少ATT&CK技术。

### 2.4 ATT&CK典型落地场景
红队方案设计、SOC检测规则建设、威胁狩猎、事件响应复盘、安全产品测评、甲方防御能力矩阵评估、威胁情报标准化描述。

> 一句话概括ATT&CK：它描述**攻击者在传统IT基础设施上会做哪些动作**，让防御者从“盯漏洞”升级为“盯攻击者行为”。

## 三、MITRE ATLAS：面向AI系统的专属ATT&CK
随着大模型、机器学习、AI智能体大规模上线，传统ATT&CK只能描述入侵服务器、窃取API密钥这类底层IT攻击，**无法描述针对模型本身的AI原生攻击**：提示注入、模型投毒、对抗样本、模型窃取、越狱等。于是MITRE推出 **ATLAS（Adversarial Threat Landscape for Artificial-Intelligence Systems）**，是专门针对AI/ML系统的对抗威胁知识库。

ATLAS完全继承ATT&CK的设计范式：同样是Tactic-Technique-Subtechnique的矩阵结构、同样基于真实攻击事件+AI红队测试案例，作为ATT&CK的补充框架，**而不是替代**。很多真实攻击是组合链路：攻击者先用ATT&CK的方式拿下训练服务器，再执行ATLAS里的模型投毒。

### 3.1 ATLAS的定位与16个战术
ATLAS一共16个战术，大部分继承自ATT&CK，额外增加两个AI独有的战术：**ML Model Access（模型访问）、ML Attack Staging（AI攻击筹备）**，完整覆盖AI全生命周期：数据采集、训练、微调、部署推理、模型调用。

![](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20261012231926018.png)

ATLAS典型AI攻击技术（AML开头编号）：
- 提示注入 Prompt Injection：直接在输入里诱导LLM绕过安全护栏、泄露提示词；
- 越狱 Jailbreak：绕过模型对齐、安全限制，生成有害内容；
- 数据投毒 Data Poisoning：污染训练集，让模型出现定向错误；
- 模型投毒 Model Poisoning：后门植入到权重，特定触发词触发异常输出；
- 对抗样本 Adversarial Example：给图像/音频加微小扰动，让CV模型误识别；
- 模型窃取/提取 Model Extraction：通过API查询，复刻出近似模型；
- 训练数据泄露 Training Data Exfiltration：从模型里还原训练数据、隐私信息；
- 模型反转 Model Inversion：利用模型输出反推训练集中的敏感样本。

### 3.2 ATLAS的应用方式
ATLAS主要服务AI红队、大模型安全评估、AI风险建模：
1. 给AI红队提供标准化测试用例库，定义攻击编号、步骤、检测思路；
2. 和OWASP LLM Top10、NIST AI RMF、ISO 42001组合使用：
   - OWASP：梳理LLM应用层风险优先级；
   - ATLAS：对AI对抗行为建模、红队测试；
   - NIST AI RMF / ISO42001：整体AI安全治理、风险评估；
3. 微软开源Counterfit工具，可自动化执行ATLAS收录的各类AI对抗攻击，批量验证模型鲁棒性。

配套衍生框架AERS，专门用来评估ATLAS在防御数据投毒场景下的落地效果，把ATLAS战术映射到杀伤链，给AI防御做定性评级。
ATLAS官网：[https://atlas.mitre.org](https://atlas.mitre.org)，开源、社区持续更新，包含大量真实AI攻击案例。

> 一句话概括ATLAS：它描述**攻击者针对AI模型、训练流水线、模型推理能力本身的攻击行为**，补齐了传统ATT&CK在生成式AI、机器学习安全上的空白。

## 四、ATT&CK vs ATLAS：详细对比表
| 项目 | MITRE ATT&CK | MITRE ATLAS |
| ---- | ---- | ---- |
| 全称 | Adversarial Tactics, Techniques, and Common Knowledge | Adversarial Threat Landscape for Artificial-Intelligence Systems |
| 保护对象 | 终端、服务器、云、账号、工控、移动设备（IT/OT基础设施） | LLM、CV模型、机器学习流水线、AI智能体、训练数据集 |
| 核心关注点 | 入侵、内网移动、持久化、数据窃取、系统破坏 | 模型操纵、提示注入、投毒、对抗样本、模型窃取、越狱 |
| 战术数量 | Enterprise共14个战术 | 16个战术（14个继承ATT&CK + 2个AI专属战术） |
| 编号前缀 | T（技术）、TA（战术） | AML-T（AI攻击技术） |
| 起源时间 | 2013构思，2015正式发布 | AI安全兴起后推出，对标ATT&CK体系 |
| 典型使用者 | SOC蓝队、传统红队、威胁情报、工控安全工程师 | AI安全工程师、LLM红队、模型风控、AI合规治理岗 |
| 二者关系 | 基础框架，描述底层IT入侵 | 扩展框架，描述AI原生攻击；攻击链路经常组合使用 |

## 五、写在最后：框架不是银弹，是通用语言
很多新手容易把ATT&CK或者ATLAS当成“一键防护工具”，这是一个常见误区。它们本质上是**分类体系、威胁词汇表、攻击行为参考库**，本身不会帮你直接检测攻击。它的价值在于统一认知：
- 红队写报告，用ATT&CK/ATLAS编号描述攻击步骤，蓝队一眼就能理解；
- 建设检测能力时，可以对照矩阵，排查自己哪些战术场景缺少检测规则；
- 评估大模型安全时，对照ATLAS梳理投毒、提示注入等风险点，完成红队测试方案。

从冷战时期的防空系统，到CVE、CWE，再到ATT&CK，如今延伸到AI安全领域的ATLAS，MITRE一直在做一件事：把复杂、零散的攻击者行为标准化。在AI快速普及的当下，ATLAS正在复刻当年ATT&CK的路径，慢慢成为生成式AI安全领域的通用基线。

如果你正在做SOC建设、红队评估、LLM安全测试，不妨从Navigator和ATLAS官网入手，把这两套框架放进自己的安全流程里。










