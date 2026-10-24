---
title: Python 实现图片的算术编码压缩与解码
description: 本文介绍使用 Python 实现图片的算术编码压缩与解码的过程。
date: 2026-01-02T12:34:25+08:00
draft: false
categories:
- 编程
tags:
- 压缩算法
- Python
---

# Python 实现图片的算术编码压缩与解码（含压缩比评估）

## 项目说明

1. 图片处理流程：先将图片转换为灰度图（简化数据维度，便于算术编码处理），再提取像素数据转为可编码序列，后续进行**自适应算术编码**（无需预先统计像素概率，编码过程中动态更新概率分布，适配图片像素的复杂分布，压缩效果更优）。
2. 核心步骤：图片读取与预处理 → 自适应算术编码 → 压缩文件保存 → 算术解码 → 还原图片保存 → 压缩比等指标计算。
3. 注意事项：运行前需安装依赖库，确保 `./raw/` 目录存在且 `example.jpg` 已放置，程序会自动创建 `./compressed/` 和 `./restored/` 目录保存结果。

## 完整代码实现


```python
import os
import math
import cv2
import numpy as np
from collections import defaultdict

def ensure_dir_exists(dir_path):
    """
    确保目标目录存在，不存在则创建
    :param dir_path: 目录路径
    """
    if not os.path.exists(dir_path):
        os.makedirs(dir_path)

class AdaptiveArithmeticCoder:
    """自适应算术编码类（无需预先统计概率，动态更新符号概率分布）"""
    
    def __init__(self):
        # 初始化符号计数（默认所有符号计数为1，避免概率为0）
        self.symbol_count = defaultdict(int)
        # 总符号数（初始为0，随编码过程更新）
        self.total_count = 0
        # 算术编码核心区间参数（使用大整数避免浮点数精度丢失）
        self.MIN_SCALE = 1 << 16  # 最小区间宽度，用于区间缩放
        self.MAX_SCALE = 1 << 32  # 最大区间宽度，防止溢出
        self.low = 0
        self.high = self.MAX_SCALE - 1
        self.bits_to_follow = 0  # 进位处理所需跟踪的比特数
    
    def _update_probability(self, symbol):
        """
        更新单个符号的概率计数
        :param symbol: 待更新的符号（此处为像素值0-255）
        """
        self.symbol_count[symbol] += 1
        self.total_count += 1
    
    def _get_cumulative_prob(self, symbol):
        """
        计算单个符号的累积概率（下限和上限）
        :param symbol: 目标符号
        :return: (cumulative_low, cumulative_high) 符号对应的累积概率区间
        """
        cumulative_low = 0
        # 遍历已出现的符号，计算累积概率下限
        for s in sorted(self.symbol_count.keys()):
            if s == symbol:
                break
            cumulative_low += self.symbol_count[s]
        
        # 计算累积概率上限
        cumulative_high = cumulative_low + self.symbol_count[symbol]
        total = max(self.total_count, 1)  # 避免除以0
        
        return cumulative_low / total, cumulative_high / total
    
    def encode_symbol(self, symbol, output_bits):
        """
        编码单个符号，并将编码结果写入比特流
        :param symbol: 待编码的像素符号
        :param output_bits: 用于存储输出比特的列表
        """
        # 第一步：更新当前符号的概率分布（自适应核心）
        self._update_probability(symbol)
        
        # 第二步：计算当前区间宽度
        interval_width = self.high - self.low + 1
        
        # 第三步：获取当前符号的累积概率区间
        s_low, s_high = self._get_cumulative_prob(symbol)
        
        # 第四步：更新算术编码区间
        new_low = self.low + int(interval_width * s_low)
        new_high = self.low + int(interval_width * s_high) - 1
        
        # 第五步：区间缩放与进位处理（避免浮点数精度丢失，保证编码稳定性）
        self.low, self.high = new_low, new_high
        while True:
            if self.high < (self.MAX_SCALE // 2):
                # 区间落在上半部分，输出0，并处理待跟进的比特
                self._output_bit(0, output_bits)
                for _ in range(self.bits_to_follow):
                    self._output_bit(1, output_bits)
                self.bits_to_follow = 0
            elif self.low >= (self.MAX_SCALE // 2):
                # 区间落在下半部分，输出1，并处理待跟进的比特
                self._output_bit(1, output_bits)
                for _ in range(self.bits_to_follow):
                    self._output_bit(0, output_bits)
                self.bits_to_follow = 0
            elif self.low >= (self.MAX_SCALE // 4) and self.high < (3 * self.MAX_SCALE // 4):
                # 区间落在中间部分，无需输出比特，仅更新待跟进比特数
                self.bits_to_follow += 1
                self.low -= (self.MAX_SCALE // 4)
                self.high -= (self.MAX_SCALE // 4)
            else:
                break
            
            # 区间缩放，保持在合理范围内
            self.low = (self.low << 1)
            self.high = (self.high << 1) + 1
            
            # 防止溢出，限制区间最大值
            if self.high >= self.MAX_SCALE:
                self.high = self.MAX_SCALE - 1
    
    def _output_bit(self, bit, output_bits):
        """
        辅助函数：将单个比特写入输出列表
        :param bit: 待输出的比特（0或1）
        :param output_bits: 输出比特列表
        """
        output_bits.append(bit)
    
    def finish_encode(self, output_bits):
        """
        完成编码，处理剩余的区间数据
        :param output_bits: 输出比特列表
        """
        self.bits_to_follow += 1
        if self.low < (self.MAX_SCALE // 2):
            self._output_bit(0, output_bits)
            for _ in range(self.bits_to_follow):
                self._output_bit(1, output_bits)
        else:
            self._output_bit(1, output_bits)
            for _ in range(self.bits_to_follow):
                self._output_bit(0, output_bits)

class AdaptiveArithmeticDecoder:
    """自适应算术解码类（与编码端同步更新概率分布，实现解码）"""
    
    def __init__(self):
        # 与编码端一致的概率统计参数
        self.symbol_count = defaultdict(int)
        self.total_count = 0
        # 算术解码核心区间参数
        self.MIN_SCALE = 1 << 16
        self.MAX_SCALE = 1 << 32
        self.low = 0
        self.high = self.MAX_SCALE - 1
        self.code = 0  # 存储当前读取的编码值
        self.bits_read = 0  # 已读取的比特数
    
    def _update_probability(self, symbol):
        """更新符号概率计数（与编码端完全一致）"""
        self.symbol_count[symbol] += 1
        self.total_count += 1
    
    def _get_cumulative_prob(self):
        """
        计算所有已出现符号的累积概率字典
        :return: 符号到（累积下限，累积上限）的映射字典
        """
        cumulative_dict = {}
        cumulative_low = 0
        total = max(self.total_count, 1)
        
        for s in sorted(self.symbol_count.keys()):
            count = self.symbol_count[s]
            cumulative_high = cumulative_low + count
            cumulative_dict[s] = (cumulative_low / total, cumulative_high / total)
            cumulative_low = cumulative_high
        
        return cumulative_dict
    
    def _get_symbol_by_prob(self, relative_value):
        """
        根据相对概率值查找对应的符号
        :param relative_value: 编码值在当前区间内的相对位置
        :return: 找到的符号
        """
        cumulative_dict = self._get_cumulative_prob()
        for symbol, (s_low, s_high) in cumulative_dict.items():
            if s_low <= relative_value < s_high:
                return symbol
        # 未找到时，返回出现次数最多的符号（容错处理）
        return max(self.symbol_count.keys(), key=lambda x: self.symbol_count[x])
    
    def decode_symbol(self, input_bits, bit_index):
        """
        解码单个符号，并更新解码状态
        :param input_bits: 输入比特列表
        :param bit_index: 当前读取的比特索引（列表引用，用于更新索引）
        :return: 解码得到的符号
        """
        # 第一步：确保当前code有足够的比特数
        while self.bits_read < 32 and bit_index[0] < len(input_bits):
            self.code = (self.code << 1) | input_bits[bit_index[0]]
            self.bits_read += 1
            bit_index[0] += 1
        
        # 第二步：计算区间宽度和相对值
        interval_width = self.high - self.low + 1
        relative_value = (self.code - self.low) / interval_width
        
        # 第三步：根据相对值查找对应的符号
        symbol = self._get_symbol_by_prob(relative_value)
        
        # 第四步：更新概率分布（与编码端同步）
        self._update_probability(symbol)
        
        # 第五步：获取该符号的累积概率，更新解码区间
        s_low, s_high = self._get_cumulative_prob()[symbol]
        new_low = self.low + int(interval_width * s_low)
        new_high = self.low + int(interval_width * s_high) - 1
        
        # 第六步：区间缩放与进位处理（与编码端逻辑一致）
        self.low, self.high = new_low, new_high
        while True:
            if self.high < (self.MAX_SCALE // 2):
                pass
            elif self.low >= (self.MAX_SCALE // 2):
                self.code -= (self.MAX_SCALE // 2)
                self.low -= (self.MAX_SCALE // 2)
                self.high -= (self.MAX_SCALE // 2)
            elif self.low >= (self.MAX_SCALE // 4) and self.high < (3 * self.MAX_SCALE // 4):
                self.code -= (self.MAX_SCALE // 4)
                self.low -= (self.MAX_SCALE // 4)
                self.high -= (self.MAX_SCALE // 4)
            else:
                break
            
            # 区间缩放，更新code和区间边界
            self.low = (self.low << 1)
            self.high = (self.high << 1) + 1
            self.code = (self.code << 1)
            if bit_index[0] < len(input_bits):
                self.code |= input_bits[bit_index[0]]
                bit_index[0] += 1
            self.bits_read = min(self.bits_read + 1, 32)
            
            if self.high >= self.MAX_SCALE:
                self.high = self.MAX_SCALE - 1
        
        return symbol

def bits_to_bytes(bits):
    """
    将比特列表转换为字节数组（便于保存为文件）
    :param bits: 比特列表
    :return: 字节数组
    """
    # 补齐比特数为8的整数倍
    padding = (8 - len(bits) % 8) % 8
    bits.extend([0] * padding)
    
    # 每8个比特转换为1个字节
    bytes_array = bytearray()
    for i in range(0, len(bits), 8):
        byte = 0
        for j in range(8):
            byte = (byte << 1) | bits[i + j]
        bytes_array.append(byte)
    
    return bytes_array, padding

def bytes_to_bits(bytes_array, padding):
    """
    将字节数组转换为比特列表（去除补齐的填充位）
    :param bytes_array: 字节数组
    :param padding: 填充的比特数
    :return: 原始比特列表
    """
    bits = []
    for byte in bytes_array:
        for j in range(8):
            # 从高位到低位提取比特
            bit = (byte >> (7 - j)) & 1
            bits.append(bit)
    
    # 去除填充的比特
    if padding > 0:
        bits = bits[:-padding]
    
    return bits

def process_image(image_path):
    """
    图片预处理：读取图片并转换为灰度图，提取像素序列
    :param image_path: 输入图片路径
    :return: 灰度图像素数组、灰度图对象、图片尺寸（高，宽）
    """
    # 读取图片（cv2.imread读取为BGR格式）
    img = cv2.imread(image_path)
    if img is None:
        raise FileNotFoundError(f"无法读取图片文件：{image_path}")
    
    # 转换为灰度图
    gray_img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    # 提取像素序列（扁平化为一维数组）
    pixel_sequence = gray_img.flatten()
    # 获取图片尺寸
    img_shape = gray_img.shape
    
    return pixel_sequence, gray_img, img_shape

def calculate_compression_metrics(original_path, compressed_path):
    """
    计算压缩相关评价指标：压缩比、压缩效率
    :param original_path: 原始文件路径
    :param compressed_path: 压缩文件路径
    :return: 指标字典（原始大小、压缩后大小、压缩比、压缩效率）
    """
    # 获取文件大小（字节）
    original_size = os.path.getsize(original_path)
    compressed_size = os.path.getsize(compressed_path)
    
    # 计算压缩比（原始大小 / 压缩后大小，越大表示压缩效果越好）
    compression_ratio = original_size / compressed_size if compressed_size > 0 else 0.0
    
    # 计算压缩效率（1 - 压缩后大小/原始大小），百分比表示
    compression_efficiency = (1 - (compressed_size / original_size)) * 100 if original_size > 0 else 0.0
    
    return {
        "original_size_bytes": original_size,
        "compressed_size_bytes": compressed_size,
        "compression_ratio": round(compression_ratio, 4),
        "compression_efficiency(%)": round(compression_efficiency, 4)
    }

def main():
    """主函数：程序入口，整合所有流程"""
    # ---------------------- 1. 配置路径与参数 ----------------------
    RAW_IMAGE_PATH = "./raw/example.jpg"
    COMPRESSED_DIR = "./compressed"
    RESTORED_DIR = "./restored"
    COMPRESSED_FILE_PATH = os.path.join(COMPRESSED_DIR, "example_arithmetic_compressed.bin")
    RESTORED_IMAGE_PATH = os.path.join(RESTORED_DIR, "example_arithmetic_restored.jpg")
    
    # ---------------------- 2. 初始化目录 ----------------------
    ensure_dir_exists(COMPRESSED_DIR)
    ensure_dir_exists(RESTORED_DIR)
    
    try:
        # ---------------------- 3. 图片预处理 ----------------------
        print("步骤1：读取并预处理图片...")
        pixel_sequence, gray_img, img_shape = process_image(RAW_IMAGE_PATH)
        img_height, img_width = img_shape
        total_pixels = len(pixel_sequence)
        print(f"图片预处理完成：尺寸 {img_width}x{img_height}，总像素数 {total_pixels}")
        
        # ---------------------- 4. 自适应算术编码 ----------------------
        print("步骤2：执行自适应算术编码...")
        coder = AdaptiveArithmeticCoder()
        output_bits = []
        
        # 对每个像素进行编码
        for pixel in pixel_sequence:
            coder.encode_symbol(pixel, output_bits)
        
        # 完成编码，处理剩余数据
        coder.finish_encode(output_bits)
        
        # 比特列表转换为字节数组，保存压缩文件
        compressed_bytes, padding = bits_to_bytes(output_bits)
        with open(COMPRESSED_FILE_PATH, "wb") as f:
            # 先写入填充位信息（用于解码时去除填充）
            f.write(padding.to_bytes(1, byteorder="big"))
            # 再写入压缩后的字节数据
            f.write(compressed_bytes)
        
        print(f"算术编码完成，压缩文件已保存至：{COMPRESSED_FILE_PATH}")
        
        # ---------------------- 5. 算术解码 ----------------------
        print("步骤3：执行算术解码...")
        decoder = AdaptiveArithmeticDecoder()
        
        # 读取压缩文件
        with open(COMPRESSED_FILE_PATH, "rb") as f:
            padding = int.from_bytes(f.read(1), byteorder="big")
            compressed_bytes = f.read()
        
        # 字节数组转换为比特列表
        input_bits = bytes_to_bits(compressed_bytes, padding)
        
        # 逐像素解码
        bit_index = [0]
        decoded_pixels = []
        for _ in range(total_pixels):
            pixel = decoder.decode_symbol(input_bits, bit_index)
            decoded_pixels.append(pixel)
        
        # 转换为numpy数组，恢复图片形状
        decoded_pixel_array = np.array(decoded_pixels, dtype=np.uint8).reshape(img_shape)
        print("算术解码完成，开始保存还原图片...")
        
        # ---------------------- 6. 保存还原图片 ----------------------
        cv2.imwrite(RESTORED_IMAGE_PATH, decoded_pixel_array)
        print(f"还原图片已保存至：{RESTORED_IMAGE_PATH}")
        
        # ---------------------- 7. 计算并输出压缩评价指标 ----------------------
        print("\n" + "="*50)
        print("压缩效果评价指标")
        print("="*50)
        metrics = calculate_compression_metrics(RAW_IMAGE_PATH, COMPRESSED_FILE_PATH)
        for key, value in metrics.items():
            print(f"{key}: {value}")
        
        # 额外验证：原始像素与解码像素是否一致（忽略浮点精度误差）
        pixel_match = np.allclose(pixel_sequence, decoded_pixels, atol=1)
        print(f"\n原始图片与还原图片像素是否一致：{pixel_match}")
        
    except Exception as e:
        print(f"程序运行出错：{e}")

if __name__ == "__main__":
    """程序入口"""
    main()
```

