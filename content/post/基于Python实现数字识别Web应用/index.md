---
title: 基于Python实现数字识别Web应用
description: 本文介绍使用Python实现数字识别Web应用开发的过程。
date: 2026-01-05T12:34:25+08:00
draft: false
categories:
- APP
- 编程
- 开发
tags:
- Python
- Web
- Gradio
---

# 手把手教你：训练 MNIST 手写数字识别模型，搭建可直接使用的 Web 识别工具

在手写数字识别的实践中，很多同学会遇到模型权重与 Web 应用不兼容的问题，最常见的就是通道数不匹配、模型结构冲突导致的运行报错。今天这篇博客，我们就来完整走一遍流程 —— 从从头训练兼容的`cnn_model_basic.pth`权重文件，到搭建一个无需前端经验的 Gradio Web 识别工具，全程避坑，确保最终成果可以直接落地使用。

## 第一部分：从头训练 MNIST 模型，生成兼容权重文件`cnn_model_basic.pth`

我们的核心目标是训练出一个与后续 Web 应用完全兼容的模型权重，彻底解决`expected 1 channels, but got 3 channels`这类兼容性错误，同时保证模型具备较高的识别准确率。

### 一、训练代码核心亮点（保障与 Web 应用无缝兼容）

1. **模型结构完全一致，杜绝权重加载冲突**
    
    训练过程中定义的`MNIST_CNN`类，与后续 Gradio Web 应用中的模型结构、层名称、维度计算完全复刻，从根源上解决权重加载时的`key`不匹配问题。无论是卷积层的通道数、全连接层的神经元数量，还是前向传播的流程，都保持高度统一，确保权重文件可以被 Web 应用直接解析。
    
2. **输入通道严格锁定单通道，解决核心报错**
    
    这是解决通道数不匹配错误的关键：
    
    - 卷积层`conv1`明确设置输入通道为`1`，对应 MNIST 数据集的灰度图格式，与 Web 应用预处理后的图像格式完全匹配。
    - 数据预处理流程中额外添加`transforms.Grayscale(num_output_channels=1)`，即使输入数据存在异常，也能强制转为单通道，形成双重保障，彻底解决`expected 1 channels, but got 3 channels`的运行时错误。
    
3. **配置参数全同步，确保端到端兼容性**
    
    图像尺寸（28x28）、归一化均值 / 标准差（`0.1307`/`0.3081`）、权重输出路径等关键配置，均与后续 Web 应用保持一致。无需在两个环节之间进行参数转换，训练完成后的权重文件可以直接投入使用，降低后续操作的复杂度。
    
4. **训练后直接可用，无需额外二次处理**
    
    训练完成后生成的`cnn_model_basic.pth`，无需修改任何参数、无需重新封装，只需将其与 Web 应用代码放在同一目录，即可正常加载运行，大大提升开发效率，适合快速落地验证。
    

### 二、完整运行步骤（零基础也能上手）

1. **环境准备：安装必备依赖包**
    
    确保本地环境安装了所需的 Python 库，与后续 Web 应用的环境保持一致，避免出现库版本冲突问题，直接执行以下命令即可：
    
    ```
    pip install torch torchvision numpy
    ```
    
2. **运行训练脚本：自动完成全流程**
    
    直接执行提前编写好的训练 Python 脚本，无需手动干预，脚本会自动完成以下四个核心步骤：
    
    - 下载 MNIST 数据集：如果本地`./data`目录下没有该数据集，会自动从官方源下载并保存，后续训练和评估均基于该标准数据集。
    - 模型训练：默认训练 5 轮模型，训练速度较快，普通 CPU 环境约 5-10 分钟，具备 CUDA 支持的 GPU 环境仅需 1-2 分钟，满足快速验证的需求。
    - 测试集评估：训练完成后，会自动在 MNIST 测试集上验证模型效果，正常情况下准确率可达 98% 以上，具备实用价值。
    - 保存权重文件：最终会在当前目录生成`cnn_model_basic.pth`，这就是我们后续 Web 应用需要的兼容权重文件。
    
