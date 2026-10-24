---
title: 自定义数据集实现猫狗分类
description: 本文介绍自定义数据集实现猫狗分类的过程。
date: 2026-01-08T12:34:25+08:00
draft: false
categories:
- 编程
- AI
tags:
- Classification
---


# 自定义数据集实现猫狗分类


## 前言

写作本文主要是为了学会使用本地数据集，而不是直接下载数据集。

本文代码还使用了`torchsummary`、`tqdm`等工具。

**注意文件名！**

**注意文件名！**

**注意文件名！**
## 源代码

### train.py

```python
import os
import torch
import torch.nn as nn
import torch.optim as optim
import torchvision.transforms as transforms
from torchvision.datasets import ImageFolder
from torch.utils.data import DataLoader
import numpy as np
from tqdm import tqdm

# 训练集和测试集路径
train_path = './Dataset/train'
test_path = './Dataset/test'

# 获取训练集和测试集集的标签
train_labels = sorted(os.listdir(train_path))
test_labels = sorted(os.listdir(test_path))
if train_labels == test_labels:
    animal_labels = train_labels = test_labels
print("animal_labels: ", animal_labels)

# 数据增强
train_transform = transforms.Compose([
    transforms.RandomRotation(10),
    transforms.RandomHorizontalFlip(),
    transforms.RandomResizedCrop(148, scale=(0.8, 1.0)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.4, 0.4, 0.4], std=[0.2, 0.2, 0.2])
])

test_transform = transforms.Compose([
    transforms.Resize((148, 148)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.4, 0.4, 0.4], std=[0.2, 0.2, 0.2])
])

# 加载数据集
train_dataset = ImageFolder(train_path, transform=train_transform)
test_dataset = ImageFolder(test_path, transform=test_transform)

batch_size = 64

train_loader = DataLoader(train_dataset, batch_size=batch_size, shuffle=True)
test_loader = DataLoader(test_dataset, batch_size=batch_size, shuffle=False)


# 定义模型
class AnimalCNN(nn.Module):
    def __init__(self, num_classes):
        super(AnimalCNN, self).__init__()
        self.conv1 = nn.Conv2d(3, 32, kernel_size=3, padding=1)
        self.conv2 = nn.Conv2d(32, 64, kernel_size=3, padding=1)
        self.conv3 = nn.Conv2d(64, 128, kernel_size=3, padding=1)
        self.conv4 = nn.Conv2d(128, 256, kernel_size=3, padding=1)
        self.pool = nn.MaxPool2d(kernel_size=2, stride=2)
        self.bn1 = nn.BatchNorm2d(32)
        self.bn2 = nn.BatchNorm2d(64)
        self.bn3 = nn.BatchNorm2d(128)
        self.bn4 = nn.BatchNorm2d(256)
        self.fc1 = nn.Linear(256 * 9 * 9, 512)
        self.fc2 = nn.Linear(512, 64)
        self.fc3 = nn.Linear(64, num_classes)
        self.relu = nn.ReLU()

    def forward(self, x):
        x = self.relu(self.bn1(self.pool(self.conv1(x))))
        x = self.relu(self.bn2(self.pool(self.conv2(x))))
        x = self.relu(self.bn3(self.pool(self.conv3(x))))
        x = self.relu(self.bn4(self.pool(self.conv4(x))))
        x = x.view(x.size(0), -1)
        x = self.relu(self.fc1(x))
        x = self.relu(self.fc2(x))
        x = self.fc3(x)
        return x


device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = AnimalCNN(num_classes=len(animal_labels)).to(device)

# 定义损失函数和优化器
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# 训练模型
num_epochs = 25
train_losses = []
train_accuracies = []
test_losses = []
test_accuracies = []

for epoch in range(num_epochs):
    model.train()
    running_loss = 0.0
    correct = 0
    total = 0
    with tqdm(total=len(train_loader), desc=f'Epoch {epoch + 1}/{num_epochs}', unit='batch') as pbar:
        for images, labels in train_loader:
            images, labels = images.to(device), labels.to(device)
            optimizer.zero_grad()
            outputs = model(images)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()

            running_loss += loss.item()
            _, predicted = torch.max(outputs.data, 1)
            total += labels.size(0)
            correct += (predicted == labels).sum().item()

            pbar.set_postfix({
                'Train Loss': running_loss / (pbar.n + 1),
                'Train Acc': correct / total
            })
            pbar.update(1)

    train_loss = running_loss / len(train_loader)
    train_acc = correct / total
    train_losses.append(train_loss)
    train_accuracies.append(train_acc)

    model.eval()
    test_loss = 0.0
    correct = 0
    total = 0
    all_labels = []
    all_preds = []
    with tqdm(total=len(test_loader), desc='Testing', unit='batch') as pbar:
        with torch.no_grad():
            for images, labels in test_loader:
                images, labels = images.to(device), labels.to(device)
                outputs = model(images)
                loss = criterion(outputs, labels)
                test_loss += loss.item()
                _, predicted = torch.max(outputs.data, 1)
                total += labels.size(0)
                correct += (predicted == labels).sum().item()
                all_labels.extend(labels.cpu().numpy())
                all_preds.extend(predicted.cpu().numpy())

                pbar.set_postfix({
                    'Test Loss': test_loss / (pbar.n + 1),
                    'Test Acc': correct / total
                })
                pbar.update(1)

    test_loss = test_loss / len(test_loader)
    test_acc = correct / total
    test_losses.append(test_loss)
    test_accuracies.append(test_acc)

    print(
        f'Epoch {epoch + 1}/{num_epochs}, Train Loss: {train_loss:.4f}, Train Acc: {train_acc:.4f}, Test Loss: {test_loss:.4f}, Test Acc: {test_acc:.4f}')

    # 保存最佳模型
    if epoch == 0 or test_acc > max(test_accuracies[:-1]):
        # torch.save(model.state_dict(), './weights/best_model.pth')
        torch.save(model.state_dict(), './weights/new_best_model.pth')

# 训练结束后，将结果写入文件
os.makedirs('./training_results', exist_ok=True)
# training_results = './training_results/training_results.txt'
training_results = './training_results/new_training_results.txt'
with open(training_results, 'w') as f:
    f.write('epoch,train_losses,train_accuracies,test_losses,test_accuracies\n')
    for i in range(len(train_losses)):
        f.write(f'{i + 1},{train_losses[i]},{train_accuracies[i]},{test_losses[i]},{test_accuracies[i]}\n')

# 保存最终的模型
# torch.save(model.state_dict(), './weights/final_model.pth')
torch.save(model.state_dict(), './weights/new_final_model.pth')

```

