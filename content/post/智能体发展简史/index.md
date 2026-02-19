---
title: 智能体发展简史
description: 本文主要介绍了基于大语言模型的智能体及其应用。
date: 2026-02-19T12:34:25+08:00
draft: false
categories:
- 编程
- AI
- LLM
- NLP
- Datawhale
tags:
- LLM-based Agent
---

# 智能体发展史

# 第一章 初识智能体

欢迎来到智能体的世界！在人工智能浪潮席卷全球的今天，<strong>智能体（Agent）</strong>已成为驱动技术变革与应用创新的核心概念之一。无论你的志向是成为 AI 领域的研究者、工程师，还是希望深刻理解技术前沿的观察者，掌握智能体的本质，都将是你知识体系中不可或缺的一环。

因此，在本章，让我们回到原点，一起探讨几个问题：智能体是什么？它有哪些主要的类型？它又是如何与我们所处的世界进行交互的？通过这些讨论，希望能为你未来的学习和探索打下坚实的基础。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-0.png" alt="图片描述" width="90%"/>
  <p>图 1.1 智能体与环境的基本交互循环</p>
</div>

## 1.1 什么是智能体？

在探索任何一个复杂概念时，我们最好从一个简洁的定义开始。在人工智能领域，智能体被定义为任何能够通过<strong>传感器（Sensors）</strong>感知其所处<strong>环境（Environment）</strong>，并<strong>自主</strong>地通过<strong>执行器（Actuators）</strong>采取<strong>行动（Action）</strong>以达成特定目标的实体。

这个定义包含了智能体存在的四个基本要素。环境是智能体所处的外部世界。对于自动驾驶汽车，环境是动态变化的道路交通；对于一个交易算法，环境则是瞬息万变的金融市场。智能体并非与环境隔离，它通过其传感器持续地感知环境状态。摄像头、麦克风、雷达或各类<strong>应用程序编程接口（Application Programming Interface, API）</strong>返回的数据流，都是其感知能力的延伸。

获取信息后，智能体需要采取行动来对环境施加影响，它通过执行器来改变环境的状态。执行器可以是物理设备（如机械臂、方向盘）或虚拟工具（如执行一段代码、调用一个服务）。

然而，真正赋予智能体"智能"的，是其<strong>自主性（Autonomy）</strong>。智能体并非只是被动响应外部刺激或严格执行预设指令的程序，它能够基于其感知和内部状态进行独立决策，以达成其设计目标。这种从感知到行动的闭环，构成了所有智能体行为的基础，如图 1.1 所示。


### 1.1.1 传统视角下的智能体

在当前<strong>大语言模型（Large Language Model, LLM）</strong>的热潮出现之前，人工智能的先驱们已经对“智能体”这一概念进行了数十年的探索与构建。这些如今我们称之为“传统智能体”的范式，并非单一的静态概念，而是经历了一条从简单到复杂、从被动反应到主动学习的清晰演进路线。

这个演进的起点，是那些结构最简单的<strong>反射智能体（Simple Reflex Agent）</strong>。它们的决策核心由工程师明确设计的“条件-动作”规则构成，如图 1.2 所示。经典的自动恒温器便是如此：若传感器感知的室温高于设定值，则启动制冷系统。

这种智能体完全依赖于当前的感知输入，不具备记忆或预测能力。它像一种数字化的本能，可靠且高效，但也因此无法应对需要理解上下文的复杂任务。它的局限性引出了一个关键问题：如果环境的当前状态不足以作为决策的全部依据，智能体该怎么办？

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-1.png" alt="图片描述" width="90%"/>
  <p>图 1.2 简单反射智能体的决策逻辑示意图</p>
</div>

为了回答这个问题，研究者们引入了“状态”的概念，发展出<strong>基于模型的反射智能体（Model-Based Reflex Agent）</strong>。这类智能体拥有一个内部的<strong>世界模型（World Model）</strong>，用于追踪和理解环境中那些无法被直接感知的方面。它试图回答：“世界现在是什么样子的？”。例如，一辆在隧道中行驶的自动驾驶汽车，即便摄像头暂时无法感知到前方的车辆，它的内部模型依然会维持对那辆车存在、速度和预估位置的判断。这个内部模型让智能体拥有了初级的“记忆”，使其决策不再仅仅依赖于瞬时感知，而是基于一个更连贯、更完整的世界状态理解。

然而，仅仅理解世界还不够，智能体需要有明确的目标。这促进了<strong>基于目标的智能体（Goal-Based Agent）</strong>的发展。与前两者不同，它的行为不再是被动地对环境做出反应，而是主动地、有预见性地选择能够导向某个特定未来状态的行动。这类智能体需要回答的问题是：“我应该做什么才能达成目标？”。经典的例子是 GPS 导航系统：你的目标是到达公司，智能体会基于地图数据（世界模型），通过搜索算法（如 A*算法）来规划（Planning）出一条最优路径。这类智能体的核心能力体现在了对未来的考量与规划上。

更进一步，现实世界的目标往往不是单一的。我们不仅希望到达公司，还希望时间最短、路程最省油并且避开拥堵。当多个目标需要权衡时，<strong>基于效用的智能体（Utility-Based Agent）</strong>便随之出现。它为每一个可能的世界状态都赋予一个效用值，这个值代表了满意度的高低。智能体的核心目标不再是简单地达成某个特定状态，而是最大化期望效用。它需要回答一个更复杂的问题：“哪种行为能为我带来最满意的结果？”。这种架构让智能体学会在相互冲突的目标之间进行权衡，使其决策更接近人类的理性选择。

至此，我们讨论的智能体虽然功能日益复杂，但其核心决策逻辑，无论是规则、模型还是效用函数，依然依赖于人类设计师的先验知识。如果智能体能不依赖预设，而是通过与环境的互动自主学习呢？

这便是<strong>学习型智能体（Learning Agent）</strong>的核心思想，而<strong>强化学习（Reinforcement Learning, RL）</strong>是实现这一思想最具代表性的路径。一个学习型智能体包含一个性能元件（即我们前面讨论的各类智能体）和一个学习元件。学习元件通过观察性能元件在环境中的行动所带来的结果来不断修正性能元件的决策策略。

想象一个学习下棋的 AI。它开始时可能只是随机落子，当它最终赢下一局时，系统会给予它一个正向的奖励。通过大量的自我对弈，学习元件会逐渐发现哪些棋路更有可能导向最终的胜利。AlphaGo Zero 是这一理念的一个里程碑式的成就。它在围棋这一复杂博弈中，通过强化学习发现了许多超越人类既有知识的有效策略。

从简单的恒温器，到拥有内部模型的汽车，再到能够规划路线的导航、懂得权衡利弊的决策者，最终到可以通过经验自我进化的学习者。这条演进之路，展示了传统人工智能在构建机器智能的道路上所经历的发展脉络。它们为我们今天理解更前沿的智能体范式，打下了坚实而必要的基础。

### 1.1.2 大语言模型驱动的新范式

以<strong>GPT（Generative Pre-trained Transformer）</strong>为代表的大语言模型的出现，正在显著改变智能体的构建方法与能力边界。由大语言模型驱动的 LLM 智能体，其核心决策机制与传统智能体存在本质区别，从而赋予了其一系列全新的特性。

这种转变，可以从两者在核心引擎、知识来源、交互方式等多个维度的对比中清晰地看出，如表 1.1 所示。简而言之，传统智能体的能力源于工程师的显式编程与知识构建，其行为模式是确定且有边界的；而 LLM 智能体则通过在海量数据上的预训练，获得了隐式的世界模型与强大的涌现能力，使其能够以更灵活、更通用的方式应对复杂任务。

<div align="center">
  <p>表 1.1 传统智能体与 LLM 驱动智能体的核心对比</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-2.png" alt="图片描述" width="90%"/>
</div>

这种差异使得 LLM 智能体可以直接处理高层级、模糊且充满上下文信息的自然语言指令。让我们以一个“智能旅行助手”为例来说明。

在 LLM 智能体出现之前，规划旅行通常意味着用户需要在多个专用应用（如天气、地图、预订网站）之间手动切换，并由用户自己扮演信息整合与决策的角色。而一个 LLM 智能体则能将这个流程整合起来。当接收到“规划一次厦门之旅”这样的模糊指令时，它的工作方式体现了以下几点：

- <strong>规划与推理</strong>：智能体首先会将这个高层级目标分解为一系列逻辑子任务，例如：`[确认出行偏好] -> [查询目的地信息] -> [制定行程草案] -> [预订票务住宿]`。这是一个内在的、由模型驱动的规划过程。
- <strong>工具使用</strong>：在执行规划时，智能体识别到信息缺口，会主动调用外部工具来补全。例如，它会调用天气查询接口获取实时天气，并基于“预报有雨”这一信息，在后续规划中倾向于推荐室内活动。
- <strong>动态修正</strong>：在交互过程中，智能体会将用户的反馈（如“这家酒店超出预算”）视为新的约束，并据此调整后续的行动，重新搜索并推荐符合新要求的选项。整个“<strong>查天气 → 调行程 → 订酒店</strong>”的流程，展现了其根据上下文动态修正自身行为的能力。

总而言之，我们正从开发专用自动化工具转向构建能自主解决问题的系统。核心不再是编写代码，而是引导一个通用的“大脑”去规划、行动和学习。

### 1.1.3 智能体的类型

继上文回顾智能体的演进后，本节将从三个互补的维度对智能体进行分类。

（1）<strong>基于内部决策架构的分类</strong>

第一种分类维度是依据智能体内部决策架构的复杂程度，这个视角在《Artificial Intelligence: A Modern Approach》中系统性地提出<sup>[1]</sup>。正如 1.1.1 节所述，传统智能体的演进路径本身就构成了最经典的分类阶梯，它涵盖了从简单的<strong>反应式</strong>智能体，到引入内部模型的<strong>模型式</strong>智能体，再到更具前瞻性的<strong>基于目标</strong>和<strong>基于效用</strong>的智能体。此外，<strong>学习能力</strong>则是一种可赋予上述所有类型的元能力，使其能通过经验自我改进。

（2）<strong>基于时间与反应性的分类</strong>

除了内部架构的复杂性，还可以从智能体处理决策的时间维度进行分类。这个视角关注智能体是在接收到信息后立即行动，还是会经过深思熟虑的规划再行动。这揭示了智能体设计中一个核心权衡：追求速度的<strong>反应性（Reactivity）</strong>与追求最优解的<strong>规划性（Deliberation）</strong>之间的平衡，如图 1.3 所示。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-3.png" alt="图片描述" width="90%"/>
  <p>图 1.3 智能体决策时间与质量关系图</p>
</div>

- <strong>反应式智能体 (Reactive Agents)</strong>

这类智能体对环境刺激做出近乎即时的响应，决策延迟极低。它们通常遵循从感知到行动的直接映射，不进行或只进行极少的未来规划。上文的<strong>简单反应式</strong>和<strong>基于模型</strong>的智能体都属于此类别。

其核心优势在于<strong>速度快、计算开销低</strong>，这在需要快速决策的动态环境中至关重要。例如，车辆的安全气囊系统必须在碰撞发生的毫秒内做出反应，任何延迟都可能导致严重后果；同样，高频交易机器人也必须依赖反应式决策来捕捉稍纵即逝的市场机会。然而，这种速度的代价是“短视”，由于缺乏长远规划，反应式智能体容易陷入局部最优，难以完成需要多步骤协调的复杂任务。

- <strong>规划式智能体(Deliberative Agents)</strong>

与反应式智能体相对，规划式（或称审议式）智能体在行动前会进行复杂的思考和规划。它们不会立即对感知做出反应，而是会先利用其内部的世界模型，系统地探索未来的各种可能性，评估不同行动序列的后果，以期找到一条能够达成目标的最佳路径 。<strong>基于目标</strong>和<strong>基于效用</strong>的智能体是典型的规划式智能体。

可以将其决策过程类比为一位棋手。他不会只看眼前的一步，而是会预想对手可能的应对，并规划出后续几步甚至十几步的棋路。这种深思熟虑的能力使其能够处理复杂的、需要长远眼光的任务，例如制定一份商业计划或规划一次长途旅行。它们的优势在于决策的战略性和远见。然而，这种优势的另一面是高昂的时间和计算成本。在瞬息万变的环境中，当规划式智能体还在深思熟虑时，采取行动的最佳时机可能早已过去。

- <strong>混合式智能体(Hybrid Agents)</strong>

现实世界的复杂任务，往往既需要即时反应，也需要长远规划。例如，我们之前提到的智能旅行助手，既要能根据用户的即时反馈（如“这家酒店太贵了”）调整推荐（反应性），又要能规划出为期数天的完整旅行方案（规划性）。因此，混合式智能体应运而生，它旨在结合两者的优点，实现反应与规划的平衡。

一种经典的混合架构是分层设计：底层是一个快速的反应模块，处理紧急情况和基本动作；高层则是一个审慎的规划模块，负责制定长远目标。而现代的 LLM 智能体，则展现了一种更灵活的混合模式。它们通常在一个“思考-行动-观察”的循环中运作，巧妙地将两种模式融为一体：

- <strong>规划(Reasoning)</strong> ：在“思考”阶段，LLM 分析当前状况，规划出下一步的合理行动。这是一个审议过程。
- <strong>反应(Acting & Observing)</strong> ：在“行动”和“观察”阶段，智能体与外部工具或环境交互，并立即获得反馈。这是一个反应过程。

通过这种方式，智能体将一个需要长远规划的宏大任务，分解为一系列“规划-反应”的微循环。这使其既能灵活应对环境的即时变化，又能通过连贯的步骤，最终完成复杂的长期目标。

<strong>（3）基于知识表示的分类</strong>

这是一个更根本的分类维度，它探究智能体用以决策的知识，究竟是以何种形式存于其“思想”之中。这个问题是人工智能领域一场持续半个多世纪的辩论核心，并塑造了两种截然不同的 AI 文化。

- <strong>符号主义 AI（Symbolic AI）</strong>