3. **验证权重文件：提前规避兼容问题**
    
    将生成的`cnn_model_basic.pth`与后续的 Gradio Web 应用代码放在同一目录下，可提前简单验证文件有效性（无需启动 Web 应用）：检查文件是否存在、大小是否合理（通常为几十 KB），确保没有出现训练中断导致的文件损坏，为后续 Web 应用的正常运行铺路。
    

### 三、关键兼容性解析（解决原错误的核心逻辑）

很多同学在实践中会遇到这样的运行时错误：`RuntimeError: Given groups=1, weight of size [16, 1, 3, 3], expected input[64, 3, 32, 32] to have 1 channels, but got 3 channels instead`，其根源其实只有两个：

1. 模型输入通道不匹配：原微调模型设计为接收 3 通道彩色图，而 Web 应用为了匹配 MNIST 数据集，会将上传图像预处理为 1 通道灰度图，两者输入格式冲突。
2. 模型结构不一致：原微调模型的层结构、维度设计与 Web 应用中定义的`MNIST_CNN`类存在差异，导致权重加载时无法对应。

而我们本次训练代码的解决措施，正是针对性地解决这两个问题：

- 强制锁定单通道输入：卷积层`conv1`的`in_channels=1`，与 Web 应用的图像预处理输出完全匹配，从模型设计层面规避通道数冲突。
- 同步数据预处理流程：训练时的图像预处理与 Web 应用保持一致，均输出 1 通道 28x28 灰度图，从数据源头确保格式统一。
- 复刻模型结构定义：完全照搬 Web 应用中的`MNIST_CNN`类结构，确保权重加载时无层名称、维度的冲突，实现无缝兼容。

### 四、补充优化说明（提升模型效果与实用性）

1. **训练效果优化：追求更高准确率**
    
    如果对模型准确率有更高要求，可将训练轮数`EPOCHS`调整为 10，或适当降低学习率（如从`1e-3`调整为`5e-4`），调整后模型在测试集上的准确率可接近 99%，进一步提升实际识别效果。
    
2. **权重文件迁移：跨环境无缝使用**
    
    生成的`cnn_model_basic.pth`具备良好的可迁移性，可直接复制到任意具备对应环境的机器上使用。若修改了权重文件的保存路径，只需同步更新 Web 应用中的`MODEL_PATH`配置项，即可正常加载，无需其他额外修改。
    
3. **无 GPU 兼容：不影响功能落地**
    
    代码内置了设备自动适配逻辑，即使没有 CUDA 支持的 GPU，也会自动切换至 CPU 运行。虽然训练速度会稍慢，但最终生成的权重文件的兼容性和功能不受任何影响，依然可以正常支撑 Web 应用的运行，满足不同环境的使用需求。
    

## 第二部分：搭建 Gradio Web 识别工具，实现可视化手写数字识别

有了兼容的`cnn_model_basic.pth`权重文件后，我们就可以搭建 Web 识别工具了。借助 Gradio 框架，无需具备任何前端开发经验，即可快速构建一个美观、易用的可视化工具，实现图像上传与实时识别。

### 一、Web 应用代码核心亮点（结构清晰、易于维护）

1. **模块化拆分，职责单一**
    
    将整个 Web 应用的功能拆分为「模型加载」「图像预处理」「预测逻辑」「界面搭建」4 个独立模块，每个函数只负责一项核心功能。这种设计方式便于后续的修改、扩展和排错，比如后续想要优化预测逻辑，只需针对性修改对应函数，无需改动整个代码框架。
    
2. **配置集中管理，降低维护成本**
    
    将设备配置、图像尺寸、归一化参数、权重文件路径等常量，集中定义在代码顶部。后续如果需要调整参数，只需在配置区域进行修改，无需深入业务逻辑代码，大大降低了后续的维护成本，也减少了因参数修改遗漏导致的错误。
    