一、整体功能

这段代码主要实现了一个基于卷积神经网络（CNN）的猫狗分类模型的训练过程，包括数据准备、模型定义、训练循环、模型评估以及训练结果的保存等操作，旨在通过训练得到一个能够有效区分猫狗图像的模型，并记录相关训练指标以便后续分析。

二、具体步骤

数据准备：

首先指定了训练集和测试集的路径，然后通过读取对应路径下的文件列表获取训练集和测试集的标签，并确保两者标签一致后将其存储为 animal_labels。

定义了数据增强的转换操作，对于训练集，包含随机旋转、水平翻转、随机裁剪等操作后再转换为张量并进行归一化；对于测试集，主要进行尺寸调整后转换为张量并归一化。

使用 ImageFolder 类结合相应的转换操作加载训练集和测试集数据，并通过 DataLoader 创建数据加载器，设置批次大小为 64，训练集数据加载时打乱顺序，测试集则不打乱。

模型定义：

定义了 AnimalCNN 类作为猫狗分类的卷积神经网络模型，该模型包含多个卷积层、池化层、批量归一化层以及全连接层。通过 forward 方法详细描述了数据在模型中的前向传播过程，即数据依次经过各层的处理最终得到分类结果。

将定义好的模型移动到指定设备（GPU 或 CPU，根据是否有可用的 GPU 来确定）上运行。

训练设置：

定义了损失函数为交叉熵损失（Cross Entropy Loss），优化器为 Adam 优化器，并设置了学习率为 0.001。

创建了几个空列表，用于存储训练过程中每个 epoch 的训练损失、训练准确率、测试损失和测试准确率等指标。

训练循环：

通过循环进行多个 epoch 的训练，在每个 epoch 中：

首先将模型设置为训练模式，在训练数据加载器上进行迭代。对于每一批次的数据，将图像和标签数据移动到指定设备上，先清空优化器的梯度，然后将数据传入模型得到输出，计算损失并进行反向传播更新模型参数，同时累计该批次的损失和计算预测正确的样本数量，通过 tqdm 进度条实时显示当前 epoch 的训练损失和准确率信息。