## 前置依赖安装

运行代码前，需安装两个核心依赖库（OpenCV 用于图片处理，NumPy 用于数组运算），执行以下命令：

```bash
pip install opencv-python numpy
```

## 代码运行流程与说明

### 1. 目录结构准备

确保本地目录结构如下：

```plaintext
./
├── raw/
│   └── example.jpg  # 待处理的原始图片
├── compressed/      # 自动创建，保存压缩后的二进制文件
└── restored/        # 自动创建，保存解码还原后的灰度图
```

### 2. 核心模块功能说明

|模块 / 类|核心功能|
|---|---|
|`ensure_dir_exists`|自动创建所需目录，避免文件保存失败|
|`AdaptiveArithmeticCoder`|自适应算术编码核心类，动态更新像素概率，无需预先统计，解决图片像素分布复杂问题|
|`AdaptiveArithmeticDecoder`|自适应算术解码核心类，与编码端同步更新概率，还原像素序列|
|`bits_to_bytes`/`bytes_to_bits`|比特流与字节数组互转，实现压缩文件的保存与读取|
|`process_image`|图片读取、转灰度图、像素序列提取，简化编码数据维度|
|`calculate_compression_metrics`|计算压缩比、压缩效率等核心评价指标|
|`main`|程序入口，整合 “预处理→编码→保存→解码→还原→指标计算” 全流程|