3. **与训练流程保持一致，确保识别准确率**
    
    Web 应用中的模型结构、图像预处理管道（Resize、Grayscale、Normalize），完全匹配之前的模型训练流程。这是确保识别准确率的关键，避免因预处理方式不一致导致模型无法有效提取特征，保证了从训练到部署的一致性。
    
4. **完善的可视化输出，结果直观易懂**
    
    应用同时提供两种输出形式：一是文本框显示最终的预测数字与置信度，二是标签组件展示所有数字的置信度分布。这种设计不仅能给出明确的识别结果，还能直观展示模型的判断依据，便于用户了解模型的识别可靠性。
    

### 二、Web 应用关键功能说明（实用、便捷、稳定）

1. **模型加载与完善的错误处理**
    
    模型加载环节内置了针对性的错误捕获机制，能够有效应对常见问题：
    
    - 捕获`FileNotFoundError`：当权重文件缺失或路径错误时，返回清晰的提示信息，便于用户排查文件路径问题。
    - 捕获`RuntimeError`：当模型结构与权重文件不匹配时，给出明确的错误提示，避免应用意外崩溃。
    - 自动切换评估模式：加载完成后自动执行`model.eval()`，禁用 Dropout 和 BatchNorm 的训练行为，确保推理结果的稳定性和一致性。
    
2. **标准化的图像预处理流程**
    
    针对用户上传的图像，应用会自动执行一套标准化的预处理流程，确保输入格式符合模型要求：
    
    - 自动调整尺寸：将上传图像调整为 28x28 的标准尺寸，匹配 MNIST 数据集的图像格式。
    - 自动转灰度图：将彩色图像转为单通道灰度图，适配模型的单通道输入要求。
    - 自动标准化处理：消除图像亮度、对比度等因素对识别结果的影响，与训练时的图像预处理保持一致。
    - 自动补充 batch 维度：通过`unsqueeze(0)`补充 batch 维度，适配 PyTorch 模型的输入格式要求。
    
3. **便捷高效的 Gradio Web 界面特性**
    
    搭建的 Web 界面具备多种实用特性，提升用户使用体验：
    
    - 多图像来源支持：支持本地图像上传、剪贴板粘贴图像、摄像头实时拍摄三种方式，满足不同场景的使用需求。
    - 双输出展示：文本框显示最终结果，标签组件展示置信度分布，结果直观易懂。
    - 响应式左右布局：界面分为输入区和输出区，美观整洁，操作便捷，在不同尺寸的设备上都能良好适配。
    - 局域网访问支持：通过`server_name="localhost"`配置，允许同一局域网内的手机、平板等设备访问该工具，提升使用灵活性。
    

### 三、Web 应用运行步骤与注意事项（快速上手）

1. **环境准备：安装额外依赖包**
    
    除了训练环节的依赖包，还需要安装 Gradio 相关依赖，执行以下命令即可：
    
    ```
    pip install torch torchvision gradio pillow numpy
    ```
    
2. **文件准备：放置兼容权重文件**
    
    将之前训练生成的`cnn_model_basic.pth`，放在与 Web 应用代码同一目录下；如果权重文件在其他目录，只需修改代码中的`MODEL_PATH`为权重文件的绝对路径即可。
    
3. **运行代码：启动 Web 应用**
    
    直接执行 Web 应用 Python 脚本，终端会输出相关运行信息，同时会自动打开浏览器进入 Web 界面，典型的终端输出如下：
    
    ```
    ✅ 模型加载成功，已切换至评估模式，运行设备：cpu/cuda
    Running on local URL:  http://localhost:7860
    Running on public URL: https://xxxx.gradio.live （仅share=True时显示）
    ```
    
4. **使用方法：上传图像完成识别**
    
    上传一张白底黑字的手写数字图像（清晰无干扰的图像识别效果更佳），无需手动点击识别按钮，图像上传后会自动触发识别，右侧输出区会实时显示识别结果与置信度分布。

