---
title: 地球Online开发实例
description: 本文介绍一些地球Online开发的实例，给自己开发提供思路。
date: 2026-10-12T12:34:25+08:00
draft: false
categories:
- GitHub
- Game
tags:
- Earth Online
---


# Earth Online Game 




## 商业化实践 ETO

https://earthonline.org.cn/

### 简介

“地球Online”这一创意及其相关表达来自广大粉丝、网友长期以来的共同讨论、传播和丰富，具有鲜明的网络文化与集体创作背景。我们非常尊重这一创意来源，不主张将“把现实世界理解或描述为网络游戏”等抽象构思、网络热梗、通用设定、常见玩法、公共话题或网友已有表达据为平台独有，也无权限制任何人围绕相同题材进行独立创作，但我们不会接受未经许可的大规模代码复制、素材挪用及整体性仿制。

感谢所有用户、开发者和关注者的支持。相关证据和处理进展将在符合法律要求、且不影响后续程序的前提下继续公布。

地球在线(成都)大数据科技有限公司EARTH.ONLINE 项目组

### 页面展示

![](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20261012140533892.png)
本任务期奖励"创世水晶"为限定纪念道具，无任何市场价值。  
它只是"这一期我们一起做成了一件事"的证据。
### 进展

地球Online 完成 C1 轮融资 840000 元

本轮由「ETO共投资本」领投，全部资金注入《核心着陆区》研发。

## 玩梗地球online——🌍 地球Online

https://github.com/zmccyy/earth-online

> 一款**强制登录、无法退游、不支持存档**的网页版人生 MMORPG。 投胎即开号,家境是开局难度,高考是新手村最终 Boss,上班是每日任务, 内卷是 PVP 排位,躺平是挂机,买房是毕业装备,死亡是注销账号。

基于"地球online"网络梗重制的 Web 游戏版,前后端分离:FastAPI 游戏引擎 + 原生 JS 前端。

### 🎮 玩法

1. **投胎**:随机分配姓名/性别/家境(牛马→拆迁户五档)/出生地/初始面板,玄学重抽次数有限。
2. **活过每一年**:属性会随年龄成长与老化,收支自动结算,随机事件与人生里程碑轮番上阵。
3. **做选择**:几乎每个事件都有多个选项——卷还是躺、氪还是省、组队还是单排。
4. **通关结算**:死亡后生成人生评分(SSS 天选剧本 → D 删号重练)、人生走马灯、好评差评。
5. **成就图鉴**:29 枚梗味徽章跨局持久化(存 localStorage),目标:全收集。

### 梗点速查

|系统|梗|
|---|---|
|投胎|角色创建/抽卡,氪金玩家与拆迁户是 SSR/UR 开局|
|高考|新手村最终 Boss,700 分触发"学神本神"|
|找工作|考编(宇宙的尽头)/大厂福报/小公司/继承家业/摆摊|
|买房|毕业装备,全款 or 三十年分期订阅|
|生娃|练小号,无底洞氪金项目|
|内卷/躺平|PVP 排位赛 / 挂机大师|
|钓鱼/广场舞|退休服两大主流玩法|
|灵魂觉醒|50 岁后低概率触发的**真结局:通关**|
|孟婆来电|梦境彩蛋:"你那个账号当时可以充值的"|

### 🗂️ 项目结构

```
backend/
  main.py            FastAPI 入口(路由 + 静态托管)
  api.py             REST API 契约层
  session.py         内存会话存储
  engine/            纯逻辑引擎(不依赖 Web 框架,可独立单测)
    character.py     投胎系统
    life.py          年度推进/阶段/死亡
    events.py        条件引擎 + 事件调度 + 效果结算
    achievements.py  成就触发
    rating.py        人生评分与结算
  content/           数据驱动内容(事件57+ / 成就29 / 风味文案)
frontend/            无构建原生 JS 前端(五屏 + 图鉴)
tests/               pytest 引擎单测 + API 集成测试
docs/ARCHITECTURE.md 架构决策与机制说明
```

### 🛠️ 技术栈

Python 3.10+ / FastAPI / Pydantic / 原生 ES Modules / pytest

## 纯前端人生模拟——地球Online