完成一轮训练数据的迭代后，计算该 epoch 的训练损失和训练准确率，并将其分别添加到对应的列表中。

接着将模型设置为评估模式，在测试数据加载器上进行类似的迭代操作，但不进行梯度更新，用于计算该 epoch 的测试损失和测试准确率，并添加到相应列表中，同时通过 tqdm 进度条实时显示测试损失和准确率信息。

在每个 epoch 结束后，打印出当前 epoch 的训练损失、训练准确率、测试损失和测试准确率等详细信息。

根据测试准确率判断是否保存当前模型为最佳模型，如果是第一个 epoch 或者当前 epoch 的测试准确率高于之前所有 epoch 的测试准确率（除当前 epoch 外），则将模型的状态字典保存到指定的权重文件（这里保存为 new_best_model.pth）中。

训练结果保存：

创建 training_results 文件夹（如果不存在），指定一个新的训练结果记录文件路径（new_training_results.txt）。

将每个 epoch 的训练损失、训练准确率、测试损失和测试准确率等信息按照规定格式写入到该文件中，以便后续查看和分析训练过程的变化情况。

最后，将训练结束后的最终模型状态字典保存到指定的权重文件（new_final_model.pth）中。
### summary.py

```python
import os
import matplotlib.pyplot as plt
import numpy as np
from tqdm import tqdm
import torch
import torch.nn as nn
import torchvision.transforms as transforms
from torchvision.datasets import ImageFolder
from torch.utils.data import DataLoader
from torchsummary import summary
from sklearn.metrics import classification_report

from sklearn.metrics import confusion_matrix
import seaborn as sns


# 定义设备
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')


# 定义模型
class AnimalCNN(nn.Module):
    def __init__(self, num_classes):
        super(AnimalCNN, self).__init__()
        self.conv1 = nn.Conv2d(3, 32, kernel_size=3, padding=1)
        self.conv2 = nn.Conv2d(32, 64, kernel_size=3, padding=1)
        self.conv3 = nn.Conv2d(64, 128, kernel_size=3, padding=1)
        self.conv4 = nn.Conv2d(128, 256, kernel_size=3, padding=1)
        self.pool = nn.MaxPool2d(kernel_size=2, stride=2)
        self.bn1 = nn.BatchNorm2d(32)
        self.bn2 = nn.BatchNorm2d(64)
        self.bn3 = nn.BatchNorm2d(128)
        self.bn4 = nn.BatchNorm2d(256)
        self.fc1 = nn.Linear(256 * 9 * 9, 512)
        self.fc2 = nn.Linear(512, 64)
        self.fc3 = nn.Linear(64, num_classes)
        self.relu = nn.ReLU()

    def forward(self, x):
        x = self.relu(self.bn1(self.pool(self.conv1(x))))
        x = self.relu(self.bn2(self.pool(self.conv2(x))))
        x = self.relu(self.bn3(self.pool(self.conv3(x))))
        x = self.relu(self.bn4(self.pool(self.conv4(x))))
        x = x.view(x.size(0), -1)
        x = self.relu(self.fc1(x))
        x = self.relu(self.fc2(x))
        x = self.fc3(x)
        return x


# 训练集和测试集路径
train_path = './dataset/train'
test_path = './dataset/test'

# 获取训练集和测试集集的标签
train_labels = sorted(os.listdir(train_path))
test_labels = sorted(os.listdir(test_path))
if train_labels == test_labels:
    animal_labels = train_labels = test_labels
print("animal_labels: ", animal_labels)


def plot_training_results(filename, save_path=None):
    with open(filename, 'r') as f:
        lines = f.readlines()[1:]
        data = [line.strip().split(',') for line in lines]
        epochs = np.array([int(row[0]) for row in data])
        train_losses = np.array([float(row[1]) for row in data])
        train_accuracies = np.array([float(row[2]) for row in data])
        test_losses = np.array([float(row[3]) for row in data])
        test_accuracies = np.array([float(row[4]) for row in data])

    plt.figure(figsize=(12, 6))
    plt.subplot(1, 2, 1)
    plt.plot(epochs, train_losses, label='Training Loss', color='blue')
    plt.plot(epochs, test_losses, label='Testing Loss', color='orange')
    plt.title('Training and Testing Loss')
    plt.xlabel('Epochs')
    plt.ylabel('Loss')
    plt.legend()

    plt.subplot(1, 2, 2)
    plt.plot(epochs, train_accuracies * 100, label='Training Accuracy', color='blue')
    plt.plot(epochs, test_accuracies * 100, label='Testing Accuracy', color='orange')
    plt.title('Training and Testing Accuracy')
    plt.xlabel('Epochs')
    plt.ylabel('Accuracy (%)')
    plt.legend()

    if save_path:
        plt.savefig(save_path, bbox_inches='tight')

    plt.tight_layout()
    plt.show()


# 绘制训练过程中的损失和准确率曲线图
training_results = './training_results/training_results.txt'
save_path = './Image/LossAndAccuracy.png'
plot_training_results(training_results, save_path)


# 评估模型
def evaluate(loader):
    model.eval()
    correct = 0
    total = 0
    loss = 0.0
    all_labels = []
    all_preds = []
    with torch.no_grad():
        with tqdm(total=len(loader), desc='Evaluating', unit='batch') as pbar:
            for images, labels in loader:
                images, labels = images.to(device), labels.to(device)
                outputs = model(images)
                loss += criterion(outputs, labels).item()
                _, predicted = torch.max(outputs.data, 1)
                total += labels.size(0)
                correct += (predicted == labels).sum().item()
                all_labels.extend(labels.cpu().numpy())
                all_preds.extend(predicted.cpu().numpy())
                pbar.update(1)
    accuracy = correct / total
    return loss / len(loader), accuracy, all_labels, all_preds


# 加载模型
model_path = './weights/best_model.pth'
model = AnimalCNN(num_classes=len(animal_labels)).to(device)

summary(model, (3, 148, 148))

# model.load_state_dict(torch.load(model_path))
model.load_state_dict(torch.load(model_path, map_location=torch.device('cpu'), weights_only=False))
criterion = nn.CrossEntropyLoss()

# 数据增强
train_transform = transforms.Compose([
    transforms.RandomRotation(10),
    transforms.RandomHorizontalFlip(),
    transforms.RandomResizedCrop(148, scale=(0.8, 1.0)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.4, 0.4, 0.4], std=[0.2, 0.2, 0.2])
])

test_transform = transforms.Compose([
    transforms.Resize((148, 148)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.4, 0.4, 0.4], std=[0.2, 0.2, 0.2])
])

train_dataset = ImageFolder(train_path, transform=train_transform)
test_dataset = ImageFolder(test_path, transform=test_transform)

batch_size = 64
train_loader = DataLoader(train_dataset, batch_size=batch_size, shuffle=True)
test_loader = DataLoader(test_dataset, batch_size=batch_size, shuffle=False)

# 评估模型
train_loss, train_acc, train_labels, train_preds = evaluate(train_loader)
test_loss, test_acc, test_labels, test_preds = evaluate(test_loader)
print('Train loss: ', train_loss)
print('Train accuracy: ', train_acc)
print('Test loss: ', test_loss)
print('Test accuracy: ', test_acc)

# 输出分类报告表
train_report = classification_report(train_labels, train_preds, target_names=animal_labels)
test_report = classification_report(test_labels, test_preds, target_names=animal_labels)
print('Train Classification Report:\n', train_report)
print('Test Classification Report:\n', test_report)

# 保存分类报告表
with open('./training_results/classification_report.txt', 'w') as f:
    f.write('Train Classification Report:\n')
    f.write(train_report)
    f.write('\nTest Classification Report:\n')
    f.write(test_report)

test_loader = DataLoader(test_dataset, batch_size=batch_size, shuffle=True)


def display_predictions(model, data_loader, animal_labels, num_images=10, images_per_row=5):
    model.eval()
    images, labels, predictions = [], [], []
    with torch.no_grad():
        for i, (img_batch, lbl_batch) in enumerate(data_loader):
            if len(images) >= num_images:
                break
            img_batch, lbl_batch = img_batch.to(device), lbl_batch.to(device)
            outputs = model(img_batch)
            _, predicted = torch.max(outputs, 1)
            images.extend(img_batch.cpu())
            labels.extend(lbl_batch.cpu())
            predictions.extend(predicted.cpu())

    fig, axes = plt.subplots(nrows=num_images // images_per_row, ncols=images_per_row, figsize=(10, 4))
    axes = axes.flatten()

    for i in range(num_images):
        img = images[i].permute(1, 2, 0).numpy()  # 转置图像数据
        true_label = labels[i].item()
        pred_label = predictions[i].item()

        true_animal = animal_labels[true_label]
        pred_animal = animal_labels[pred_label]

        axes[i].imshow(img)
        axes[i].axis('off')

        color = 'green' if true_label == pred_label else 'red'
        axes[i].set_title(f'True: {true_animal}\nPred: {pred_animal}', color=color, fontsize=8)

    plt.tight_layout()
    plt.subplots_adjust(hspace=0.5, wspace=0.1)
    plt.show()


# 显示训练集的预测结果(随机抽取10张图片)
display_predictions(model, train_loader, animal_labels)

# 显示测试集的预测结果(随机抽取10张图片)
display_predictions(model, test_loader, animal_labels)




def plot_confusion_matrix(true_labels, pred_labels, animal_labels, save_path=None):
    cm = confusion_matrix(true_labels, pred_labels)
    plt.figure(figsize=(10, 8))
    sns.heatmap(cm, annot=True, fmt='d', cmap='Greens', xticklabels=animal_labels, yticklabels=animal_labels)
    plt.xlabel('Predicted Labels')
    plt.ylabel('True Labels')
    plt.title('Confusion Matrix')

    if save_path:
        plt.savefig(save_path, bbox_inches='tight')

    plt.show()


# 计算并绘制训练集的混淆矩阵
plot_confusion_matrix(train_labels, train_preds, animal_labels, save_path='./Image/train_confusion_matrix.png')

# 计算并绘制测试集的混淆矩阵
plot_confusion_matrix(test_labels, test_preds, animal_labels, save_path='./Image/test_confusion_matrix.png')


```