### 四、常见问题排查（避坑指南）

1. **权重加载失败**
    
    核心排查点：检查`MNIST_CNN`类的结构是否与训练时完全一致，重点关注卷积层通道数、全连接层维度等关键参数，否则会出现`key`不匹配错误。
    
2. **识别准确率低**
    
    核心排查点：确保上传的图像清晰无多余干扰，尽量与 MNIST 数据集格式保持一致（白底黑字、单个数字居中），避免因图像质量问题导致模型无法有效提取特征。
    
3. **端口被占用**
    
    核心排查点：如果终端提示端口被占用，可将代码中的`server_port`修改为其他未被占用的端口（如 7861），避免与其他服务发生冲突。
    
4. **GPU 无法使用**
    
    核心排查点：确保已安装 CUDA 版本的 PyTorch，若未安装，应用会自动切换至 CPU 运行，不影响核心功能使用，仅推理速度会稍慢。
    

## 效果展示

![](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20260105165808493.png)


![](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20260105165918679.png)

## 总结

本次我们完整实现了从「MNIST 模型从头训练」到「Gradio Web 应用搭建」的全流程，核心成果有两点：

1. 生成了`cnn_model_basic.pth`兼容权重文件，彻底解决了通道数不匹配、模型结构冲突等常见问题，具备良好的可迁移性和实用性。
2. 搭建了无需前端经验的 Web 识别工具，实现了「模型加载→图像预处理→Web 界面交互→结果可视化」的完整流程，结构清晰、易于维护，满足快速落地使用的需求。

整个流程避开了常见的坑，所有环节保持高度一致性，无论是零基础的同学，还是有一定 PyTorch 实践经验的开发者，都可以快速上手并实现落地。借助这个工具，我们可以轻松完成手写数字的可视化识别，也可以在此基础上进行进一步的扩展和优化，比如支持多数字识别、优化图像预处理逻辑等。

## 附录


### 完整训练代码（生成兼容的`cnn_model_basic.pth`）