[将现实生活比喻为大型多人在线角色扮演游戏(MMORPG)的Web应用！](https://github.com/Stone5618/earth-online)

一个纯前端的人生模拟游戏，使用 React + TypeScript + Vite 构建。

### 技术栈

- 前端框架：React 19 + TypeScript
- 构建工具：Vite
- 样式：Tailwind CSS
- 动画：Framer Motion + GSAP
- 状态管理：React useReducer + Context API
- UI 组件：Radix UI (shadcn/ui)
- 音效：Howler.js
- 图标：Lucide React

### 项目结构

```
app/
├── public/              # 静态资源
│   └── images/
├── src/
│   ├── components/
│   │   ├── game/       # 游戏组件
│   │   └── ui/         # UI 组件
│   ├── config/         # 游戏配置
│   ├── game/           # 游戏核心逻辑
│   ├── sections/       # 页面区域
│   ├── types/          # 类型定义
│   ├── App.tsx
│   └── main.tsx
├── package.json
├── tsconfig.json
├── vite.config.ts
└── ...其他配置
```

### 主要功能

- 出生系统：随机家庭背景、天赋/缺陷
- 职业系统：多行业、晋升机制
- 教育系统：从小学到博士的教育路径
- 事件系统：随机人生事件
- 健康系统：健康管理、疾病治疗
- 经济系统：收入、支出、债务管理
- 技能系统：14种可升级技能
- 存档系统：自动/手动存档
- 成就系统：游戏成就解锁

## 地球Online - 人生成就系统 🌍

[地球Online人生成就系统](https://github.com/LKlingkong/EarthOnline-Achievement)

把人生当作一款 MMORPG，记录你解锁的每一个成就。

[![地球Online](https://camo.githubusercontent.com/03212f4d323a11e84376ae8c88ac5be77b7f0ffe616d1ec11fee048ec540cfc7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2545372538392538382545362539432541432d76323032362e30362d676f6c64)](https://camo.githubusercontent.com/03212f4d323a11e84376ae8c88ac5be77b7f0ffe616d1ec11fee048ec540cfc7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2545372538392538382545362539432541432d76323032362e30362d676f6c64) [![HTML5](https://camo.githubusercontent.com/df6013a179820b423dc89f0110703765c77fe130dc6b6f9d9efab67b2352cb10/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2545362538412538302545362539432541462d25453725424125414648544d4c352d6f72616e6765)](https://camo.githubusercontent.com/df6013a179820b423dc89f0110703765c77fe130dc6b6f9d9efab67b2352cb10/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2545362538412538302545362539432541462d25453725424125414648544d4c352d6f72616e6765) [![无依赖](https://camo.githubusercontent.com/9653de742d19fea4cbf7c6575402d03928509b85b110b456ea34be2aee59d424/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2545342542452539442545382542352539362d2545362539372541302d627269676874677265656e)](https://camo.githubusercontent.com/9653de742d19fea4cbf7c6575402d03928509b85b110b456ea34be2aee59d424/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2545342542452539442545382542352539362d2545362539372541302d627269676874677265656e)

### ✨ 功能特性

- **60 个精心设计的成就**，涵盖学业、职业、情感、生活、旅行、财富等 12 个类别
- **4 级稀有度**：普通 → 稀有 → 史诗 → 传说，越稀有越难达成
- **暗色主题 + 金色高亮**：已解锁成就金色闪耀，未解锁成就灰色暗淡
- **按难度自动排序**：传说成就排最前，普通成就排最后
- **点击查看详情**：每个成就都有专属故事描述
- **搜索与筛选**：支持关键词搜索，按分类筛选
- **本地存档**：解锁进度自动保存到浏览器 localStorage
- **导出 / 导入**：可将成就进度导出为 JSON 文件，分享给朋友导入
- **重置进度**：一键清除本地存档，恢复初始状态
- **完全离线**：纯前端实现，无需服务器，打开即用

### 🎮 使用说明

1. **浏览成就**：滚动页面查看所有 60 个人生成就
2. **解锁成就**：点击任意成就卡片 → 在弹窗中点击「✨ 解锁此成就」
3. **筛选分类**：点击顶部分类标签（学业 / 职业 / 情感等）
4. **搜索**：在搜索框输入关键词，快速定位成就
5. **按难度排序**：点击「解锁难度」按钮切换升序 / 降序
6. **导出进度**：点击「📥 导出」将存档保存为 JSON 文件
7. **导入进度**：点击「📤 导入」选择 JSON 存档文件
8. **重置**：点击「🔄 重置」清除本地所有进度

### 📊 成就稀有度

|稀有度|达成率范围|颜色|
|---|---|---|
|🏆 传说|< 10%|金色|
|🌟 史诗|10% ~ 20%|紫色|
|🔷 稀有|20% ~ 50%|蓝色|
|· 普通|> 50%|灰色|

### 🛠️ 技术栈

- 纯 HTML5 + CSS3 + Vanilla JavaScript
- 无任何外部依赖
- 数据持久化：localStorage
- 单文件，可完全离线运行
## 地球Online · 中国

14亿玩家同时在线。出生点遍布山河四省，每一个抉择都将塑造独一无二的命运线。

https://stellarcn2020.github.io/Earth-Online/
### 开放世界 · 永久在线

地球Online是一款大型多人在线现实模拟游戏，由熵增与进化法则联合开发，运营已超46亿年。当前中国区在线玩家14亿+，每位玩家拥有独一无二的生命轨迹。

### 核心玩法

探索、建造、交易、社交、传承……无固定主线，每个抉择都实时改变世界状态。技能树高度自由，隐藏职业多达数百种。

### 角色创建

出生地随机分配至34个省级行政区，初始天赋受地域影响。后期可通过学习与历练重塑属性。


## 地球Online 3D Web

https://a330900neo.github.io/EarthOnlinememe/

![](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20261012140933124.png)

## 地球Online 文字版 Web

[https://a330900neo.github.io/EarthOnlinememe/](https://pgy521.github.io/-online/)

![](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20261012141153338.png)

## 🌍 地球Online指南

[地球Online指南：从蒸米饭到坐高铁——写给「第一次」的全场景生活技能手册](https://github.com/PensiveFei/earth-online-guide)

> 从蒸米饭到坐高铁——写给「第一次」的全场景生活技能手册。

[![地球Online指南：写给「第一次」的全场景生活技能手册](https://github.com/PensiveFei/earth-online-guide/raw/main/docs/public/og/home.jpg)](https://eartholguide.com/)

**在线阅读：[https://eartholguide.com/](https://eartholguide.com/)**

本项目是一份面向「地球服新手玩家」的开源攻略手册。 收录现实世界各种任务流程、基础技能、避坑指南、隐藏规则。

**共 47 篇已上线**：🍚 饮食做饭 11 · 🚄 出行交通 10 · 🏠 租房搬家 7 · 🏥 就医问诊 4 · 🛒 购物防骗 5 · 🧺 日常家务 10

做饭板块刻意做「精简版」——追求进阶菜谱请移步 [HowToCook](https://github.com/Anduin2017/HowToCook)（102k★，我们致敬的同类前辈）。