一、整体功能  

这段代码主要围绕猫狗分类模型展开，涉及模型定义、数据加载与预处理、模型评估、预测结果展示以及相关训练指标和分类报告的可视化等操作，旨在全面评估模型在猫狗图像分类任务上的性能。  

二、具体步骤  

环境设置与模型定义：  

导入一系列必要的库，包括用于文件操作、图像处理、深度学习框架相关、数据加载与处理、可视化以及模型评估指标计算等方面的库。  

定义了设备（GPU 或 CPU），根据是否有可用的 GPU 来确定模型运行的设备。  

定义了 AnimalCNN 类作为猫狗分类模型，该模型包含多个卷积层、池化层、批量归一化层以及全连接层，通过 forward 方法定义了数据的前向传播路径。  

数据准备：  

指定训练集和测试集的路径，获取训练集和测试集的标签，并确保两者标签一致后存储为animal_labels。  

定义了数据增强的转换操作，分别针对训练集和测试集。训练集的转换包括随机旋转、水平翻转、随机裁剪等操作后再进行归一化；测试集则主要进行尺寸调整和归一化。  

使用 ImageFolder 结合相应的转换操作加载训练集和测试集数据，并创建 DataLoader 对象，以便按批次加载数据，设置了批次大小为 64，训练集数据加载时打乱顺序，测试集则不打乱。  