符号主义，常被称为传统人工智能，其核心信念是：智能源于对符号的逻辑操作。这里的符号是人类可读的实体（如词语、概念），操作则遵循严格的逻辑规则，如图 1.4 左侧所示。这好比一位一丝不苟的图书管理员，将世界知识整理为清晰的规则库和知识图谱。

其主要优势在于透明和可解释。由于推理步骤明确，其决策过程可以被完整追溯，这在金融、医疗等高风险领域至关重要。然而，其“阿喀琉斯之踵”在于脆弱性：它依赖于一个完备的规则体系，但在充满模糊和例外的现实世界中，任何未被覆盖的新情况都可能导致系统失灵，这就是所谓的“知识获取瓶颈”。

- <strong>亚符号主义 AI（Sub-symbolic AI）</strong>

亚符号主义，或称连接主义，则提供了一幅截然不同的图景。在这里，知识并非显式的规则，而是内隐地分布在一个由大量神经元组成的复杂网络中，是从海量数据中学习到的统计模式。神经网络和深度学习是其代表。

如图 1.4 中间所示，如果说符号主义 AI 是图书管理员，那么亚符号主义 AI 就像一个牙牙学语的孩童 。他不是通过学习“猫有四条腿、毛茸茸、会喵喵叫”这样的规则来认识猫的，而是在看过成千上万张猫的图片后，大脑中的神经网络能辨识出“猫”这个概念的视觉模式 。这种方法的强大之处在于其模式识别能力和对噪声数据的鲁棒性 。它能够轻松处理图像、声音等非结构化数据，这在符号主义 AI 看来是极其困难的任务。

然而，这种强大的直觉能力也伴随着不透明性。亚符号主义系统通常被视为一个<strong>黑箱（Black Box）</strong>。它能以惊人的准确率识别出图片中的猫，但你若问它“为什么你认为这是猫？”，它很可能无法给出一个合乎逻辑的解释。此外，它在纯粹的逻辑推理任务上表现不佳，有时会产生看似合理却事实错误的幻觉 。

- <strong>神经符号主义 AI（Neuro-Symbolic AI）</strong>

长久以来，符号主义和亚符号主义这两大阵营如同两条平行线，各自发展。为克服上述两种范式的局限，一种“大和解”的思想开始兴起，这就是神经符号主义 AI，也称神经符号混合主义。它的目标，是融合两大范式的优点，创造出一个既能像神经网络一样从数据中学习，又能像符号系统一样进行逻辑推理的混合智能体。它试图弥合感知与认知、直觉与理性之间的鸿沟。诺贝尔经济学奖得主丹尼尔·卡尼曼（Daniel Kahneman）在其著作《思考，快与慢》（Thinking, Fast and Slow）中提出的双系统理论，为我们理解神经符号主义提供了一个绝佳的类比<sup>[2]</sup>，如图 1.4 所示：

- <strong>系统 1</strong>是快速、凭直觉、并行的思维模式，类似于亚符号主义 AI 强大的模式识别能力。
- <strong>系统 2</strong>是缓慢、有条理、基于逻辑的审慎思维，恰如符号主义 AI 的推理过程。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-4.png" alt="图片描述" width="90%"/>
  <p>图 1.4 符号主义、亚符号主义与神经符号混合主义的知识表示范式</p>
</div>

人类的智能，正源于这两个系统的协同工作。同样，一个真正鲁棒的 AI，也需要兼具二者之长。大语言模型驱动的智能体是神经符号主义的一个极佳实践范例。其内核是一个巨大的神经网络，使其具备模式识别和语言生成能力。然而，当它工作时，它会生成一系列结构化的中间步骤，如思想、计划或 API 调用，这些都是明确的、可操作的符号。通过这种方式，它实现了感知与认知、直觉与理性的初步融合。



## 1.2 智能体的构成与运行原理

### 1.2.1 任务环境定义

要理解智能体的运作，我们必须先理解它所处的<strong>任务环境</strong>。在人工智能领域，通常使用<strong>PEAS 模型</strong>来精确描述一个任务环境，即分析其<strong>性能度量(Performance)、环境(Environment)、执行器(Actuators)和传感器(Sensors)</strong> 。以上文提到的智能旅行助手为例，下表 1.2 展示了如何运用 PEAS 模型对其任务环境进行规约。

<div align="center">
  <p>表 1.2 智能旅行助手的 PEAS 描述</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-6.png" alt="图片描述" width="90%"/>
</div>


在实践中，LLM 智能体所处的数字环境展现出若干复杂特性，这些特性直接影响着智能体的设计。

首先，环境通常是<strong>部分可观察的</strong>。例如，旅行助手在查询航班时，无法一次性获取所有航空公司的全部实时座位信息。它只能通过调用航班预订 API，看到该 API 返回的部分数据，这就要求智能体必须具备记忆（记住已查询过的航线）和探索（尝试不同的查询日期）的能力。

其次，行动的结果也并非总是确定的。根据结果的可预测性，环境可分为<strong>确定性</strong>和<strong>随机性</strong>。旅行助手的任务环境就是典型的随机性环境。当它搜索票价时，两次相邻的调用返回的机票价格和余票数量都可能不同，这就要求智能体必须具备处理不确定性、监控变化并及时决策的能力。

此外，环境中还可能存在其他行动者，从而形成<strong>多智能体(Multi-agent)</strong> 环境。对于旅行助手而言，其他用户的预订行为、其他自动化脚本，甚至航司的动态调价系统，都是环境中的其他“智能体”。它们的行动（例如，订走最后一张特价票）会直接改变旅行助手所处环境的状态，这对智能体的快速响应和策略选择提出了更高要求。

最后，几乎所有任务都发生在<strong>序贯</strong>且<strong>动态</strong>的环境中。“序贯”意味着当前动作会影响未来；而“动态”则意味着环境自身可能在智能体决策时发生变化。这就要求智能体的“感知-思考-行动-观察”循环必须能够快速、灵活地适应持续变化的世界。

### 1.2.2 智能体的运行机制

在定义了智能体所处的任务环境后，我们来探讨其核心的运行机制。智能体并非一次性完成任务，而是通过一个持续的循环与环境进行交互，这个核心机制被称为 <strong>智能体循环 (Agent Loop)</strong>。如图 1.5 所示，该循环描述了智能体与环境之间的动态交互过程，构成了其自主行为的基础。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-5.png" alt="图片描述" width="90%"/>
  <p>图 1.5 智能体与环境交互的基本循环</p>
</div>

这个循环主要包含以下几个相互关联的阶段：

1. <strong>感知 (Perception)</strong>：这是循环的起点。智能体通过其传感器（例如，API 的监听端口、用户输入接口）接收来自环境的输入信息。这些信息，即<strong>观察 (Observation)</strong>，既可以是用户的初始指令，也可以是上一步行动所导致的环境状态变化反馈。
2. <strong>思考 (Thought)</strong>：接收到观察信息后，智能体进入其核心决策阶段。对于 LLM 智能体而言，这通常是由大语言模型驱动的内部推理过程。如图所示，“思考”阶段可进一步细分为两个关键环节：
   - <strong>规划 (Planning)</strong>：智能体基于当前的观察和其内部记忆，更新对任务和环境的理解，并制定或调整一个行动计划。这可能涉及将复杂目标分解为一系列更具体的子任务。
   - <strong>工具选择 (Tool Selection)</strong>：根据当前计划，智能体从其可用的工具库中，选择最适合执行下一步骤的工具，并确定调用该工具所需的具体参数。
3. <strong>行动 (Action)</strong>：决策完成后，智能体通过其执行器（Actuators）执行具体的行动。这通常表现为调用一个选定的工具（如代码解释器、搜索引擎 API），从而对环境施加影响，意图改变环境的状态。

行动并非循环的终点。智能体的行动会引起<strong>环境 (Environment)</strong> 的<strong>状态变化 (State Change)</strong>，环境随即会产生一个新的<strong>观察 (Observation)</strong> 作为结果反馈。这个新的观察又会在下一轮循环中被智能体的感知系统捕获，形成一个持续的“感知-思考-行动-观察”的闭环。智能体正是通过不断重复这一循环，逐步推进任务，从初始状态向目标状态演进。

### 1.2.3 智能体的感知与行动

在工程实践中，为了让 LLM 能够有效驱动这个循环，我们需要一套明确的<strong>交互协议 (Interaction Protocol)</strong> 来规范其与环境之间的信息交换。

在许多现代智能体框架中，这一协议体现在对智能体每一次输出的结构化定义上。智能体的输出不再是单一的自然语言回复，而是一段遵循特定格式的文本，其中明确地展示了其内部的推理过程与最终决策。

这个结构通常包含两个核心部分：

- <strong>Thought (思考)</strong>：这是智能体内部决策的“快照”。它以自然语言形式阐述了智能体如何分析当前情境、回顾上一步的观察结果、进行自我反思与问题分解，并最终规划出下一步的具体行动。
- <strong>Action (行动)</strong>：这是智能体基于思考后，决定对环境施加的具体操作，通常以函数调用的形式表示。

例如，一个正在规划旅行的智能体可能会生成如下格式化的输出：

```Bash
Thought: 用户想知道北京的天气。我需要调用天气查询工具。
Action: get_weather("北京")
```

这里的`Action`字段构成了对外部世界的指令。一个外部的<strong>解析器 (Parser)</strong> 会捕捉到这个指令，并调用相应的`get_weather`函数。

行动执行后，环境会返回一个结果。例如，`get_weather`函数可能返回一个包含详细天气数据的 JSON 对象。然而，原始的机器可读数据（如 JSON）通常包含 LLM 无需关注的冗余信息，且格式不符合其自然语言处理的习惯。

因此，感知系统的一个重要职责就是扮演传感器的角色：将这个原始输出处理并封装成一段简洁、清晰的自然语言文本，即观察。

```Bash
Observation: 北京当前天气为晴，气温25摄氏度，微风。
```

这段`Observation`文本会被反馈给智能体，作为下一轮循环的主要输入信息，供其进行新一轮的`Thought`和`Action`。

综上所述，通过这个由 Thought、Action、Observation 构成的严谨循环，LLM 智能体得以将内部的语言推理能力，与外部环境的真实信息和工具操作能力有效地结合起来。

## 1.3 动手体验：5 分钟实现第一个智能体

在前面的小节，我们学习了智能体的任务环境、核心运行机制以及 `Thought-Action-Observation` 交互范式。理论知识固然重要，但最好的学习方式是亲手实践。在本节中，我们将引导您使用几行简单的 Python 代码，从零开始构建一个可以工作的智能旅行助手。这个过程将遵循我们刚刚学到的理论循环，让您直观地感受到一个智能体是如何“思考”并与外部“工具”互动的。让我们开始吧！

在本案例中，我们的目标是构建一个能处理分步任务的智能旅行助手。需要解决的用户任务定义为："你好，请帮我查询一下今天北京的天气，然后根据天气推荐一个合适的旅游景点。"要完成这个任务，智能体必须展现出清晰的逻辑规划能力。它需要先调用天气查询工具，并将获得的观察结果作为下一步的依据。在下一轮循环中，它再调用景点推荐工具，从而得出最终建议。

### 1.3.1 准备工作