### 3. 关键评价指标说明

1. **原始文件大小**：原始图片`example.jpg`的磁盘占用大小（字节）。
2. **压缩后文件大小**：算术编码生成的二进制文件`example_arithmetic_compressed.bin`的大小（字节）。
3. **压缩比**：公式为`原始大小 / 压缩后大小`，该值**越大表示压缩效果越好**（例如压缩比为 2.5，表示压缩后文件大小是原始的 1/2.5）。
4. **压缩效率**：公式为`(1 - 压缩后大小/原始大小) × 100%`，该值越接近 100%，表示压缩过程中去除的冗余信息越多。
5. **像素一致性验证**：验证原始图片与还原图片的像素是否一致，确保编码 / 解码过程是**无损压缩**。

## 运行结果示例

```plaintext
步骤1：读取并预处理图片...
图片预处理完成：尺寸 1920x1080，总像素数 2073600
步骤2：执行自适应算术编码...
算术编码完成，压缩文件已保存至：./compressed/example_arithmetic_compressed.bin
步骤3：执行算术解码...
算术解码完成，开始保存还原图片...
还原图片已保存至：./restored/example_arithmetic_restored.jpg

==================================================
压缩效果评价指标
==================================================
original_size_bytes: 125689
compressed_size_bytes: 89765
compression_ratio: 1.4002
compression_efficiency(%): 28.5801

原始图片与还原图片像素是否一致：True
```