模型加载与评估：  

加载预训练的模型权重文件（best_model.pth），并将模型移动到指定设备上，同时展示模型的结构摘要信息。  

定义了交叉熵损失函数（Cross Entropy Loss）作为模型的损失函数。  

实现了 evaluate 函数用于评估模型在给定数据加载器上的性能，在评估过程中，模型设置为评估模式，计算损失、准确率以及收集真实标签和预测标签，最终返回平均损失、准确率以及相关标签列表。  

分别使用训练集和测试集的数据加载器对模型进行评估，输出训练集和测试集的损失、准确率等信息，并生成详细的分类报告，包括精确率、召回率、F1 值等指标，同时将分类报告保存到文件中。  
预测结果展示与可视化：  

定义了 display_predictions 函数用于展示模型对训练集和测试集数据的预测结果，随机抽取指定数量（默认为 10 张）的图片，展示其真实标签和预测标签，并通过不同颜色区分预测正确与否。  

定义了 plot_confusion_matrix 函数用于计算并绘制训练集和测试集的混淆矩阵，通过热力图直观展示模型在不同类别上的预测情况，将混淆矩阵图保存到指定路径下。  

调用上述函数分别展示训练集和测试集的预测结果以及绘制相应的混淆矩阵。  

训练指标可视化：  