```python
# 导入所需依赖库
import torch
import torch.nn as nn
import torch.optim as optim
import torchvision
import torchvision.transforms as transforms
from torch.utils.data import DataLoader

# ====================== 1. 定义常量配置（与Web应用保持一致，确保兼容性） ======================
# 设备配置：优先GPU，无则CPU
DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")
# 训练参数
BATCH_SIZE = 64
EPOCHS = 5
LEARNING_RATE = 1e-3
# 输出权重文件路径（与Web应用中MODEL_PATH一致）
OUTPUT_MODEL_PATH = "cnn_model_basic.pth"
# MNIST图像参数（与Web应用完全匹配）
MNIST_IMAGE_SIZE = (28, 28)
NORMALIZE_MEAN = (0.1307,)
NORMALIZE_STD = (0.3081,)

# ====================== 2. 定义CNN模型结构（与Web应用完全一致，核心兼容性保障） ======================
# 该模型结构与Gradio Web应用中的MNIST_CNN类完全相同，确保权重加载无冲突
class MNIST_CNN(nn.Module):
    def __init__(self):
        super(MNIST_CNN, self).__init__()
        # 卷积层：输入1通道（灰度图），输出16通道，卷积核3x3，padding=1保持尺寸
        self.conv1 = nn.Conv2d(1, 16, kernel_size=3, padding=1)
        self.relu1 = nn.ReLU()
        self.pool1 = nn.MaxPool2d(2, 2)  # 池化后尺寸：14x14
        
        self.conv2 = nn.Conv2d(16, 32, kernel_size=3, padding=1)
        self.relu2 = nn.ReLU()
        self.pool2 = nn.MaxPool2d(2, 2)  # 池化后尺寸：7x7
        
        # 全连接层：32*7*7=1568个特征 → 10个分类（0-9）
        self.fc1 = nn.Linear(32 * 7 * 7, 128)
        self.relu3 = nn.ReLU()
        self.fc2 = nn.Linear(128, 10)

    def forward(self, x):
        # 前向传播流程（与Web应用模型完全一致）
        x = self.pool1(self.relu1(self.conv1(x)))
        x = self.pool2(self.relu2(self.conv2(x)))
        x = x.view(-1, 32 * 7 * 7)  # 展平特征图，维度匹配
        x = self.relu3(self.fc1(x))
        x = self.fc2(x)
        return x

# ====================== 3. 数据加载与预处理（与Web应用保持一致，避免输入格式不兼容） ======================
# 预处理管道与Web应用完全相同：灰度图→28x28→Tensor→标准化
transform = transforms.Compose([
    transforms.Resize(MNIST_IMAGE_SIZE),
    transforms.Grayscale(num_output_channels=1),  # 强制转为1通道（MNIST原生已为1通道，双重保障）
    transforms.ToTensor(),
    transforms.Normalize(NORMALIZE_MEAN, NORMALIZE_STD)
])

# 加载MNIST训练集和测试集
train_dataset = torchvision.datasets.MNIST(
    root='./data',
    train=True,
    download=True,
    transform=transform
)

test_dataset = torchvision.datasets.MNIST(
    root='./data',
    train=False,
    download=True,
    transform=transform
)

# 构建数据加载器
train_loader = DataLoader(train_dataset, batch_size=BATCH_SIZE, shuffle=True)
test_loader = DataLoader(test_dataset, batch_size=BATCH_SIZE, shuffle=False)

# ====================== 4. 模型初始化、损失函数与优化器 ======================
# 初始化模型并移至指定设备
model = MNIST_CNN().to(DEVICE)

# 分类任务损失函数（交叉熵损失）
criterion = nn.CrossEntropyLoss()

# 优化器（Adam优化，学习率适中）
optimizer = optim.Adam(model.parameters(), lr=LEARNING_RATE)

# ====================== 5. 模型训练循环 ======================
print(f"开始训练模型，运行设备：{DEVICE}，训练轮数：{EPOCHS}，批次大小：{BATCH_SIZE}")
for epoch in range(EPOCHS):
    # 切换为训练模式（启用BatchNorm/Dropout训练行为）
    model.train()
    running_loss = 0.0
    
    for i, (images, labels) in enumerate(train_loader):
        # 将数据移至指定设备
        images = images.to(DEVICE)
        labels = labels.to(DEVICE)
        
        # 1. 清空上一轮梯度
        optimizer.zero_grad()
        
        # 2. 前向传播
        outputs = model(images)
        
        # 3. 计算损失
        loss = criterion(outputs, labels)
        
        # 4. 反向传播
        loss.backward()
        
        # 5. 更新模型参数
        optimizer.step()
        
        # 统计损失值
        running_loss += loss.item()
        
        # 每100个批次打印一次训练状态
        if (i + 1) % 100 == 0:
            avg_loss = running_loss / 100
            print(f"Epoch [{epoch+1}/{EPOCHS}], Step [{i+1}/{len(train_loader)}], Average Loss: {avg_loss:.4f}")
            running_loss = 0.0

# ====================== 6. 训练后模型评估（验证模型效果） ======================
print("\n训练完成，开始评估模型在测试集上的表现...")
model.eval()  # 切换为评估模式
correct = 0
total = 0

# 推理时禁用梯度计算，节省内存
with torch.no_grad():
    for images, labels in test_loader:
        images = images.to(DEVICE)
        labels = labels.to(DEVICE)
        
        outputs = model(images)
        _, predicted = torch.max(outputs.data, 1)
        
        total += labels.size(0)
        correct += (predicted == labels).sum().item()

# 输出测试集准确率
test_accuracy = 100 * correct / total
print(f"测试集准确率: {test_accuracy:.2f}%")

# ====================== 7. 保存训练好的权重文件（与Web应用兼容） ======================
torch.save(model.state_dict(), OUTPUT_MODEL_PATH)
print(f"\n✅ 权重文件已成功保存至：{OUTPUT_MODEL_PATH}")
print(f"该文件可直接用于之前的Gradio Web手写数字识别应用，无兼容性冲突")
```