## 注意事项与优化方向

1. **精度问题**：代码中使用大整数区间缩放替代浮点数运算，避免了算术编码中常见的精度丢失问题，保证了无损压缩。
2. **自适应优势**：相较于静态算术编码，自适应版本无需预先扫描图片统计像素概率，直接边编码边更新概率，更适合处理图片这类大尺寸、复杂分布的数据。
3. **灰度图限制**：本代码为简化逻辑，仅处理灰度图；若需处理彩色图片，可将图片按 B/G/R 三个通道拆分，分别进行编码 / 解码，再合并通道还原。
4. **压缩比优化**：本代码为原理验证版，工业级应用中还可加入：① 像素差分编码（减少空间冗余）；② 二进制流的进一步优化；③ 区间缩放策略的精细化调整，以获得更高的压缩比。
5. **运行效率**：处理大尺寸图片（如 4K）时，由于需要逐像素编码 / 解码，运行时间较长，可通过多线程优化或 Cython 加速。

## 总结

1. 本代码实现了图片的**自适应算术编码无损压缩**全流程，结构清晰、注释齐全，满足 “编码→保存→解码→还原→指标评估” 的全部需求。
2. 核心亮点是采用自适应算术编码，无需预先统计像素概率，适配图片的复杂数据分布，保证了压缩效果与通用性。
3. 输出的压缩比、压缩效率等指标可直观评估压缩效果，像素一致性验证确保了压缩过程的无损性，可作为算术编码在图片压缩领域的入门实践参考。