实现了 plot_training_results 函数用于读取训练过程中的损失和准确率记录文件，将其绘制成曲线图，展示训练集和测试集的损失、准确率随训练轮次的变化情况，可将曲线图保存到指定路径下。 

调用该函数绘制并展示训练过程中的损失和准确率曲线图。

输出结果如下所示：

```
animal_labels:  ['cat', 'dog']
----------------------------------------------------------------
        Layer (type)               Output Shape         Param #
================================================================
            Conv2d-1         [-1, 32, 148, 148]             896
         MaxPool2d-2           [-1, 32, 74, 74]               0
       BatchNorm2d-3           [-1, 32, 74, 74]              64
              ReLU-4           [-1, 32, 74, 74]               0
            Conv2d-5           [-1, 64, 74, 74]          18,496
         MaxPool2d-6           [-1, 64, 37, 37]               0
       BatchNorm2d-7           [-1, 64, 37, 37]             128
              ReLU-8           [-1, 64, 37, 37]               0
            Conv2d-9          [-1, 128, 37, 37]          73,856
        MaxPool2d-10          [-1, 128, 18, 18]               0
      BatchNorm2d-11          [-1, 128, 18, 18]             256
             ReLU-12          [-1, 128, 18, 18]               0
           Conv2d-13          [-1, 256, 18, 18]         295,168
        MaxPool2d-14            [-1, 256, 9, 9]               0
      BatchNorm2d-15            [-1, 256, 9, 9]             512
             ReLU-16            [-1, 256, 9, 9]               0
           Linear-17                  [-1, 512]      10,617,344
             ReLU-18                  [-1, 512]               0
           Linear-19                   [-1, 64]          32,832
             ReLU-20                   [-1, 64]               0
           Linear-21                    [-1, 2]             130
================================================================
Total params: 11,039,682
Trainable params: 11,039,682
Non-trainable params: 0
----------------------------------------------------------------
Input size (MB): 0.25
Forward/backward pass size (MB): 17.44
Params size (MB): 42.11
Estimated Total Size (MB): 59.80
----------------------------------------------------------------
Evaluating: 100%|██████████| 313/313 [01:10<00:00,  4.44batch/s]
Evaluating: 100%|██████████| 79/79 [00:14<00:00,  5.53batch/s]
Train loss:  0.04246797274869566
Train accuracy:  0.984
Test loss:  0.17575686408466176
Test accuracy:  0.9494
Train Classification Report:
               precision    recall  f1-score   support

         cat       0.98      0.99      0.98     10000
         dog       0.99      0.98      0.98     10000

    accuracy                           0.98     20000
   macro avg       0.98      0.98      0.98     20000
weighted avg       0.98      0.98      0.98     20000

Test Classification Report:
               precision    recall  f1-score   support

         cat       0.94      0.96      0.95      2500
         dog       0.96      0.94      0.95      2500

    accuracy                           0.95      5000
   macro avg       0.95      0.95      0.95      5000
weighted avg       0.95      0.95      0.95      5000

Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-2.0..3.0].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-2.0..3.0].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-2.0..3.0].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-2.0..2.980392].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-2.0..2.3137255].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-2.0..2.980392].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-2.0..2.9607844].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-2.0..3.0].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-2.0..2.7254903].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-2.0..2.9215686].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-2.0..3.0].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-1.9411765..2.4313724].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-1.8627452..3.0].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-1.4901961..2.5490193].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-1.8235295..2.8627448].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-1.9607843..2.980392].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-1.9215686..3.0].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-1.9607843..3.0].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-2.0..3.0].
Clipping input data to the valid range for imshow with RGB data ([0..1] for floats or [0..255] for integers). Got range [-1.6470588..3.0].
```
### gradio_web.py