### Gradio实现Web页面完整代码

```python
# 导入所需库
import torch
import torch.nn as nn
import torchvision.transforms as transforms
import gradio as gr
from PIL import Image
import numpy as np

# ====================== 1. 定义常量与固定配置（易于维护） ======================
# 设备配置：优先GPU，无则CPU
DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")
# 模型权重文件路径
MODEL_PATH = "cnn_model_basic.pth"
# MNIST图像标准尺寸（28x28灰度图）
MNIST_IMAGE_SIZE = (28, 28)
# 图像归一化参数（与训练/微调时保持一致）
NORMALIZE_MEAN = (0.1307,)
NORMALIZE_STD = (0.3081,)


# ====================== 2. 定义与预训练模型匹配的CNN结构（必须与微调时一致） ======================
class MNIST_CNN(nn.Module):
    def __init__(self):
        super(MNIST_CNN, self).__init__()
        # 卷积层：提取图像通用特征
        self.conv1 = nn.Conv2d(1, 16, kernel_size=3, padding=1)
        self.relu1 = nn.ReLU()
        self.pool1 = nn.MaxPool2d(2, 2)

        self.conv2 = nn.Conv2d(16, 32, kernel_size=3, padding=1)
        self.relu2 = nn.ReLU()
        self.pool2 = nn.MaxPool2d(2, 2)

        # 全连接层：完成分类任务
        self.fc1 = nn.Linear(32 * 7 * 7, 128)
        self.relu3 = nn.ReLU()
        self.fc2 = nn.Linear(128, 10)

    def forward(self, x):
        # 前向传播流程（与微调时一致）
        x = self.pool1(self.relu1(self.conv1(x)))
        x = self.pool2(self.relu2(self.conv2(x)))
        x = x.view(-1, 32 * 7 * 7)  # 展平特征图
        x = self.relu3(self.fc1(x))
        x = self.fc2(x)
        return x


# ====================== 3. 初始化模型并加载权重（包含错误处理） ======================
def load_mnist_model(model_path, device):
    """
    加载MNIST手写数字识别模型
    :param model_path: 权重文件路径
    :param device: 运行设备
    :return: 初始化完成的模型
    """
    try:
        # 1. 初始化模型结构
        model = MNIST_CNN().to(device)
        # 2. 加载权重文件
        model.load_state_dict(torch.load(model_path, map_location=device))
        # 3. 切换为评估模式（禁用Dropout/BatchNorm训练行为）
        model.eval()
        print(f"✅ 模型加载成功，已切换至评估模式，运行设备：{device}")
        return model
    except FileNotFoundError:
        raise Exception(f"❌ 权重文件未找到，请检查路径：{model_path}")
    except RuntimeError as e:
        raise Exception(f"❌ 模型结构与权重不匹配，加载失败：{str(e)}")
    except Exception as e:
        raise Exception(f"❌ 模型加载未知错误：{str(e)}")


# ====================== 4. 图像预处理与预测函数（核心业务逻辑） ======================
# 定义图像预处理管道（与训练/微调时保持一致，确保输入格式匹配）
image_transform = transforms.Compose([
    transforms.Resize(MNIST_IMAGE_SIZE),  # 调整为28x28标准尺寸
    transforms.Grayscale(num_output_channels=1),  # 转为单通道灰度图
    transforms.ToTensor(),  # 转为Tensor并归一化至[0,1]
    transforms.Normalize(NORMALIZE_MEAN, NORMALIZE_STD)  # 标准化（与训练一致）
])


def predict_handwritten_digit(image, model):
    """
    处理上传图像并预测手写数字
    :param image: Gradio上传的图像（PIL.Image格式）
    :param model: 加载完成的预测模型
    :return: 预测结果（字典格式，包含数字与置信度）
    """
    try:
        # 步骤1：图像预处理
        if image is None:
            raise ValueError("请上传一张手写数字图像")

        # 应用预处理管道
        processed_image = image_transform(image).unsqueeze(0)  # 增加batch维度（[1,1,28,28]）
        processed_image = processed_image.to(DEVICE)

        # 步骤2：模型推理（禁用梯度计算，节省内存）
        with torch.no_grad():
            outputs = model(processed_image)
            # 计算各类别置信度（softmax转换为概率分布）
            probabilities = torch.softmax(outputs, dim=1).squeeze(0).cpu().numpy()

        # 步骤3：整理预测结果
        pred_digit = np.argmax(probabilities)  # 取置信度最高的数字
        pred_confidence = round(probabilities[pred_digit] * 100, 2)  # 保留2位小数

        # 构建完整结果（返回所有数字的置信度，便于可视化）
        result = {str(i): round(probabilities[i] * 100, 2) for i in range(10)}
        print(f"🔍 预测完成：数字{pred_digit}，置信度{pred_confidence}%")

        return f"预测结果：{pred_digit}（置信度：{pred_confidence}%）", result

    except ValueError as e:
        return f"⚠️ 输入错误：{str(e)}", {str(i): 0.0 for i in range(10)}
    except Exception as e:
        return f"⚠️ 预测失败：{str(e)}", {str(i): 0.0 for i in range(10)}


# ====================== 5. Gradio Web界面搭建与运行 ======================
def main():
    # 步骤1：加载模型（捕获加载错误，避免程序崩溃）
    try:
        model = load_mnist_model(MODEL_PATH, DEVICE)
    except Exception as e:
        print(f"模型初始化失败：{e}")
        return

    # 步骤2：定义Gradio界面组件
    with gr.Blocks(title="MNIST手写数字识别工具") as demo:
        # 页面标题
        gr.Markdown("# 🖋️ MNIST 手写数字识别工具")
        gr.Markdown("### 上传一张包含单个手写数字（0-9）的图像，模型将自动识别并返回结果")

        # 布局：分为输入区、输出区
        with gr.Row():
            # 左侧：输入区（图像上传）
            with gr.Column(scale=1):
                image_input = gr.Image(
                    type="pil",  # 输出PIL.Image格式，便于预处理
                    label="上传手写数字图像",
                    height=300,
                    sources=["upload", "clipboard", "webcam"]  # 支持上传、剪贴板、摄像头
                )
                submit_btn = gr.Button("开始识别", variant="primary", size="lg")

            # 右侧：输出区（预测结果）
            with gr.Column(scale=1):
                text_output = gr.Textbox(
                    label="识别结果",
                    placeholder="识别结果将显示在这里...",
                    lines=2,
                    interactive=False
                )
                confidence_output = gr.Label(
                    label="各数字置信度分布",
                    num_top_classes=5  # 显示置信度前5的数字
                )

        # 步骤3：绑定事件（按钮点击/图像上传自动触发识别）
        def run_prediction(image):
            return predict_handwritten_digit(image, model)

        # 绑定触发方式
        submit_btn.click(fn=run_prediction, inputs=image_input, outputs=[text_output, confidence_output])
        image_input.change(fn=run_prediction, inputs=image_input, outputs=[text_output, confidence_output])

        # 步骤4：添加说明文字
        gr.Markdown("### 📌 注意事项")
        gr.Markdown("1. 建议上传白底黑字的手写数字图像，识别效果更佳")
        gr.Markdown("2. 图像将自动转为28x28灰度图，与MNIST数据集格式保持一致")
        gr.Markdown("3. 模型基于CNN微调训练，仅支持单个数字识别")

    # 步骤5：启动Web服务
    demo.launch(
        server_name="localhost",  # 允许局域网访问（可选）
        server_port=7860,  # 自定义端口（可选）
        share=False  # 如需公网临时链接，设置为True（需网络通畅）
    )


# ====================== 6. 程序入口 ======================
if __name__ == "__main__":
    main()
```