为了能从 Python 程序中访问网络 API，我们需要一个 HTTP 库。`requests`是 Python 社区中最流行、最易用的选择。`tavily-python`是一个强大的 AI 搜索 API 客户端，用于获取实时的网络搜索结果，可以在[官网](https://www.tavily.com/)注册后获取 API。`openai`是 OpenAI 官方提供的 Python SDK，用于调用 GPT 等大语言模型服务。请先通过以下命令安装它们：：

```bash
pip install requests tavily-python openai
```

（1）指令模板

驱动真实 LLM 的关键在于<strong>提示工程（Prompt Engineering）</strong>。我们需要设计一个“指令模板”，告诉 LLM 它应该扮演什么角色、拥有哪些工具、以及如何格式化它的思考和行动。这是我们智能体的“说明书”，它将作为`system_prompt`传递给 LLM。

```
AGENT_SYSTEM_PROMPT = """
你是一个智能旅行助手。你的任务是分析用户的请求，并使用可用工具一步步地解决问题。

# 可用工具:
- `get_weather(city: str)`: 查询指定城市的实时天气。
- `get_attraction(city: str, weather: str)`: 根据城市和天气搜索推荐的旅游景点。

# 行动格式:
你的回答必须严格遵循以下格式。首先是你的思考过程，然后是你要执行的具体行动，每次回复只输出一对Thought-Action：
Thought: [这里是你的思考过程和下一步计划]
Action: [这里是你要调用的工具，格式为 function_name(arg_name="arg_value")]

# 任务完成:
当你收集到足够的信息，能够回答用户的最终问题时，你必须在`Action:`字段后使用 `finish(answer="...")` 来输出最终答案。

请开始吧！
"""
```

（2）工具 1：查询真实天气

我们将使用免费的天气查询服务 `wttr.in`，它能以 JSON 格式返回指定城市的天气数据。下面是实现该工具的代码：

```python
import requests
import json

def get_weather(city: str) -> str:
    """
    通过调用 wttr.in API 查询真实的天气信息。
    """
    # API端点，我们请求JSON格式的数据
    url = f"https://wttr.in/{city}?format=j1"
    
    try:
        # 发起网络请求
        response = requests.get(url)
        # 检查响应状态码是否为200 (成功)
        response.raise_for_status() 
        # 解析返回的JSON数据
        data = response.json()
        
        # 提取当前天气状况
        current_condition = data['current_condition'][0]
        weather_desc = current_condition['weatherDesc'][0]['value']
        temp_c = current_condition['temp_C']
        
        # 格式化成自然语言返回
        return f"{city}当前天气:{weather_desc}，气温{temp_c}摄氏度"
        
    except requests.exceptions.RequestException as e:
        # 处理网络错误
        return f"错误:查询天气时遇到网络问题 - {e}"
    except (KeyError, IndexError) as e:
        # 处理数据解析错误
        return f"错误:解析天气数据失败，可能是城市名称无效 - {e}"
```

（3）工具 2：搜索并推荐旅游景点

我们将定义一个新工具 `search_attraction`，它会根据城市和天气状况，互联网上搜索合适的景点：

```python
import os
from tavily import TavilyClient

def get_attraction(city: str, weather: str) -> str:
    """
    根据城市和天气，使用Tavily Search API搜索并返回优化后的景点推荐。
    """
    # 1. 从环境变量中读取API密钥
    api_key = os.environ.get("TAVILY_API_KEY")
    if not api_key:
        return "错误:未配置TAVILY_API_KEY环境变量。"

    # 2. 初始化Tavily客户端
    tavily = TavilyClient(api_key=api_key)
    
    # 3. 构造一个精确的查询
    query = f"'{city}' 在'{weather}'天气下最值得去的旅游景点推荐及理由"
    
    try:
        # 4. 调用API，include_answer=True会返回一个综合性的回答
        response = tavily.search(query=query, search_depth="basic", include_answer=True)
        
        # 5. Tavily返回的结果已经非常干净，可以直接使用
        # response['answer'] 是一个基于所有搜索结果的总结性回答
        if response.get("answer"):
            return response["answer"]
        
        # 如果没有综合性回答，则格式化原始结果
        formatted_results = []
        for result in response.get("results", []):
            formatted_results.append(f"- {result['title']}: {result['content']}")
        
        if not formatted_results:
             return "抱歉，没有找到相关的旅游景点推荐。"

        return "根据搜索，为您找到以下信息:\n" + "\n".join(formatted_results)

    except Exception as e:
        return f"错误:执行Tavily搜索时出现问题 - {e}"
```

最后，我们将所有工具函数放入一个字典，供主循环调用：

```python
# 将所有工具函数放入一个字典，方便后续调用
available_tools = {
    "get_weather": get_weather,
    "get_attraction": get_attraction,
}
```



### 1.3.2 接入大语言模型

当前，许多 LLM 服务提供商（包括 OpenAI、Azure、以及众多开源模型服务框架如 Ollama、vLLM 等）都遵循了与 OpenAI API 相似的接口规范。这种标准化为开发者带来了极大的便利。智能体的自主决策能力来源于 LLM。我们将实现一个通用的客户端 `OpenAICompatibleClient`，它可以连接到任何兼容 OpenAI 接口规范的 LLM 服务。

```python
from openai import OpenAI

class OpenAICompatibleClient:
    """
    一个用于调用任何兼容OpenAI接口的LLM服务的客户端。
    """
    def __init__(self, model: str, api_key: str, base_url: str):
        self.model = model
        self.client = OpenAI(api_key=api_key, base_url=base_url)

    def generate(self, prompt: str, system_prompt: str) -> str:
        """调用LLM API来生成回应。"""
        print("正在调用大语言模型...")
        try:
            messages = [
                {'role': 'system', 'content': system_prompt},
                {'role': 'user', 'content': prompt}
            ]
            response = self.client.chat.completions.create(
                model=self.model,
                messages=messages,
                stream=False
            )
            answer = response.choices[0].message.content
            print("大语言模型响应成功。")
            return answer
        except Exception as e:
            print(f"调用LLM API时发生错误: {e}")
            return "错误:调用语言模型服务时出错。"
```

要实例化此类，您需要提供三个信息：`API_KEY`、`BASE_URL` 和 `MODEL_ID`，具体值取决于您使用的服务商（如 OpenAI 官方、Azure、或 Ollama 等本地模型），如果暂时没有渠道获取，可以参考 Datawhale 另一本教程的[1.2 API 设置](https://datawhalechina.github.io/handy-multi-agent/#/chapter1/1.2.api-setup)。

### 1.3.3 执行行动循环

下面的主循环将整合所有组件，并通过格式化后的 Prompt 驱动 LLM 进行决策。

```python
import re

# --- 1. 配置LLM客户端 ---
# 请根据您使用的服务，将这里替换成对应的凭证和地址
API_KEY = "YOUR_API_KEY"
BASE_URL = "YOUR_BASE_URL"
MODEL_ID = "YOUR_MODEL_ID"
TAVILY_API_KEY="YOUR_Tavily_KEY"
os.environ['TAVILY_API_KEY'] = "YOUR_TAVILY_API_KEY"

llm = OpenAICompatibleClient(
    model=MODEL_ID,
    api_key=API_KEY,
    base_url=BASE_URL
)

# --- 2. 初始化 ---
user_prompt = "你好，请帮我查询一下今天北京的天气，然后根据天气推荐一个合适的旅游景点。"
prompt_history = [f"用户请求: {user_prompt}"]

print(f"用户输入: {user_prompt}\n" + "="*40)

# --- 3. 运行主循环 ---
for i in range(5): # 设置最大循环次数
    print(f"--- 循环 {i+1} ---\n")
    
    # 3.1. 构建Prompt
    full_prompt = "\n".join(prompt_history)
    
    # 3.2. 调用LLM进行思考
    llm_output = llm.generate(full_prompt, system_prompt=AGENT_SYSTEM_PROMPT)
    # 模型可能会输出多余的Thought-Action，需要截断
    match = re.search(r'(Thought:.*?Action:.*?)(?=\n\s*(?:Thought:|Action:|Observation:)|\Z)', llm_output, re.DOTALL)
    if match:
        truncated = match.group(1).strip()
        if truncated != llm_output.strip():
            llm_output = truncated
            print("已截断多余的 Thought-Action 对")
    print(f"模型输出:\n{llm_output}\n")
    prompt_history.append(llm_output)
    
    # 3.3. 解析并执行行动
    action_match = re.search(r"Action: (.*)", llm_output, re.DOTALL)
    if not action_match:
        print("解析错误:模型输出中未找到 Action。")
        break
    action_str = action_match.group(1).strip()

    if action_str.startswith("finish"):
        final_answer = re.search(r'finish\(answer="(.*)"\)', action_str).group(1)
        print(f"任务完成，最终答案: {final_answer}")
        break
    
    tool_name = re.search(r"(\w+)\(", action_str).group(1)
    args_str = re.search(r"\((.*)\)", action_str).group(1)
    kwargs = dict(re.findall(r'(\w+)="([^"]*)"', args_str))

    if tool_name in available_tools:
        observation = available_tools[tool_name](**kwargs)
    else:
        observation = f"错误:未定义的工具 '{tool_name}'"

    # 3.4. 记录观察结果
    observation_str = f"Observation: {observation}"
    print(f"{observation_str}\n" + "="*40)
    prompt_history.append(observation_str)
```

通过以上步骤，我们构建了一个完整的、由真实 LLM 驱动的智能体。其核心在于“工具”和“提示工程”的结合，这正是当前主流智能体框架（如 LangChain、LlamaIndex 等）的设计精髓。

### 1.3.4 运行案例分析

以下输出完整地展示了一个成功的智能体执行流程。通过对这个三轮循环的分析，我们可以清晰地看到智能体解决问题的核心能力。

```bash
用户输入: 你好，请帮我查询一下今天北京的天气，然后根据天气推荐一个合适的旅游景点。
========================================
--- 循环 1 ---

正在调用大语言模型...
大语言模型响应成功。
模型输出:
Thought: 首先需要获取北京今天的天气情况，之后再根据天气情况来推荐旅游景点。
Action: get_weather(city="北京")

Observation: 北京当前天气:Sunny，气温26摄氏度
========================================      
--- 循环 2 ---

正在调用大语言模型...
大语言模型响应成功。
模型输出:
Thought: 现在已经知道了北京今天的天气是晴朗且温度适中，接下来可以基于这个信息来推荐一个适合的旅游景点了。
Action: get_attraction(city="北京", weather="Sunny")

Observation: 北京在晴天最值得去的旅游景点是颐和园，因其美丽的湖景和古建筑。另一个推荐是长城，因其壮观的景观和历史意义。
========================================
--- 循环 3 ---

正在调用大语言模型...
大语言模型响应成功。
模型输出:
Thought: 已经获得了两个适合晴天游览的景点建议，现在可以根据这些信息给用户提供满意的答复。
Action: finish(answer="今天北京的天气是晴朗的，气温26摄氏度，非常适合外出游玩。我推荐您去颐和园欣赏美丽的湖景和古建筑，或者前往长城体验其壮观的景观和深厚的历史意义。希望您有一个愉快的旅行！
")

任务完成，最终答案: 今天北京的天气是晴朗的，气温26摄氏度，非常适合外出游玩。我推荐您去颐和园欣赏美丽的湖景和古建筑，或者前往长城体验其壮观的景观和深厚的历史意义。希望您有一个愉快的旅行！
```

这个简单的旅行助手案例，集中演示了基于`Thought-Action-Observation`范式的智能体所具备的四项基本能力：任务分解、工具调用、上下文理解和结果合成。正是通过这个循环的不断迭代，智能体才得以将一个模糊的用户意图，转化为一系列具体、可执行的步骤，并最终达成目标。

## 1.4 智能体应用的协作模式

上一节，我们通过亲手构建一个智能体，深入理解了其内部的运作循环。不过在更广泛的应用场景中，我们的角色正越来越多地转变为使用者与协作者。基于智能体在任务中的角色和自主性程度，其协作模式主要分为两种：一种是作为高效工具，深度融入我们的工作流；另一种则是作为自主的协作者，与其他智能体协作完成复杂目标。

### 1.4.1 作为开发者工具的智能体

在这种模式下，智能体被深度集成到开发者的工作流中，作为一种强大的辅助工具。它增强而非取代开发者的角色，通过自动化处理繁琐、重复的任务，让开发者能更专注于创造性的核心工作。这种人机协同的方式，极大地提升了软件开发的效率与质量。

目前，市场上涌现了多款优秀的 AI 编程辅助工具，它们虽然均能提升开发效率，但在实现路径和功能侧重上各有千秋：

- <strong>GitHubCopilot</strong>: 作为该领域最具影响力的产品之一，Copilot 由 GitHub 与 OpenAI 联合开发。它深度集成于 Visual Studio Code 等主流编辑器中，以其强大的代码自动补全能力而闻名。开发者在编写代码时，Copilot 能实时提供整行甚至整个函数块的建议。近年来，它也通过 Copilot Chat 扩展了对话式编程的能力，允许开发者在编辑器内通过聊天解决编程问题。
- <strong>Claude Code</strong>: Claude Code 是由 Anthropic 开发的 AI 编程助手，旨在通过自然语言指令帮助开发者在终端中高效地完成编码任务。它能够理解完整的代码库结构，执行代码编辑、测试和调试等操作，支持从描述功能到代码实现的全流程开发。Claude Code 还提供了无交互（headless）模式，适用于 CI、pre-commit hooks、构建脚本和其他自动化场景，为开发者提供了强大的命令行编程体验。
- <strong>Trae</strong>: 作为新兴的 AI 编程工具，Trae 专注于为开发者提供智能化的代码生成和优化服务。它通过深度学习技术分析代码模式，能够为开发者提供精准的代码建议和自动化重构方案。Trae 的特色在于其轻量级的设计和快速响应能力，特别适合需要频繁迭代和快速原型开发的场景。
- <strong>Cursor</strong>: 与上述主要作为插件或集成功能存在的工具不同，Cursor 则选择了一条更具整合性的路径，它本身就是一个 AI 原生的代码编辑器。它并非在现有编辑器上增加 AI 功能，而是在设计之初就将 AI 交互作为核心。除了具备顶级的代码生成和聊天能力外，它更强调让 AI 理解整个代码库的上下文，从而实现更深层次的问答、重构和调试。

当然还有许多优秀的工具没有例举，不过它们共同指向了一个明确的趋势：AI 正在深度融入软件开发的全生命周期，通过构建高效的人机协同工作流，深刻地重塑着软件工程的效率边界与开发范式。

### 1.4.2 作为自主协作者的智能体

与作为工具辅助人类不同，第二种交互模式将智能体的自动化程度提升到了一个全新的层次，自主协作者。在这种模式下，我们不再是手把手地指导 AI 完成每一步，而是将一个高层级的目标委托给它。智能体会像一个真正的项目成员一样，独立地进行规划、推理、执行和反思，直到最终交付成果。这种从助手到协作者的转变，使得 LLM 智能体更深的进入了大众的视野。它标志着我们与 AI 的关系从“命令-执行”演变为“目标-委托”。智能体不再是被动的工具，而是主动的目标追求者。

当前，实现这种自主协作的思路百花齐放，涌现了大量优秀的框架和产品，从早期的 BabyAGI、AutoGPT，到如今更为成熟的 CrewAI、AutoGen、MetaGPT、LangGraph 等优秀框架，共同推动着这一领域的高速发展。虽然具体实现千差万别，但它们的架构范式大致可以归纳为几个主流方向：

1. <strong>单智能体自主循环</strong>：这是早期的典型范式，如 <strong>AgentGPT</strong> 所代表的模式。其核心是一个通用智能体通过“思考-规划-执行-反思”的闭环，不断进行自我提示和迭代，以完成一个开放式的高层级目标。
2. <strong>多智能体协作</strong>：这是当前最主流的探索方向，旨在通过模拟人类团队的协作模式来解决复杂问题。它又可细分为不同模式： <strong>角色扮演式对话</strong>：如 <strong>CAMEL</strong> 框架，通过为两个智能体（例如，“程序员”和“产品经理”）设定明确的角色和沟通协议，让它们在一个结构化的对话中协同完成任务。 <strong>组织化工作流</strong>：如 <strong>MetaGPT</strong> 和 <strong>CrewAI</strong>，它们模拟一个分工明确的“虚拟团队”（如软件公司或咨询小组）。每个智能体都有预设的职责和工作流程（SOP），通过层级化或顺序化的方式协作，产出高质量的复杂成果（如完整的代码库或研究报告）。<strong>AutoGen</strong> 和 <strong>AgentScope</strong> 则提供了更灵活的对话模式，允许开发者自定义智能体间的复杂交互网络。
3. <strong>高级控制流架构</strong>：诸如 <strong>LangGraph</strong> 等框架，则更侧重于为智能体提供更强大的底层工程基础。它将智能体的执行过程建模为状态图（State Graph），从而能更灵活、更可靠地实现循环、分支、回溯以及人工介入等复杂流程。

这些不同的架构范式，共同推动着自主智能体从理论构想走向更广泛的实际应用，使其有能力应对日益复杂的真实世界任务。在我们的后续章节中，也会感受不同类型框架之间的差异和优势。

### 1.4.3 Workflow 和 Agent 的差异

在理解了智能体作为“工具”和“协作者”两种模式后，我们有必要对 Workflow 和 Agent 的差异展开讨论，尽管它们都旨在实现任务自动化，但其底层逻辑、核心特征和适用场景却截然不同。

简单来说，<strong>Workflow 是让 AI 按部就班地执行指令，而 Agent 则是赋予 AI 自由度去自主达成目标。</strong>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-18.png" alt="图片描述" width="90%"/>
  <p>图 1.6 Workflow 和 Agent 的差异</p>
</div>

如图 1.6 所示，工作流是一种传统的自动化范式，其核心是<strong>对一系列任务或步骤进行预先定义的、结构化的编排</strong>。它本质上是一个精确的、静态的流程图，规定了在何种条件下、以何种顺序执行哪些操作。一个典型的案例：某企业的费用报销审批流程。员工提交报销单（触发）-> 如果金额小于 500 元，直接由部门经理审批 -> 如果金额大于 500 元，先由部门经理审批，再流转至财务总监审批 -> 审批通过后，通知财务部打款。整个过程的每一步、每一个判断条件都被精确地预先设定。

与工作流不同，基于大型语言模型的智能体是一个<strong>具备自主性的、以目标为导向的系统</strong>。它不仅仅是执行预设指令，而是能够在一定程度上理解环境、进行推理、制定计划，并动态地采取行动以达成最终目标。LLM 在其中扮演着“大脑”的角色。一个典型的例子，便是我们在 1.3 节中写的智能旅行助手。当我们向它下达一个新指令，例如：<strong>“你好，请帮我查询一下今天北京的天气，然后根据天气推荐一个合适的旅游景点。”</strong> 它的处理过程充分展现了其自主性：

1. <strong>规划与工具调用：</strong> Agent 首先会把任务拆解为两个步骤：① 查询天气；② 基于天气推荐景点。随即，它会自主选择并调用“天气查询 API”，并将“北京”作为参数传入。
2. <strong>推理与决策：</strong> 假设 API 返回结果为“晴朗，微风”。Agent 的 LLM 大脑会基于这个信息进行推理：“晴天适合户外活动”。接着，它会根据这个判断，在它的知识库或通过搜索引擎这个工具中，筛选出北京的户外景点，如故宫、颐和园、天坛公园等。
3. <strong>生成结果：</strong> 最后，Agent 会综合信息，给出一个完整的、人性化的回答：“今天北京天气晴朗，微风，非常适合户外游玩。为您推荐前往【颐和园】，您可以在昆明湖上泛舟，欣赏美丽的皇家园林景色。”

在这个过程中，没有任何写死的`if天气=晴天 then 推荐颐和园`的规则。如果天气是“雨天”，Agent 会自主推理并推荐国家博物馆、首都博物馆等室内场所。<strong>这种基于实时信息进行动态推理和决策的能力，正是 Agent 的核心价值所在。</strong>



## 1.4 本章小结

在本章中，我们共同踏上了探索智能体的初识之旅。我们的旅程从最基本的问题开始：

- <strong>什么是大语言模型驱动的智能体？</strong> 我们首先明确了其定义，理解了现代智能体是具备了能力的实体。它不再仅仅是执行预设程序的脚本，而是能够自主推理和使用工具的决策者。
- <strong>智能体如何工作？</strong> 我们深入探讨了智能体与环境交互的运行机制。我们了解到，这个持续的闭环是智能体处理信息、做出决策、影响环境并根据反馈调整自身行为的基础。
- <strong>如何构建智能体？</strong> 这是本章的实践核心。我们以“智能旅行助手”为例，亲手构建了一个完整的、由真实 LLM 驱动的智能体。
- <strong>智能体有哪些主流的应用范式？</strong> 最后，我们将视野投向了更广阔的应用领域。我们探讨了两种主流的智能体交互模式：一是以 GitHub Copilot 和 Cursor 等为代表的、增强人类工作流的“开发者工具”；二是以 CrewAI、MetaGPT 和 AgentScope 等框架为代表的、能够独立完成高层级目标的“自主协作者”。同时讲解了 Workflow 与 Agent 的差异。

通过本章的学习，我们建立了一个关于智能体的基础认知框架。那么，它是如何一步步从最初的构想演进至今的呢？在下一章中，我们将探索智能体的发展历史，一段追本溯源的旅程即将开始！

## 本章完整代码

```python
AGENT_SYSTEM_PROMPT = """
你是一个智能旅行助手。你的任务是分析用户的请求，并使用可用工具一步步地解决问题。

# 可用工具:
- `get_weather(city: str)`: 查询指定城市的实时天气。
- `get_attraction(city: str, weather: str)`: 根据城市和天气搜索推荐的旅游景点。

# 行动格式:
你的回答必须严格遵循以下格式。首先是你的思考过程，然后是你要执行的具体行动，每次回复只输出一对Thought-Action：
Thought: [这里是你的思考过程和下一步计划]
Action: [这里是你要调用的工具，格式为 function_name(arg_name="arg_value")]

# 任务完成:
当你收集到足够的信息，能够回答用户的最终问题时，你必须在`Action:`字段后使用 `finish(answer="...")` 来输出最终答案。

请开始吧！
"""


import requests
import json

def get_weather(city: str) -> str:
    """
    通过调用 wttr.in API 查询真实的天气信息。
    """
    # API端点，我们请求JSON格式的数据
    url = f"https://wttr.in/{city}?format=j1"
    
    try:
        # 发起网络请求
        response = requests.get(url)
        # 检查响应状态码是否为200 (成功)
        response.raise_for_status() 
        # 解析返回的JSON数据
        data = response.json()
        
        # 提取当前天气状况
        current_condition = data['current_condition'][0]
        weather_desc = current_condition['weatherDesc'][0]['value']
        temp_c = current_condition['temp_C']
        
        # 格式化成自然语言返回
        return f"{city}当前天气：{weather_desc}，气温{temp_c}摄氏度"
        
    except requests.exceptions.RequestException as e:
        # 处理网络错误
        return f"错误：查询天气时遇到网络问题 - {e}"
    except (KeyError, IndexError) as e:
        # 处理数据解析错误
        return f"错误：解析天气数据失败，可能是城市名称无效 - {e}"



import os
from tavily import TavilyClient

def get_attraction(city: str, weather: str) -> str:
    """
    根据城市和天气，使用Tavily Search API搜索并返回优化后的景点推荐。
    """

    # 从环境变量或主程序配置中获取API密钥
    api_key = os.environ.get("TAVILY_API_KEY") # 推荐方式
    # 或者，我们可以在主循环中传入，如此处代码所示

    if not api_key:
        return "错误：未配置TAVILY_API_KEY。"

    # 2. 初始化Tavily客户端
    tavily = TavilyClient(api_key=api_key)
    
    # 3. 构造一个精确的查询
    query = f"'{city}' 在'{weather}'天气下最值得去的旅游景点推荐及理由"
    
    try:
        # 4. 调用API，include_answer=True会返回一个综合性的回答
        response = tavily.search(query=query, search_depth="basic", include_answer=True)
        
        # 5. Tavily返回的结果已经非常干净，可以直接使用
        # response['answer'] 是一个基于所有搜索结果的总结性回答
        if response.get("answer"):
            return response["answer"]
        
        # 如果没有综合性回答，则格式化原始结果
        formatted_results = []
        for result in response.get("results", []):
            formatted_results.append(f"- {result['title']}: {result['content']}")
        
        if not formatted_results:
             return "抱歉，没有找到相关的旅游景点推荐。"

        return "根据搜索，为您找到以下信息：\n" + "\n".join(formatted_results)

    except Exception as e:
        return f"错误：执行Tavily搜索时出现问题 - {e}"


# 将所有工具函数放入一个字典，方便后续调用
available_tools = {
    "get_weather": get_weather,
    "get_attraction": get_attraction,
}

from openai import OpenAI

class OpenAICompatibleClient:
    """
    一个用于调用任何兼容OpenAI接口的LLM服务的客户端。
    """
    def __init__(self, model: str, api_key: str, base_url: str):
        self.model = model
        self.client = OpenAI(api_key=api_key, base_url=base_url)

    def generate(self, prompt: str, system_prompt: str) -> str:
        """调用LLM API来生成回应。"""
        print("正在调用大语言模型...")
        try:
            messages = [
                {'role': 'system', 'content': system_prompt},
                {'role': 'user', 'content': prompt}
            ]
            response = self.client.chat.completions.create(
                model=self.model,
                messages=messages,
                stream=False
            )
            answer = response.choices[0].message.content
            print("大语言模型响应成功。")
            return answer
        except Exception as e:
            print(f"调用LLM API时发生错误: {e}")
            return "错误：调用语言模型服务时出错。"

import re

# --- 1. 配置LLM客户端 ---
# 请根据您使用的服务，将这里替换成对应的凭证和地址
API_KEY = "YOUR_API_KEY"
BASE_URL = "YOUR_BASE_URL"
MODEL_ID = "YOUR_MODEL_ID"
os.environ['TAVILY_API_KEY'] = "YOUR_TAVILY_API_KEY"

llm = OpenAICompatibleClient(
    model=MODEL_ID,
    api_key=API_KEY,
    base_url=BASE_URL
)

# --- 2. 初始化 ---
user_prompt = "你好，请帮我查询一下今天北京的天气，然后根据天气推荐一个合适的旅游景点。"
prompt_history = [f"用户请求: {user_prompt}"]

print(f"用户输入: {user_prompt}\n" + "="*40)

# --- 3. 运行主循环 ---
for i in range(5): # 设置最大循环次数
    print(f"--- 循环 {i+1} ---\n")
    
    # 3.1. 构建Prompt
    full_prompt = "\n".join(prompt_history)
    
    # 3.2. 调用LLM进行思考
    llm_output = llm.generate(full_prompt, system_prompt=AGENT_SYSTEM_PROMPT)
    # 模型可能会输出多余的Thought-Action，需要截断
    match = re.search(r'(Thought:.*?Action:.*?)(?=\n\s*(?:Thought:|Action:|Observation:)|\Z)', llm_output, re.DOTALL)
    if match:
        truncated = match.group(1).strip()
        if truncated != llm_output.strip():
            llm_output = truncated
            print("已截断多余的 Thought-Action 对")
    print(f"模型输出:\n{llm_output}\n")
    prompt_history.append(llm_output)
    
    # 3.3. 解析并执行行动
    action_match = re.search(r"Action: (.*)", llm_output, re.DOTALL)
    if not action_match:
        print("解析错误：模型输出中未找到 Action。")
        break
    action_str = action_match.group(1).strip()

    if action_str.startswith("finish"):
        final_answer = re.search(r'finish\(answer="(.*)"\)', action_str).group(1)
        print(f"任务完成，最终答案: {final_answer}")
        break
    
    tool_name = re.search(r"(\w+)\(", action_str).group(1)
    args_str = re.search(r"\((.*)\)", action_str).group(1)
    kwargs = dict(re.findall(r'(\w+)="([^"]*)"', args_str))

    if tool_name in available_tools:
        observation = available_tools[tool_name](**kwargs)
    else:
        observation = f"错误：未定义的工具 '{tool_name}'"

    # 3.4. 记录观察结果
    observation_str = f"Observation: {observation}"
    print(f"{observation_str}\n" + "="*40)
    prompt_history.append(observation_str)
```



## 习题

> <strong>提示</strong>：以下的部分习题没有标准答案，重点在于培养学习者对智能体系统批判性的深入思考和动手实践能力。

1. 请分析以下四个 `case` 中的<strong>主体</strong>是否属于智能体，如果是，那么属于哪种类型的智能体（可以从多个分类维度进行分析），并说明理由：

   `case A`：<strong>一台符合冯·诺依曼结构的超级计算机</strong>，拥有高达每秒 2EFlop 的峰值算力

   `case B`：<strong>特斯拉自动驾驶系统</strong>在高速公路上行驶时，突然检测到前方有障碍物，需要在毫秒级做出刹车或变道决策

   `case C`：<strong>AlphaGo</strong>在与人类棋手对弈时，需要评估当前局面并规划未来数十步的最优策略

   `case D`：<strong>ChatGPT 扮演的智能客服</strong>在处理用户投诉时，需要查询订单信息、分析问题原因、提供解决方案并安抚用户情绪

2. 假设你需要为一个"智能健身教练"设计任务环境。这个智能体能够：
   - 通过可穿戴设备监测用户的心率、运动强度等生理数据
   - 根据用户的健身目标（减脂/增肌/提升耐力）动态调整训练计划
   - 在用户运动过程中提供实时语音指导和动作纠正
   - 评估训练效果并给出饮食建议

   请使用 PEAS 模型完整描述这个智能体的任务环境，并分析该环境具有哪些特性（如部分可观察、随机性、动态性等）。

3. 某电商公司正在考虑两种方案来处理售后退款申请：
   
   方案 A（`Workflow`）：设计一套固定流程，例如：

   A.1 对于一般商品且在 7 天之内，金额 `< 100RMB` 自动通过；`100-500RMB `由客服审核；`>500RMB` 需主管审批；而特殊商品（如定制品）一律拒绝退款
   
   A.2 对于超过 7 天的商品，无论金额，只能由客服审核或主管审批；
   
   方案 B（`Agent`）：搭建一个智能体系统，让它理解退款政策、分析用户历史行为、评估商品状况，并自主决策是否批准退款
   
   请分析：
   - 这两种方案各自的优缺点是什么？
   - 在什么情况下 `Workflow` 更合适？什么情况下 `Agent` 更有优势？如果你是该电商公司的负责人，你更倾向于采用哪种方案？
   - 是否存在一个方案 C，能够结合两种方案，达到扬长避短的效果？
   
4. 在 1.3 节的智能旅行助手基础上，请思考如何添加以下功能（可以只描述设计思路，也可以进一步尝试代码实现）：

   > <strong>提示</strong>：思考如何修改 `Thought-Action-Observation` 循环来实现这些功能。

   - 添加一个"记忆"功能，让智能体记住用户的偏好（如喜欢历史文化景点、预算范围等）
   - 当推荐的景点门票已售罄时，智能体能够自动推荐备选方案
   - 如果用户连续拒绝了 3 个推荐，智能体能够反思并调整推荐策略

5. 卡尼曼的"系统 1"（快速直觉）和"系统 2"（慢速推理）理论<sup>[2]</sup>为神经符号主义 AI 提供了很好的类比。请首先构思一个具体的智能体的落地应用场景，然后说明场景中的：

   > <strong>提示</strong>：医疗诊断助手、法律咨询机器人、金融风控系统等都是常见的应用场景

   - 哪些任务应该由"系统 1"处理？
   - 哪些任务应该由"系统 2"处理？
   - 这两个系统如何协同工作以达成最终目标？

6. 尽管大语言模型驱动的智能体系统展现出了强大的能力，但它们仍然存在诸多局限。请分析以下问题：
   - 为什么智能体或智能体系统有时会产生"幻觉"（生成看似合理但实际错误的信息）？
   - 在 1.3 节的案例中，我们设置了最大循环次数为 5 次。如果没有这个限制，智能体可能会陷入什么问题？
   - 如何评估一个智能体的"智能"程度？仅使用准确率指标是否足够？


## 参考文献

[1] RUSSELL S, NORVIG P. Artificial Intelligence: A Modern Approach[M]. 4th ed. London: Pearson, 2020.

[2] KAHNEMAN D. Thinking, Fast and Slow[M]. New York: Farrar, Straus and Giroux, 2011.

# 第二章 智能体发展史

为了深刻理解现代智能体为何呈现出如今的形态，以及其核心设计思想的由来，本章将回溯历史：从人工智能领域的古典时代出发，探寻最早的“智能”如何在逻辑与符号的规则体系中被定义；继而见证从单一、集中的智能模型到分布式、协作式智能思想的重大转折；最终理解“学习”范式如何彻底改变了智能体获取能力的方式，并催生出我们今天所见的现代智能体。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-00.png" alt="图片描述" width="90%"/>
  <p>图 2.1 AI智能体的演进阶梯</p>
</div>


如图2.1所示，<strong>每一个新范式的出现，都是为了解决上一代范式的核心“痛点”或根本局限。</strong> 而新的解决方案在带来能力飞跃的同时，也引入了新的、在当时难以克服的“局限”，而这又为下一代范式的诞生埋下了伏笔。理解这一“问题驱动”的迭代历程，能帮助我们更深刻地把握现代智能体技术选型背后的深层原因与历史必然性。

## 2.1 基于符号与逻辑的早期智能体

人工智能领域的早期探索，深受数理逻辑和计算机科学基本原理的影响。在那个时代，研究者们普遍持有一种信念：人类的智能，尤其是逻辑推理能力，可以被形式化的符号体系所捕捉和复现。这一核心思想催生了人工智能的第一个重要范式——符号主义（Symbolicism），也被称为“逻辑AI”或“传统AI”。

在符号主义看来，智能行为的核心是基于一套明确规则对符号进行操作。因此，一个智能体可以被视为一个物理符号系统：它通过内部的符号来表示外部世界，并通过逻辑推理来规划行动。这个时代的智能体，其“智慧”完全来源于设计者预先编码的知识库和推理规则，而非通过自主学习获得。

### 2.1.1 物理符号系统假说

符号主义时代的理论根据，是1976年由<strong>艾伦·纽厄尔（Allen Newell）</strong>和<strong>赫伯特·西蒙（Herbert A. Simon）</strong>共同提出的<strong>物理符号系统假说（PhysicalSymbol SystemHypothesis, PSSH）</strong><sup>[1]</sup>。这两位图灵奖得主通过这一假说，为在计算机上实现通用人工智能提供了理论指导和判定标准。

该假说包含两个核心论断：

1. <strong>充分性论断</strong>：任何一个物理符号系统，都具备产生通用智能行为的充分手段。
2. <strong>必要性论断</strong>：任何一个能够展现通用智能行为的系统，其本质必然是一个物理符号系统。

这里的物理符号系统指的是一个能够在物理世界中存在的系统，它由一组可被区分的符号和一系列对这些符号进行操作的过程组成，其构成元素如图2.2所示。这些符号可以组合成更复杂的结构（例如表达式），而过程则可以创建、修改、复制和销毁这些符号结构。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-0.png" alt="图片描述" width="90%"/>
  <p>图 2.2 物理符号系统的构成元素</p>
</div>


简而言之，PSSH大胆地宣称：<strong>智能的本质，就是符号的计算与处理。</strong>

这个假说具有深远的影响。它将对人类心智这一模糊、复杂的哲学问题的研究，转化为了一个可以在计算机上进行工程化实现的具体问题。它为早期人工智能研究者注入了强大的信心，即只要我们能找到正确的方式来表示知识并设计出有效的推理算法，就一定能创造出与人类媲美的机器智能。整个符号主义时代的研究，从专家系统到自动规划，几乎都是在这一假说的指引下展开的。

### 2.1.2 专家系统

在物理符号系统假说的直接影响下，<strong>专家系统（Expert System）</strong>成为符号主义时代最重要、最成功的应用成果。专家系统的核心目标，是模拟人类专家在特定领域内解决问题的能力。它通过将专家的知识和经验编码成计算机程序，使其能够在面对相似问题时，给出媲美甚至超越人类专家的结论或建议。

一个典型的专家系统通常由知识库、推理机、用户界面等几个核心部分构成，其通用架构如图2.3所示。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-1.png" alt="图片描述" width="90%"/>
  <p>图 2.3 专家系统的通用架构</p>
</div>



这种架构清晰地体现了知识与推理相分离的设计思想，是符号主义AI的重要特征。

<strong>知识库与推理机</strong>

专家系统的“智能”主要源于其两大核心组件：知识库和推理机。

- <strong>知识库（Knowledge Base）</strong>：这是专家系统的知识存储中心，用于存放领域专家的知识和经验。<strong>知识表示（Knowledge Representation）</strong>是构建知识库的关键。在专家系统中，最常用的一种知识表示方法是<strong>产生式规则（Production Rules）</strong>，即一系列“IF-THEN”形式的条件语句。例如：IF 病人有发烧症状 AND 咳嗽 THEN 可能患有呼吸道感染。这些规则将特定情境（IF部分，条件）与相应的结论或行动（THEN部分，结论）关联起来。一个复杂的专家系统可能包含成百上千条这样的规则，共同构成一个庞大的知识网络。
- <strong>推理机（Inference Engine）</strong>：推理机是专家系统的核心计算引擎。它是一个通用的程序，其任务是根据用户提供的事实，在知识库中寻找并应用相关的规则，从而推导出新的结论。推理机的工作方式主要有两种：
  - <strong>正向链（Forward Chaining）</strong>：从已知事实出发，不断匹配规则的IF部分，触发THEN部分的结论，并将新结论加入事实库，直到最终推导出目标或无新规则可匹配。这是一种“数据驱动”的推理方式。
  - <strong>反向链（Backward Chaining）</strong>：从一个假设的目标（比如“病人是否患有肺炎”）出发，寻找能够推导出该目标的规则，然后将该规则的IF部分作为新的子目标，如此递归下去，直到所有子目标都能被已知事实所证明。这是一种“目标驱动”的推理方式。

<strong>应用案例与分析：MYCIN系统</strong>

MYCIN是历史上最著名、最具影响力的专家系统之一，由斯坦福大学于20世纪70年代开发<sup>[2]</sup>。它被设计用于辅助医生诊断细菌性血液感染并推荐合适的抗生素治疗方案。

- <strong>工作原理</strong>：MYCIN通过与医生进行问答式交互来收集病人的症状、病史和化验结果。其知识库包含了约600条由医学专家提供的“IF-THEN”规则。推理机主要采用反向链的方式工作：从“确定致病菌”这一最高目标出发，反向推导需要哪些证据和条件，然后向医生提问以获取这些信息。其简化的工作流程如图2.4所示。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-2.png" alt="图片描述" width="90%"/>
  <p>图 2.4 MYCIN反向链推理流程示意图</p>
</div>


- <strong>不确定性处理</strong>：医学诊断充满了不确定性。MYCIN的一个重要创新是引入了<strong>置信因子（Certainty Factor, CF）</strong>的概念，用一个-1到1之间的数值来表示一个结论的可信度。这使得系统能够处理不确定的、模糊的医学知识，并给出带有可信度评估的诊断结果，这比简单的布尔逻辑更贴近现实世界。
- <strong>成就与意义</strong>：在一项评估中，MYCIN在血液感染诊断方面的表现超过了非专业医生，甚至达到了人类专家的水平。它的成功雄辩地证明了物理符号系统假说的有效性：通过精心的知识工程和符号推理，机器确实可以在高度复杂的专业领域展现出卓越的“智能”。MYCIN不仅是专家系统发展史上的一个里程碑，也为后续人工智能在各个垂直领域的商业化应用铺平了道路。

### 2.1.3 SHRDLU

如果说专家系统展示了符号AI在专业领域的“深度”，那么由<strong>特里·威诺格拉德（Terry Winograd）</strong>于1968-1970年开发的SHRDLU项目<sup>[3]</sup>，则在“广度”上实现了革命性的突破。如图2.5所示，SHRDLU旨在构建一个能在“积木世界”这一微观环境中，通过自然语言与人类流畅交互的综合性智能体。“积木世界”是一个模拟的三维虚拟空间，其中包含不同形状、颜色和大小的积木，以及一个可以抓取和移动它们的虚拟机械臂。用户通过自然语言向SHRDLU下达指令或提问，SHRDLU则在虚拟世界中执行动作或给出文字回答。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-3.png" alt="图片描述" width="90%"/>
  <p>图 2.5 SHRDLU的“积木世界”交互界面</p>
</div>


SHRDLU在当时引起广泛关注，主要原因在于它首次将多个独立的人工智能模块（如语言解析、规划、记忆）集成在一个统一的系统中，并使它们协同工作：

- <strong>自然语言理解</strong>：SHRDLU能够解析结构复杂且含有歧义的英语句子。它不仅能理解直接的命令（如 `Pick up a big red block.`），还能处理更复杂的指令，例如：
  - 指代消解：`Find a block which is taller than the one you are holding and put it into the box.` 在这条指令中，系统需要理解 `the one you are holding` 指代的是当前机械臂正抓取的物体。
  - 上下文记忆：用户可以说 `Grasp the pyramid.`，然后接着问 `What does the box contain?`，系统能够联系上下文进行回答。
- <strong>规划与行动</strong>：在理解指令后，SHRDLU能够自主规划出一系列必要的动作来完成任务。例如，如果指令是“把蓝色积木放到红色积木上”，而红色积木上已经有另一个绿色积木，系统会规划出“先把绿色积木移开，再把蓝色积木放上去”的动作序列。
- <strong>记忆与问答</strong>：SHRDLU拥有关于其所处环境和自身行为的记忆。用户可以就此提问，例如：
  - 询问世界状态：`Is there a large block behind a pyramid?`
  - 询问行为历史：`Did you touch any pyramid before you put the green one on the little cube?`
  - 询问行为动机：`Why did you pick up the red block?` SHRDLU可以回答：`BECAUSE YOU ASKED ME TO.`

SHRDLU的历史地位与影响主要体现在三个方面：

- <strong>综合性智能的典范</strong>：在SHRDLU之前，AI研究大多聚焦于单一功能。它首次将语言理解、推理规划与行动记忆等多个AI模块集成于统一系统，其“感知-思考-行动”的闭环设计，奠定了现代智能体研究的基础。
- <strong>微观世界研究方法的普及</strong>：它的成功证明了在一个规则明确的简化环境中，探索和验证复杂智能体基本原理的可行性，这一方法深刻影响了后续的机器人学与AI规划研究。
- <strong>引发的乐观与反思</strong>：SHRDLU的成功激发了对AGI的早期乐观预期，但其能力又严格局限于积木世界。这种局限性引发了AI领域关于“符号处理”与“真正理解”之间差异的长期思辨，揭示了通往通用智能的深层挑战。

### 2.1.4 符号主义面临的根本性挑战

尽管早期项目成就显著，但从20世纪80年代起，符号主义AI在从“微观世界”走向开放、复杂的现实世界时，遇到了其方法论固有的根本性难题。这些难题主要可归结为两大类：

<strong>（1）常识知识与知识获取瓶颈</strong>

符号主义智能体的“智能”完全依赖于其知识库的质量和完备性。然而，如何构建一个能够支撑真实世界交互的知识库，被证明是一项极其艰巨的任务，主要体现在两个方面：

- <strong>知识获取瓶颈（Knowledge Acquisition Bottleneck）</strong>：专家系统的知识需要由人类专家和知识工程师通过繁琐的访谈、提炼和编码过程来构建。这个过程成本高昂、耗时漫长，且难以规模化。更重要的是，人类专家的许多知识是内隐的、直觉性的，很难被清晰地表达为“IF-THEN”规则。试图将整个世界的知识都进行手工符号化，被认为是一项几乎不可能完成的任务。
- <strong>常识问题（Common-sense Problem）</strong>：人类行为依赖于庞大的常识背景（例如，“水是湿的”、“绳子可以拉不能推”），但符号系统除非被明确编码，否则对此一无所知。为广阔、模糊的常识建立完备的知识库至今仍是重大挑战，Cyc项目<sup>[4]</sup>历经数十年努力，其成果和应用仍然非常有限。

<strong>（2）框架问题与系统脆弱性</strong>

除了知识层面的挑战，符号主义在处理动态变化的世界时也遇到了逻辑上的困境。

- <strong>框架问题（Frame Problem）</strong>：在一个动态世界中，智能体执行一个动作后，如何高效判断哪些事物未发生改变是一个逻辑难题<sup>[5]</sup>。为每个动作显式地声明所有不变的状态，在计算上是不可行的，而人类却能毫不费力地忽略不相关的变化。
- <strong>系统脆弱性（Brittleness）</strong>：符号系统完全依赖预设规则，导致其行为非常“脆弱”。一旦遇到规则之外的任何微小变化或新情况，系统便可能完全失灵，无法像人类一样灵活变通。SHRDLU的成功，也正是因为它运行在一个规则完备的封闭世界里，而真实世界充满了例外。

## 2.2 构建基于规则的聊天机器人

在探讨了符号主义的理论挑战后，本节我们将通过一个具体的编程实践，来直观地感受基于规则的系统是如何工作的。我们将尝试复现人工智能历史上一个极具影响力的早期聊天机器人——ELIZA。

### 2.2.1 ELIZA 的设计思想

ELIZA是由麻省理工学院的计算机科学家<strong>约瑟夫·魏泽鲍姆（Joseph Weizenbaum）</strong>于1966年发布的一个计算机程序<sup>[6]</sup>，是早期自然语言处理领域的著名尝试之一。ELIZA并非一个单一的程序，而是一个可以执行不同“脚本”的框架。其中，最广为人知也最成功的脚本是“DOCTOR”，它模仿了一位罗杰斯学派的非指导性心理治疗师。

ELIZA的工作方式极其巧妙：它从不正面回答问题或提供信息，而是通过识别用户输入中的关键词，然后应用一套预设的转换规则，将用户的陈述转化为一个开放式的提问。例如，当用户说“我为我的男朋友感到难过”时，ELIZA可能会识别出关键词“我为……感到难过”，并应用规则生成回应：“你为什么会为你的男朋友感到难过？”

魏泽鲍姆的设计思想并非要创造一个真正能够“理解”人类情感的智能体，恰恰相反，他想证明的是，通过一些简单的句式转换技巧，机器可以在完全不理解对话内容的情况下，营造出一种“智能”和“共情”的假象。然而，出乎他意料的是，许多与ELIZA交互过的人（包括他的秘书）都对其产生了情感上的依赖，深信它能够理解自己。

本节的实践目标即为复现ELIZA的核心机制，以深入理解这种规则驱动方法的优势与根本局限。

### 2.2.2 模式匹配与文本替换

ELIZA的算法流程基于<strong>模式匹配（Pattern Matching）与文本替换（Text Substitution）</strong>，可被清晰地分解为以下四个步骤：

1. <strong>关键词识别与排序：</strong>规则库为每个关键词（如 `mother`, `dreamed`, `depressed`）设定一个优先级。当输入包含多个关键词时，程序会选择优先级最高的关键词所对应的规则进行处理。
2. <strong>分解规则：</strong>找到关键词后，程序使用带通配符（`*`）的分解规则来捕获句子的其余部分。
   1. <strong>规则示例</strong>： `* my *`
   2. <strong>用户输入</strong>： `"My mother is afraid of me"`
   3. <strong>捕获结果</strong>： `["", "mother is afraid of me"]`
3. <strong>重组规则：</strong>程序从与分解规则关联的一组重组规则中，选择一条来生成回应（通常随机选择以增加多样性），并可选择性地使用上一步捕获的内容。
   1. <strong>规则示例</strong>： `"Tell me more about your family."`
   2. <strong>生成输出</strong>： `"Tell me more about your family."`
4. <strong>代词转换：</strong>在重组前，程序会进行简单的代词转换（如 `I` → `you`, `my` → `your`），以维持对话的连贯性。

整个工作流程可以用一个简单的伪代码思路来表示：

```Python
FUNCTION generate_response(user_input):
    // 1. 将用户输入拆分成单词
    words = SPLIT(user_input)

    // 2. 寻找优先级最高的关键词规则
    best_rule = FIND_BEST_RULE(words)
    IF best_rule is NULL:
        RETURN a_generic_response() // 例如:"Please go on."

    // 3. 使用规则分解用户输入
    decomposed_parts = DECOMPOSE(user_input, best_rule.decomposition_pattern)
    IF decomposition_failed:
        RETURN a_generic_response()

    // 4. 对分解出的部分进行代词转换
    transformed_parts = TRANSFORM_PRONOUNS(decomposed_parts)

    // 5. 使用重组规则生成回应
    response = REASSEMBLE(transformed_parts, best_rule.reassembly_patterns)

    RETURN response
```

通过这套机制，ELIZA成功地将复杂的自然语言理解问题，简化为了一个可操作的、基于规则的模式匹配游戏。

### 2.2.3 核心逻辑的实现

现在，我们将上一节描述的技术原理转化为一个简单的、可运行的Python函数。下面的代码实现了一个迷你版的ELIZA，它包含了一小部分规则，但足以展示其核心工作机制。

```Python
import re
import random

# 定义规则库:模式(正则表达式) -> 响应模板列表
rules = {
    r'I need (.*)': [
        "Why do you need {0}?",
        "Would it really help you to get {0}?",
        "Are you sure you need {0}?"
    ],
    r'Why don\'t you (.*)\?': [
        "Do you really think I don't {0}?",
        "Perhaps eventually I will {0}.",
        "Do you really want me to {0}?"
    ],
    r'Why can\'t I (.*)\?': [
        "Do you think you should be able to {0}?",
        "If you could {0}, what would you do?",
        "I don't know -- why can't you {0}?"
    ],
    r'I am (.*)': [
        "Did you come to me because you are {0}?",
        "How long have you been {0}?",
        "How do you feel about being {0}?"
    ],
    r'.* mother .*': [
        "Tell me more about your mother.",
        "What was your relationship with your mother like?",
        "How do you feel about your mother?"
    ],
    r'.* father .*': [
        "Tell me more about your father.",
        "How did your father make you feel?",
        "What has your father taught you?"
    ],
    r'.*': [
        "Please tell me more.",
        "Let's change focus a bit... Tell me about your family.",
        "Can you elaborate on that?"
    ]
}

# 定义代词转换规则
pronoun_swap = {
    "i": "you", "you": "i", "me": "you", "my": "your",
    "am": "are", "are": "am", "was": "were", "i'd": "you would",
    "i've": "you have", "i'll": "you will", "yours": "mine",
    "mine": "yours"
}

def swap_pronouns(phrase):
    """
    对输入短语中的代词进行第一/第二人称转换
    """
    words = phrase.lower().split()
    swapped_words = [pronoun_swap.get(word, word) for word in words]
    return " ".join(swapped_words)

def respond(user_input):
    """
    根据规则库生成响应
    """
    for pattern, responses in rules.items():
        match = re.search(pattern, user_input, re.IGNORECASE)
        if match:
            # 捕获匹配到的部分
            captured_group = match.group(1) if match.groups() else ''
            # 进行代词转换
            swapped_group = swap_pronouns(captured_group)
            # 从模板中随机选择一个并格式化
            response = random.choice(responses).format(swapped_group)
            return response
    # 如果没有匹配任何特定规则，使用最后的通配符规则
    return random.choice(rules[r'.*'])

# 主聊天循环
if __name__ == '__main__':
    print("Therapist: Hello! How can I help you today?")
    while True:
        user_input = input("You: ")
        if user_input.lower() in ["quit", "exit", "bye"]:
            print("Therapist: Goodbye. It was nice talking to you.")
            break
        response = respond(user_input)
        print(f"Therapist: {response}")
        
>>>
Therapist: Hello! How can I help you today?
You: I am feeling sad today.
Therapist: How long have you been feeling sad today?
You: I need some help with my project.
Therapist: Are you sure you need some help with your project?
You: My mother is not happy with my work.
Therapist: Tell me more about your mother.
You: quit
Therapist: Goodbye. It was nice talking to you.
```

通过上述的编程实践，我们可以直观地总结出规则驱动系统的根本局限性，这些局限正是对 `2.1.4` 节中符号主义理论挑战的直接印证：

- <strong>缺乏语义理解</strong>：系统不理解词义。例如，面对“I am <strong>not</strong> happy”的输入，它仍会机械地匹配 `I am (.*)` 规则并生成语义不通的回应，因为它无法理解否定词“not”的作用。
- <strong>无上下文记忆</strong>：系统是<strong>无状态的（Stateless）</strong>，每次回应仅基于当前单句输入，无法进行连贯的多轮对话。
- <strong>规则的扩展性问题</strong>：尝试增加更多规则会导致规则库的规模爆炸式增长，规则间的冲突与优先级管理将变得极其复杂，最终导致系统难以维护。

然而，尽管存在这些显而易见的缺陷，ELIZA在当时却产生了著名的“<strong>ELIZA效应</strong>”，许多用户相信它能理解自己。这种智能的幻觉主要源于其巧妙的对话策略（如扮演被动的提问者、使用开放式模板）以及人类天生的情感投射心理。

ELIZA的实践清晰地揭示了符号主义方法的核心矛盾：系统看似智能的表现，完全依赖于设计者预先编码的规则。然而，面对真实世界语言的无限可能性，这种穷举式的方法注定不可扩展。系统没有真正的理解，只是在执行符号操作，这正是其脆弱性的根源。

## 2.3 马文·明斯基的心智社会

符号主义的探索和ELIZA的实践，共同指向了一个问题：通过预设规则构建的、单一的、集中的推理引擎，似乎难以通向真正的智能。无论规则库多么庞大，系统在面对真实世界的模糊性、复杂性和无穷变化时，总是显得僵化而脆弱。这一困境促使一些顶尖的思考者开始反思人工智能最底层的设计哲学。其中，<strong>马文·明斯基（Marvin Minsky）</strong>没有继续尝试为单一推理核心添加更多规则，而是在他的<strong>《心智社会》（The Society of Mind）</strong><sup>[7]</sup> 一书中提出了一个革命性的问题："What magical trick makes us intelligent? The trick is that there is no trick. The power of intelligence stems from our vast diversity, not from any single, perfect principle."

### 2.3.1 对单一整体智能模型的反思

20世纪70至80年代，符号主义的局限性日益明显。专家系统虽然在高度垂直的领域取得了成功，但它们无法拥有儿童般的常识；SHRDLU虽然能在一个封闭的积木世界中表现出色，但它无法理解这个世界之外的任何事情；ELIZA虽然能模仿对话，但它对对话内容本身一无所知。这些系统都遵循着一种自上而下（Top-down）的设计思路：一个全知全能的中央处理器，根据一套统一的逻辑规则来处理信息和做出决策。

面对这种普遍的失败，明斯基开始提出一系列根本性的问题：

- <strong>“理解”是什么？</strong> 当我们说我们理解一个故事时，这是一种单一的能力吗？还是说，它其实是视觉化能力、逻辑推理能力、情感共鸣能力、社会关系常识等数十种不同心智过程协同工作的结果？
- <strong>“常识”是什么？</strong> 常识是一个包含了数百万条逻辑规则的庞大知识库吗（如Cyc项目的尝试）？还是说，它是一种分布式的、由无数具体经验和简单规则片段交织而成的网络？
- <strong>智能体应该如何构建？</strong> 我们是否应该继续追求一个完美的、统一的逻辑系统，还是应该承认，智能本身就是“不完美”的、由许多功能各异、甚至会彼此冲突的简单部分组成的大杂烩？

这些问题直指单一整体智能模型的核心弊端。该类模型试图用一种统一的表示和推理机制来解决所有问题，但这与我们观察到的自然智能（尤其是人类智能）的运作方式相去甚远。明斯基认为，强行将多样化的心智活动塞进一个僵化的逻辑框架中，正是导致早期人工智能研究停滞不前的根源。

正是基于这样的反思，明斯基提出了一个颠覆性的构想，他不再将心智视为一个金字塔式的层级结构，而是将其看作一个扁平化的、充满了互动与协作的“社会”。

### 2.3.2 作为协作体的智能

在明斯基的理论框架中，智能体的定义与我们第一章讨论的现代智能体有所不同。这里的智能体指的是一个极其简单的、专门化的心智过程，它自身是“无心”的。例如，一个负责识别线条的`LINE-FINDER`智能体，或一个负责抓握的`GRASP`智能体。

这些简单的智能体被组织起来，形成功能更强大的<strong>机构（Agency）</strong>。一个机构是一组协同工作的智能体，旨在完成一个更复杂的任务。例如，一个负责搭积木的`BUILD`机构，可能由`SEE`、`FIND`、`GET`、`PUT`等多个下层智能体或机构组成。它们之间通过去中心化的激活与抑制信号相互影响，形成动态的控制流。

<strong>涌现（Emergence）</strong>是理解心智社会理论的关键。复杂的、有目的性的智能行为，并非由某个高级智能体预先规划，而是从大量简单的底层智能体之间的局部交互中自发产生的。

让我们以经典的“搭建积木塔”任务为例，来说明这一过程，如图2.6所示。当一个高层目标（如“我要搭一个塔”）出现时，它会激活一个名为`BUILD-TOWER`的高层机构。

1. `BUILD-TOWER`机构并不知道如何执行具体的物理动作，它的唯一作用是激活它的下属机构，比如`BUILDER`。
2. `BUILDER`机构同样很简单，它可能只包含一个循环逻辑：只要塔还没搭完，就激活`ADD-BLOCK`机构。
3. `ADD-BLOCK`机构则负责协调更具体的子任务，它会依次激活`FIND-BLOCK`、`GET-BLOCK`和`PUT-ON-TOP`这三个子机构。
4. 每一个子机构又由更底层的智能体构成。例如，`GET-BLOCK`机构会激活视觉系统中的`SEE-SHAPE`智能体、运动系统中的`REACH`和`GRASP`智能体。

在这个过程中，没有任何一个智能体或机构拥有整个任务的全局规划。`GRASP`只负责抓握，它不知道什么是塔；`BUILDER`只负责循环，它不知道如何控制手臂。然而，当这个由无数“无心”的智能体组成的社会，通过简单的激活和抑制规则相互作用时，一个看似高度智能的行为，搭建积木塔，就自然而然地涌现了出来。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-4.png" alt="图片描述" width="90%"/>
  <p>图 2.6 “心智社会”中搭建积木塔行为的涌现机制示意图</p>
</div>


### 2.3.3 对多智能体系统的理论启发

心智社会理论最深远的影响，在于它为<strong>分布式人工智能（Distributed Artificial Intelligence, DAI）</strong>以及后来的<strong>多智能体系统（Multi-Agent System, MAS）</strong>提供了重要的概念基础。它引出研究者们的思考：

<strong>如果一个心智内部的智能，是通过大量简单智能体的协作而涌现的，那么，在多个独立的、物理上分离的计算实体（计算机、机器人）之间，是否也能通过协作涌现出更强大的“群体智能”？</strong>

这个问题的提出，直接将研究焦点从“如何构建一个全能的单一智能体”转向了“如何设计一个高效协作的智能体群体”。具体而言，心智社会在以下几个方面直接启发了多智能体系统的研究：

- <strong>去中心化控制（Decentralized Control）</strong>：理论的核心在于不存在中央控制器。这一思想被MAS领域完全继承，如何设计没有中心节点的协调机制和任务分配策略，成为了MAS的核心研究课题之一。
- <strong>涌现式计算（Emergent Computation）</strong>：复杂问题的解决方案可以从简单的局部交互规则中自发产生。这启发了MAS中大量基于涌现思想的算法，如蚁群算法、粒子群优化等，用于解决复杂的优化和搜索问题。
- <strong>智能体的社会性（Agent Sociality）</strong>：明斯基的理论强调了智能体之间的交互（激活、抑制）。MAS领域将其进一步扩展，系统地研究智能体之间的通信语言（如ACL）、交互协议（如契约网）、协商策略、信任模型乃至组织结构，从而构建起真正的计算社会。

可以说，明斯基的“心智社会”理论，为AI研究者理解“群体智能”的内在构造提供了重要的分析框架。它为后来的研究者们提供了一套全新的视角，去探索由独立的、自治的、具备社会能力的计算智能体所构成的复杂系统，从而正式开启了多智能体系统研究的序幕。

## 2.4 学习范式的演进与现代智能体

前文探讨的“心智社会”理论，在哲学层面为群体智能和去中心化协作指明了方向，但实现路径尚不明确。与此同时，符号主义在应对真实世界复杂性时暴露的根本性挑战也表明仅靠预先编码的规则无法构建真正鲁棒的智能。

这两条线索共同指向了一个问题：如果智能无法被完全设计，那么它是否可以被学习出来？

这一设问开启了人工智能的“学习”时代。其核心目标不再是手动编码知识，而是构建能从经验和数据中自动获取知识与能力的系统。本节将追溯这一范式的演进历程：从联结主义奠定的学习基础，到强化学习实现的交互式学习，直至今日由大型语言模型驱动的现代智能体。

### 2.4.1 从符号到联结

作为对符号主义局限性的直接回应，<strong>联结主义（Connectionism）</strong>在20世纪80年代重新兴起。与符号主义自上而下、依赖明确逻辑规则的设计哲学不同，联结主义是一种自下而上的方法，其灵感来源于对生物大脑神经网络结构的模仿<sup>[8]</sup>。它的核心思想可以概括为以下几点：

1. <strong>知识的分布式表示</strong>：知识并非以明确的符号或规则形式存储在某个知识库中，而是以连接权重的形式，分布式地存储在大量简单的处理单元（即人工神经元）的连接之间。整个网络的连接模式本身就构成了知识。
2. <strong>简单的处理单元</strong>：每个神经元只执行非常简单的计算，如接收来自其他神经元的加权输入，通过一个激活函数进行处理，然后将结果输出给下一个神经元。
3. <strong>通过学习调整权重</strong>：系统的智能并非来自于设计者预先编写的复杂程序，而是来自于“学习”过程。系统通过接触大量样本，根据某种学习算法（如反向传播算法）自动、迭代地调整神经元之间的连接权重，从而使得整个网络的输出逐渐接近期望的目标。

在这种范式下，智能体不再是一个被动执行规则的逻辑推理机，而是一个能够通过经验自我优化的适应性系统。如图2.7所示，这代表了构建智能体核心思想的根本性转变。符号主义试图将人类的知识显式地编码给机器，而联结主义则试图创造出能够像人类一样学习知识的机器。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-5.png" alt="图片描述" width="90%"/>
  <p>图 2.7 符号主义与联结主义范式对比</p>
</div>


联结主义的兴起，特别是深度学习在21世纪的成功，为智能体赋予了强大的感知和模式识别能力，使其能够直接从原始数据（如图像、声音、文本）中理解世界，这是符号主义时代难以想象的。然而，如何让智能体学会在与环境的动态交互中做出最优的序贯决策，则需要另一种学习范式的补充。

### 2.4.2 基于强化学习的智能体

联结主义主要解决了感知问题（例如，“这张图片里有什么？”），但智能体更核心的任务是进行决策（例如，“在这种情况下，我应该做什么？”）。<strong>强化学习（Reinforcement Learning, RL）</strong>正是专注于解决序贯决策问题的学习范式。它并非直接从标注好的静态数据集中学习，而是通过智能体与环境的直接交互，在“试错”中学习如何最大化其长期收益。

以AlphaGo为例，其核心的自我对弈学习过程便是强化学习的经典体现<sup>[9]</sup>。在这个过程中，AlphaGo（智能体）通过观察棋盘的当前布局（环境状态），决定下一步棋的落子位置（行动）。一局棋结束后，根据胜负结果，它会收到一个明确的信号：赢了就是正向奖励，输了则是负向奖励。通过数百万次这样的自我对弈，AlphaGo不断调整其内部策略，逐渐学会了在何种棋局下选择何种行动，最有可能导向最终的胜利。这个过程完全是自主的，不依赖于人类棋谱的直接指导。

这种通过与环境互动、根据反馈信号来优化自身行为的学习机制，就是强化学习的核心框架。下面我们将详细拆解其基本构成要素和工作模式。

强化学习的框架可以用几个核心要素来描述：

- <strong>智能体（Agent）</strong>：学习者和决策者。在AlphaGo的例子中，就是其决策程序。
- <strong>环境（Environment）</strong>：智能体外部的一切，是智能体与之交互的对象。对AlphaGo而言，就是围棋的规则和对手。
- <strong>状态（State, S）</strong>：对环境在某一时刻的特定描述，是智能体做出决策的依据。例如，棋盘上所有棋子的当前位置。
- <strong>行动（Action, A）</strong>：智能体根据当前状态所能采取的操作。例如，在棋盘的某个合法位置上落下一子。
- <strong>奖励（Reward, R）</strong>：环境在智能体执行一个行动后，反馈给智能体的一个标量信号，用于评价该行动在特定状态下的好坏。例如，在一局棋结束后，胜利获得+1的奖励，失败获得-1的奖励。

基于上述核心要素，强化学习智能体在一个“感知-行动-学习”的闭环中持续迭代，其工作模式如图2.8所示。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-6.png" alt="图片描述" width="90%"/>
  <p>图 2.8 强化学习的核心交互循环</p>
</div>


这个循环的具体步骤如下：

1. 在时间步t，智能体观察到环境的当前状态$S_{t}$。
2. 基于状态 $S_{t}$，智能体根据其内部的<strong>策略（Policy, π）</strong>选择一个行动 $A_{t}$ 并执行它。策略本质上是一个从状态到行动的映射，定义了智能体的行为方式。
3. 环境接收到行动 $A_{t}$ 后，会转移到一个新的状态 $S_{t+1}$。
4. 同时，环境会反馈给智能体一个即时奖励 $R_{t+1}$。
5. 智能体利用这个反馈（新状态 $S_{t+1}$ 和奖励 $R_{t+1}$）来更新和优化其内部策略，以便在未来做出更好的决策。这个更新过程就是学习。

智能体的学习目标，并非最大化某一个时间步的即时奖励，而是最大化从当前时刻开始到未来的<strong>累积奖励（Cumulative Reward）</strong>，也称为<strong>回报（Return）</strong>。这意味着智能体需要具备“远见”，有时为了获得未来更大的奖励，需要牺牲当前的即时奖励（例如，围棋中的“弃子”策略）。通过在上述循环中不断探索、收集反馈并优化策略，智能体最终能够学会在复杂动态环境中进行自主决策和长期规划。

### 2.4.3 基于大规模数据的预训练

强化学习赋予了智能体从交互中学习决策策略的能力，但这通常需要海量的、针对特定任务的交互数据，导致智能体在学习之初缺乏先验知识，需要从零开始构建对任务的理解。无论是符号主义试图手动编码的常识，还是人类在决策时所依赖的背景知识，在RL智能体中都是缺失的。如何让智能体在开始学习具体任务前，就先具备对世界的广泛理解？这一问题的解决方案，最终在<strong>自然语言处理（Natural Language Processing, NLP）</strong>领域中浮现，其核心便是基于大规模数据的<strong>预训练（Pre-training）</strong>。

<strong>从特定任务到通用模型</strong>

在预训练范式出现之前，传统的自然语言处理模型通常是为单一特定任务（如情感分析、机器翻译）在专门标注的中小规模数据集上从零开始独立训练的。这种模式导致了几个问题：模型的知识面狭窄，难以将在一个任务中学到的知识泛化到另一个任务，并且每一个新任务都需要耗费大量的人力去标注数据。预训练与微调（Pre-training, Fine-tuning）范式的提出彻底改变了这一现状。其核心思想分为两步：

1. <strong>预训练阶段</strong>：首先在一个包含互联网级别海量文本数据的通用语料库上，通过<strong>自监督学习（Self-supervised Learning）</strong>的方式训练一个超大规模的神经网络模型。这个阶段的目标不是完成任何特定任务，而是学习语言本身内在的规律、语法结构、事实知识以及上下文逻辑。最常见的目标是“预测下一个词”。
2. <strong>微调阶段</strong>：完成预训练后，这个模型就已经学习到了和数据集有关的丰富知识。之后，针对特定的下游任务，只需使用少量该任务的标注数据对模型进行微调，即可让模型适应对应任务。

如图2.9所示，直观地展示了这一预训练与微调的完整流程：通用文本数据经过自监督学习形成基础模型，随后通过特定任务数据进行微调，最终适应各项下游任务。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-7.png" alt="图片描述" width="90%"/>
  <p>图 2.9 “预训练-微调”范式示意图</p>
</div>


<strong>大型语言模型的诞生与涌现能力</strong>

通过在数万亿级别的文本上进行预训练，大型语言模型的神经网络权重实际上已经构建了一个关于世界知识的、高度压缩的隐式模型。它以一种全新的方式，解决了符号主义时代最棘手的“知识获取瓶颈”问题。更令人惊讶的是，当模型的规模（参数量、数据量、计算量）跨越某个阈值后，它们开始展现出未被直接训练的、预料之外的<strong>涌现能力（Emergent Abilities）</strong>，例如：

- <strong>上下文学习（In-context Learning）</strong>：无需调整模型权重，仅在输入中提供<strong>几个示例（Few-shot）</strong>甚至<strong>零个示例（Zero-shot）</strong>，模型就能理解并完成新的任务。
- <strong>思维链（Chain-of-Thought）推理</strong>：通过引导模型在回答复杂问题前，先输出一步步的推理过程，可以显著提升其在逻辑、算术和常识推理任务上的准确性。

这些能力的出现，标志着LLM不再仅仅是一个语言模型，它已经演变成了一个兼具海量知识库和通用推理引擎双重角色的组件。

至此，智能体发展的历史长河中，几大关键的技术拼图已经悉数登场：符号主义提供了逻辑推理的框架，联结主义和强化学习提供了学习与决策的能力，而大型语言模型则提供了前所未有的、通过预训练获得的世界知识和通用推理能力。下一节，我们将看到这些技术是如何在现代智能体的设计中融为一体的。

### 2.4.4 基于大语言模型的智能体

随着大型语言模型技术的飞速发展，以LLM为核心的智能体已成为人工智能领域的新范式。它不仅能够理解和生成人类语言，更重要的是，能够通过与环境的交互，自主地感知、规划、决策和执行任务。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-8.png" alt="图片描述" width="90%"/>
  <p>图 2.10 LLM驱动的智能体核心组件架构</p>
</div>


如第一章所述，智能体与环境的交互可以被抽象为一个核心循环。LLM驱动的智能体通过一个由多个模块协同工作的、持续迭代的闭环流程来完成任务。该流程遵循图2.10所示的架构，具体步骤如下：

1. <strong>感知 (Perception)</strong> ：流程始于<strong>感知模块 (Perception Module)</strong>。它通过传感器从<strong>外部环境 (Environment)</strong> 接收原始输入，形成<strong>观察 (Observation)</strong>。这些观察信息（如用户指令、API返回的数据或环境状态的变化）是智能体决策的起点，处理后将被传递给思考阶段。
2. <strong>思考 (Thought)</strong> ：这是智能体的认知核心，对应图中的<strong>规划模块 (Planning Module)</strong> 和<strong>大型语言模型 (LLM)</strong> 的协同工作。
   - <strong>规划与分解</strong>：首先，规划模块接收观察信息，进行高级策略制定。它通过<strong>反思 (Reflection)</strong> 和<strong>自我批判 (Self-criticism)</strong> 等机制，将宏观目标分解为更具体、可执行的步骤。
   - <strong>推理与决策</strong>：随后，作为中枢的<strong>LLM</strong> 接收来自规划模块的指令，并与<strong>记忆模块 (Memory)</strong> 交互以整合历史信息。LLM进行深度推理，最终决策出下一步要执行的具体操作，这通常表现为一个<strong>工具调用 (Tool Call)</strong>。
3. <strong>行动 (Action)</strong> ：决策完成后，便进入行动阶段，由<strong>执行模块 (Execution Module)</strong> 负责。LLM生成的工具调用指令被发送到执行模块。该模块解析指令，从<strong>工具箱 (Tool Use)</strong> 中选择并调用合适的工具（如代码执行器、搜索引擎、API等）来与环境交互或执行任务。这个与环境的实际交互就是智能体的<strong>行动 (Action)</strong>。
4. <strong>观察 (Observation)</strong> 与循环 ：行动会改变环境的状态，并产生结果。
   - 工具执行后会返回一个<strong>工具结果 (Tool Result)</strong> 给LLM，这构成了对行动效果的直接反馈。同时，智能体的行动改变了环境，从而产生了一个全新的<strong>环境状态</strong>。
   - 这个“工具结果”和“新的环境状态”共同构成了一轮全新的<strong>观察 (Observation)</strong>。这个新的观察会被感知模块再次捕获，同时LLM会根据行动结果<strong>更新记忆 (Memory Update)</strong>，从而启动下一轮“感知-思考-行动”的循环。

这种模块化的协同机制与持续的迭代循环，构成了LLM驱动智能体解决复杂问题的核心工作流。

### 2.4.5 智能体发展关键节点概览

人工智能体的发展史并非一条笔直的单行道，而是几大核心思想流派长达半个多世纪交织、竞争与融合的历程。理解这一历程，有助于我们洞察当前智能体架构范式形成的深刻根源。

这其中，主要有三大思潮主导着不同时期的研究范式：

1. <strong>符号主义 (Symbolism)</strong> ：以<strong>赫伯特·西蒙 (Herbert A. Simon)</strong> 、<strong>明斯基 (Marvin Minsky)</strong> 等先驱为代表，认为智能的核心在于对符号的操作与逻辑推理。这一思想催生了能够理解自然语言指令的SHRDLU、知识驱动的专家系统以及在国际象棋领域取得巨大成功的“深蓝”计算机。
2. <strong>联结主义 (Connectionism)</strong> ：其灵感源于对大脑神经网络的模拟。尽管早期发展受限，但在<strong>杰弗里·辛顿 (Geoffrey Hinton)</strong> 等研究者的推动下，反向传播算法为神经网络的复苏奠定了基础。最终，随着深度学习时代的到来，这一思想通过卷积神经网络、Transformer等模型成为当前的主流。
3. <strong>行为主义 (Behaviorism)</strong> ：强调智能体通过与环境的互动和试错来学习最优策略，其现代化身为强化学习 。从早期的TD-Gammon到与深度学习结合并击败人类顶尖棋手的AlphaGo，这一流派为智能体赋予了从经验中习得复杂决策行为的能力。

进入21世纪20年代，这些思想流派以前所未有的方式深度融合。以GPT系列为代表的大语言模型，其本身是联结主义的产物，却成为了执行符号推理、进行工具调用和规划决策的核心“大脑”，形成了神经-符号结合的现代智能体架构。为了系统性地回顾这一发展脉络，下图2.11梳理了从20世纪50年代至今，人工智能体发展史上的关键理论、项目与事件，为读者提供一个清晰的全局概览，作为本章知识的沉淀。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-9.png" alt="图片描述" width="90%"/>
  <p>图 2.11 智能体发展演进时间线（未完全版）</p>
</div>


得益于大语言模型的突破，智能体技术栈呈现出前所未有的活跃度和多样性。图2.12展示了当前AI Agent领域的一个典型技术栈全貌，涵盖了从底层模型到上层应用的各个环节。

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-10.png" alt="图片描述" width="90%"/>
  <p>图 2.12 AI Agent 技术栈概览</p>
</div>


该技术栈图由Letta公司于2024年11月发布<sup>[10]</sup>，它将AI智能体相关的工具、平台和服务进行了分层与分类，为我们理解当前的市场格局和技术选型提供了宝贵的参考。

## 2.5 本章小结

本章回顾了智能体发展的历史脉络，探索了其核心思想从诞生到演进的过程，内容涵盖了人工智能领域几次关键的范式革命：

- <strong>符号主义的探索与局限</strong>：从人工智能的古典时代出发，本章阐述了以专家系统为代表的早期智能体是如何尝试通过“知识+推理”来模拟智能的。通过亲手构建一个基于规则的聊天机器人，我们深刻体会到这一范式的能力边界及其面临的根本性挑战。
- <strong>分布式智能思想的萌芽</strong>：探讨了马文·明斯基的“心智社会”理论。这一革命性的思想揭示了复杂的整体智能可以从简单的局部单元的交互中涌现，为后续的多智能体系统研究提供了重要的哲学启发。
- <strong>学习范式的演进</strong>：见证了智能体获取能力方式的根本性变革。从联结主义赋予智能体感知世界的能力，到强化学习使其学会在与环境的交互中进行最优决策，再到基于大规模数据预训练的大型语言模型（LLM）为其提供了前所未有的世界知识和通用推理能力。
- <strong>现代智能体的诞生</strong>：最后，我们对LLM驱动智能体进行分析。通过对其核心组件（模型、记忆、规划、工具等）和工作原理的分析，我们理解了历史上的各种技术思想是如何在现代Agent的架构中实现技术融合的。

通过本章的学习，我们不仅理解了第一章所介绍的现代智能体从何而来，更能建立了一个关于智能体技术演进的宏观认知框架。可以发现，智能体的发展并非简单的技术迭代，而是一场关于如何定义“智能”、获取“知识”、进行“决策”的思想变革。

既然现代智能体的核心是大型语言模型，那么深入理解其底层原理便至关重要。下一章将聚焦于大语言模型本身，探讨其基本概念，为后续在多智能体系统中的高级应用打下坚实的基础。

## Eliza程序完整代码

````python
import re
import random

# 定义规则库:模式(正则表达式) -> 响应模板列表
rules = {
    r'I need (.*)': [
        "Why do you need {0}?",
        "Would it really help you to get {0}?",
        "Are you sure you need {0}?"
    ],
    r'Why don\'t you (.*)\?': [
        "Do you really think I don't {0}?",
        "Perhaps eventually I will {0}.",
        "Do you really want me to {0}?"
    ],
    r'Why can\'t I (.*)\?': [
        "Do you think you should be able to {0}?",
        "If you could {0}, what would you do?",
        "I don't know -- why can't you {0}?"
    ],
    r'I am (.*)': [
        "Did you come to me because you are {0}?",
        "How long have you been {0}?",
        "How do you feel about being {0}?"
    ],
    r'.* mother .*': [
        "Tell me more about your mother.",
        "What was your relationship with your mother like?",
        "How do you feel about your mother?"
    ],
    r'.* father .*': [
        "Tell me more about your father.",
        "How did your father make you feel?",
        "What has your father taught you?"
    ],
    r'.*': [
        "Please tell me more.",
        "Let's change focus a bit... Tell me about your family.",
        "Can you elaborate on that?"
    ]
}

# 定义代词转换规则
pronoun_swap = {
    "i": "you", "you": "i", "me": "you", "my": "your",
    "am": "are", "are": "am", "was": "were", "i'd": "you would",
    "i've": "you have", "i'll": "you will", "yours": "mine",
    "mine": "yours"
}

def swap_pronouns(phrase):
    """
    对输入短语中的代词进行第一/第二人称转换
    """
    words = phrase.lower().split()
    swapped_words = [pronoun_swap.get(word, word) for word in words]
    return " ".join(swapped_words)

def respond(user_input):
    """
    根据规则库生成响应
    """
    for pattern, responses in rules.items():
        match = re.search(pattern, user_input, re.IGNORECASE)
        if match:
            # 捕获匹配到的部分
            captured_group = match.group(1) if match.groups() else ''
            # 进行代词转换
            swapped_group = swap_pronouns(captured_group)
            # 从模板中随机选择一个并格式化
            response = random.choice(responses).format(swapped_group)
            return response
    # 如果没有匹配任何特定规则，使用最后的通配符规则
    return random.choice(rules[r'.*'])

# 主聊天循环
if __name__ == '__main__':
    print("Therapist: Hello! How can I help you today?")
    while True:
        user_input = input("You: ")
        if user_input.lower() in ["quit", "exit", "bye"]:
            print("Therapist: Goodbye. It was nice talking to you.")
            break
        response = respond(user_input)
        print(f"Therapist: {response}")
````



## 习题

> <strong>提示</strong>：以下的部分习题没有标准答案，旨在帮助学习者建立对智能体发展历史的系统性理解，并培养"以史为鉴"的技术洞察力。

1. 物理符号系统假说<sup>[1]</sup>是符号主义时代的理论基石。请分析：

   - 该假说的"充分性论断"和"必要性论断"分别是什么含义？
   - 结合本章内容，说明符号主义智能体在实践中遇到的哪些问题对该假说的"充分性"提出了挑战？
   - 大语言模型驱动的智能体是否符合物理符号系统假说？

2. 专家系统MYCIN<sup>[2]</sup>在医疗诊断领域取得了显著成功，但最终并未大规模应用于临床实践。请思考：

   > <strong>提示</strong>：可以从技术、伦理、法律、用户接受度等多个角度分析

   - 除了本章提到的"知识获取瓶颈"和"脆弱性"，还有哪些因素可能阻碍了专家系统在医疗等高风险领域的应用？
   - 如果让现在的你设计一个医疗诊断智能体，你会如何设计系统来克服MYCIN的局限？
   - 在哪些垂直领域中，基于规则的专家系统至今仍然是比深度学习更好的选择？请举例说明。

3. 在2.2节中，我们实现了一个简化版的ELIZA聊天机器人。请在此基础上进行扩展实践：

   > <strong>提示</strong>：这是一道动手实践题，建议实际编写代码

   - 为ELIZA添加3-5条新的规则，使其能够处理更多样化的对话场景（如谈论工作、学习、爱好等）
   - 实现一个简单的"上下文记忆"功能：让ELIZA能够记住用户在对话中提到的关键信息（如姓名、年龄、职业），并在后续对话中引用
   - 对比你扩展后的ELIZA与[ChatGPT](https://chatgpt.com/)，列举至少3个维度上存在的本质差异
   - 为什么基于规则的方法在处理开放域对话时会遇到"组合爆炸"问题并且难以扩展维护？能否使用数学的方法来说明？

4. 马文·明斯基在"心智社会"理论<sup>[7]</sup>中提出了一个革命性的观点：智能源于大量简单智能体的协作，而非单一的完美系统。

   - 在图2.6"搭建积木塔"的例子中，如果 `GRASP` 智能体突然失效了，整个系统会发生什么？这种去中心化架构的优势和劣势是什么？
   - 将"心智社会"理论与现在的一些多智能体系统（如[CAMEL-Workforce](https://docs.camel-ai.org/key_modules/workforce)、[MetaGPT](https://github.com/FoundationAgents/MetaGPT)、[CrewAI](https://github.com/crewAIInc/crewAI)）进行对比，它们之间存在哪些关联和不同之处？
   - 马文·明斯基认为智能体可以是"无心"的简单过程，然而现在的大语言模型和智能体往往都拥有强大的推理能力。这是否意味着"心智社会"理论在大语言模型时代不再适用了？

5. 强化学习与监督学习是两种不同的学习范式。请分析：

   - 用AlphaGo的例子说明强化学习的"试错学习"机制是如何工作的
   - 为什么强化学习特别适合序贯决策问题？它与监督学习在数据需求上有什么本质区别？
   - 现在我们需要训练一个会玩超级马里奥游戏的智能体。如果分别使用监督学习和强化学习，各需要什么数据？哪种方法对于这个任务来说更合适？
   - 在大语言模型的训练过程中，强化学习起到了什么关键性的作用？

6. 预训练-微调范式是现代人工智能领域的重要突破。请深入思考：

   - 为什么说预训练解决了符号主义时代的"知识获取瓶颈"问题？它们在知识表示方式上有什么本质区别？
   - 预训练模型的知识绝大部分来自互联网数据，这可能带来哪些问题？如何缓解以上问题？
   - 你认为"预训练-微调"范式是否可能会被某种新范式取代？或者它会长期存在？

7. 假设你要设计一个"智能代码审查助手"，它能够自动审查代码提交（Pull Request），概括代码的实现逻辑、检查代码质量、发现潜在BUG、提出改进建议。

   - 如果在符号主义时代（1980年代）设计这个系统，你会如何实现？会遇到什么困难？
   - 如果在没有大语言模型的深度学习时代（2015年左右），你会如何实现？
   - 在当前的大语言模型和智能体的时代，你会如何设计这个智能体的架构？它应该包含哪些模块（参考图2.10）？
   - 对比这三个时代的方案，说明智能体技术的演进如何使这个任务从"几乎不可能"变为"可行"



## 参考文献

[1] NEWELL A, SIMON H A. Computer science as empirical inquiry: symbols and search[J]. Communications of the ACM, 1976, 19(3): 113-126.

[2] BUCHANAN B G, SHORTLIFFE E H, ed. Rule-based expert systems: the MYCIN experiments of the Stanford Heuristic Programming Project[M]. Reading, Mass.: Addison-Wesley, 1984.

[3] WINOGRAD T. Understanding natural language[M]. New York: Academic Press, 1972.

[4] LENAT D B, GUHA R V. Cyc: a midterm report[J]. AI magazine, 1990, 11(3): 32.

[5] MCCARTHY J, HAYES P J. Some philosophical problems from the standpoint of artificial intelligence[C]//MELTZER B, MICHIE D, ed. Machine intelligence 4. Edinburgh: Edinburgh University Press, 1969: 463-502.

[6] WEIZENBAUM J. ELIZA: a computer program for the study of natural language communication between man and machine[J]. Communications of the ACM, 1966, 9(1): 36-45.

[7] MINSKY M. The society of mind[M]. New York: Simon & Schuster, 1986.

[8] RUMELHART D E, MCCLELLAND J L, PDP RESEARCH GROUP. Parallel distributed processing: explorations in the microstructure of cognition[M]. Cambridge, MA: MIT Press, 1986.

[9] SILVER D, HUANG A, MADDISON C J, ed. Mastering the game of Go with deep neural networks and tree search[J]. Nature, 2016, 529(7587): 484-489.

[10] LETTA. The AI agents stack[EB/OL]. (2024-11) [2025-09-07]. https://www.letta.com/blog/ai-agents-stack.