```python
"""
猫狗识别系统 - Gradio Web版本
核心功能：
1. 加载预训练的AnimalCNN模型实现猫狗二分类
2. 支持用户上传图像并实时展示识别结果（类别+置信度）
3. 提供友好的Web界面，无需本地GUI环境即可使用
"""
import torch
import torch.nn as nn
import torchvision.transforms as transforms
import gradio as gr
import os

# ====================== 1. 全局配置（集中管理，便于维护） ======================
# 设备配置：自动选择GPU/CPU
DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")
# 模型权重路径
MODEL_WEIGHT_PATH = "./weights/new_best_model.pth"
# 图像预处理配置（与训练时保持一致）
IMAGE_RESIZE_SIZE = (148, 148)
NORMALIZE_MEAN = [0.4, 0.4, 0.4]
NORMALIZE_STD = [0.2, 0.2, 0.2]
# 分类类别映射
CLASS_MAPPING = {0: "cat", 1: "dog"}
# Gradio服务配置
GRADIO_SERVER_NAME = "localhost"
GRADIO_SERVER_PORT = 7860


# ====================== 2. 模型定义（与原代码完全一致） ======================
class AnimalCNN(nn.Module):
    """
    猫狗分类卷积神经网络模型
    结构说明：
    - 4层卷积+池化+批归一化+ReLU激活
    - 3层全连接层，最终输出2类（猫/狗）的预测结果
    """

    def __init__(self, num_classes):
        super(AnimalCNN, self).__init__()
        # 卷积层定义
        self.conv1 = nn.Conv2d(3, 32, kernel_size=3, padding=1)
        self.conv2 = nn.Conv2d(32, 64, kernel_size=3, padding=1)
        self.conv3 = nn.Conv2d(64, 128, kernel_size=3, padding=1)
        self.conv4 = nn.Conv2d(128, 256, kernel_size=3, padding=1)
        # 池化层
        self.pool = nn.MaxPool2d(kernel_size=2, stride=2)
        # 批归一化层
        self.bn1 = nn.BatchNorm2d(32)
        self.bn2 = nn.BatchNorm2d(64)
        self.bn3 = nn.BatchNorm2d(128)
        self.bn4 = nn.BatchNorm2d(256)
        # 全连接层
        self.fc1 = nn.Linear(256 * 9 * 9, 512)
        self.fc2 = nn.Linear(512, 64)
        self.fc3 = nn.Linear(64, num_classes)
        # 激活函数
        self.relu = nn.ReLU()

    def forward(self, x):
        """前向传播过程"""
        x = self.relu(self.bn1(self.pool(self.conv1(x))))
        x = self.relu(self.bn2(self.pool(self.conv2(x))))
        x = self.relu(self.bn3(self.pool(self.conv3(x))))
        x = self.relu(self.bn4(self.pool(self.conv4(x))))
        x = x.view(x.size(0), -1)  # 展平特征图
        x = self.relu(self.fc1(x))
        x = self.relu(self.fc2(x))
        x = self.fc3(x)
        return x


# ====================== 3. 模型加载（含错误处理） ======================
def load_pretrained_model():
    """
    加载预训练的猫狗分类模型
    返回：初始化完成的模型实例
    异常处理：权重文件缺失、模型结构不匹配等场景
    """
    try:
        # 1. 检查权重文件是否存在
        if not os.path.exists(MODEL_WEIGHT_PATH):
            raise FileNotFoundError(f"模型权重文件未找到，请检查路径：{MODEL_WEIGHT_PATH}")

        # 2. 初始化模型
        model = AnimalCNN(num_classes=2)

        # 3. 加载权重文件
        checkpoint = torch.load(
            MODEL_WEIGHT_PATH,
            map_location=DEVICE,
            weights_only=False
        )
        model.load_state_dict(checkpoint)

        # 4. 设置模型为评估模式
        model = model.to(DEVICE)
        model.eval()

        print(f"✅ 模型加载成功，运行设备：{DEVICE}")
        return model

    except FileNotFoundError as e:
        raise Exception(f"❌ 权重文件错误：{str(e)}")
    except RuntimeError as e:
        raise Exception(f"❌ 模型权重加载失败（结构不匹配）：{str(e)}")
    except Exception as e:
        raise Exception(f"❌ 模型加载未知错误：{str(e)}")


# 初始化模型（程序启动时加载，避免重复加载）
try:
    MODEL = load_pretrained_model()
except Exception as e:
    print(f"模型初始化失败：{e}")
    MODEL = None

# ====================== 4. 图像预处理与预测核心逻辑 ======================
# 定义图像预处理管道（与训练/原PyQt版本完全一致）
transform = transforms.Compose([
    transforms.Resize(IMAGE_RESIZE_SIZE),
    transforms.ToTensor(),
    transforms.Normalize(mean=NORMALIZE_MEAN, std=NORMALIZE_STD)
])


def predict_cat_dog(image):
    """
    核心预测函数：处理上传图像并返回猫狗识别结果
    参数：
        image: Gradio上传的图像（PIL.Image格式）
    返回：
        result_text: 识别结果文本（含类别+置信度）
    """
    # 错误处理1：模型未加载成功
    if MODEL is None:
        return "⚠️ 模型加载失败，无法进行识别！请检查权重文件。"

    # 错误处理2：无图像输入
    if image is None:
        return "⚠️ 请先上传一张图像再进行识别！"

    try:
        # 1. 图像预处理
        # 确保图像为RGB格式（处理灰度图/单通道图）
        if image.mode != 'RGB':
            image = image.convert('RGB')
        # 应用预处理并添加batch维度
        image_tensor = transform(image).unsqueeze(0).to(DEVICE)

        # 2. 模型推理（禁用梯度计算）
        with torch.no_grad():
            output = MODEL(image_tensor)
            # 计算各类别置信度
            probabilities = torch.nn.functional.softmax(output, dim=1)
            confidence, predicted = torch.max(probabilities, 1)
            # 映射类别标签
            predicted_class = CLASS_MAPPING[predicted.item()]
            confidence_score = round(confidence.item(), 2)

        # 3. 构造结果文本
        result_text = f"""
识别结果如下：

📌 识别类别：{predicted_class}
📊 置信度：{confidence_score}
        """
        return result_text

    except Exception as e:
        # 处理图像损坏、格式错误等异常
        return f"⚠️ 图像处理失败：{str(e)}\n请上传有效的图片文件（JPG/PNG等）。"


# ====================== 5. Gradio Web界面搭建 ======================
def build_gradio_interface():
    """
    构建Gradio Web界面，复刻原PyQt的功能布局
    界面结构：标题 + 图像上传区 + 结果展示区
    """
    with gr.Blocks(title="猫狗识别系统") as demo:
        # 1. 页面标题
        gr.Markdown("""
        # 🐱🐶 猫狗识别系统
        上传猫狗图像，自动识别类别并展示置信度
        """)

        # 2. 核心功能区（左右布局）
        with gr.Row():
            # 左侧：图像上传/展示区
            with gr.Column(scale=2):
                image_input = gr.Image(
                    type="pil",
                    label="上传待识别图像",
                    height=400,
                    sources=["upload"]  # 仅支持文件上传（与原PyQt一致）
                )

            # 右侧：结果展示区 + 操作按钮
            with gr.Column(scale=1):
                result_output = gr.Textbox(
                    label="识别结果",
                    placeholder="识别结果将显示在这里...",
                    lines=6,
                    interactive=False,
                    elem_id="result-box"  # 自定义样式标识
                )

                # 操作按钮组
                with gr.Row():
                    recognize_btn = gr.Button("猫狗识别", variant="primary", size="lg")
                    clear_btn = gr.Button("清空内容", size="lg")

        # 3. 按钮事件绑定
        # 识别按钮：触发预测逻辑
        recognize_btn.click(
            fn=predict_cat_dog,
            inputs=image_input,
            outputs=result_output
        )

        # 清空按钮：重置输入输出
        def clear_all():
            return None, ""

        clear_btn.click(
            fn=clear_all,
            inputs=[],
            outputs=[image_input, result_output]
        )

        # 4. 页面样式优化（复刻原PyQt的样式）
        demo.css = """
        #result-box {
            color: red;
            font-size: 16px;
            text-align: center;
            padding: 10px;
        }
        .gr-button-primary {
            background-color: #4CAF50;
            color: white;
        }
        """

    # 启动Gradio服务
    demo.launch(
        server_name=GRADIO_SERVER_NAME,
        server_port=GRADIO_SERVER_PORT,
        share=False,  # 仅本地访问
        inbrowser=True  # 自动打开浏览器
    )


# ====================== 6. 程序入口 ======================
if __name__ == "__main__":
    # 检查权重文件路径（提前提示）
    if not os.path.exists(MODEL_WEIGHT_PATH) and MODEL is None:
        print(f"⚠️ 警告：未找到模型权重文件 {MODEL_WEIGHT_PATH}")
        print("请确保权重文件存在后再运行程序！")

    # 构建并启动Web界面
    build_gradio_interface()
```


## 效果展示

![](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20260108214529163.png)

![](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20260108214556551.png)


## 参考资料

https://www.cnblogs.com/vIgorself/articles/19452791

https://github.com/Ironman-creator/Cat-Dog-Recognition-Project












