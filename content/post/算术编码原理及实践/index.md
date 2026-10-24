---
title: 一文读懂算术编码原理：从核心思想到实战示例
description: 本文介绍算术编码的核心思想和实战示例。
date: 2026-01-01T12:34:25+08:00
draft: false
categories:
- 算法
tags:
- 压缩算法
- Python
---


# 一文读懂算术编码原理：从核心思想到实战示例

算术编码（Arithmetic Coding）是一种高效的无损数据压缩算法，与霍夫曼编码齐名，但在处理短序列、高冗余数据时往往能实现更高的压缩比。它打破了 “一个符号对应一个编码码字” 的传统思路，而是将整个待编码数据序列映射为一个 `[0,1)` 区间内的小数 ，最终通过输出这个小数的二进制表示实现压缩。本文将从核心原理入手，结合具体示例逐步拆解，并通过 Python 代码实现一个简易版算术编码与解码。

## 一、算术编码的核心思想

与霍夫曼编码的 “符号→码字” 一对一映射不同，算术编码的核心逻辑可以概括为 3 点：

1. **基于概率的区间划分**：首先统计待编码数据中每个符号的出现概率，将初始区间`[0,1)`按照各符号的概率比例进行划分，每个符号对应一个子区间。
2. **迭代收缩区间**：逐个处理待编码序列中的符号，每处理一个符号，就将当前的区间更新为该符号对应的子区间，然后在新的子区间内再次按照符号概率进行划分，重复此过程，直到所有符号处理完毕。
3. **区间映射为小数**：最终得到的收缩后区间内的任意一个小数，都可以唯一映射回原始数据序列，这个小数就是编码结果，通常转换为二进制以进一步节省存储空间。

关键优势：无需为单个符号分配整数长度的码字，对概率分布更敏感，尤其适合符号概率差异较小的场景，压缩效率更高。

算术编码为什么可以压缩数据？

**算术编码的压缩本质，就是在保留字符排列顺序的同时，对于更高频出现的字符，也就是概率更大的字符，赋予更大的小数区间。**
## 二、算术编码的前置准备

在开始具体示例前，我们需要明确两个基础概念，这是理解算术编码的前提：

1. **符号概率统计**：必须先统计待编码数据集中每个符号的出现概率（或频率），概率之和必须为 1。
2. **区间边界计算**：每个符号对应的区间是`[累积概率下限, 累积概率上限)`，累积概率是指该符号之前所有符号的概率之和。

举个简单的前提示例：若待编码符号集为`{a, b, c}`，对应的概率分别为`0.5, 0.3, 0.2`，则它们的区间划分如下：

- 符号`a`：`[0, 0.5)`（累积下限 = 0，累积上限 = 0+0.5=0.5）
- 符号`b`：`[0.5, 0.8)`（累积下限 = 0.5，累积上限 = 0.5+0.3=0.8）
- 符号`c`：`[0.8, 1.0)`（累积下限 = 0.8，累积上限 = 0.8+0.2=1.0）

注意：频率表可以自定义。
## 三、算术编码分步示例（手把手拆解）

为了让原理更易懂，我们选择一个简单且典型的示例进行分步讲解：

### 示例设定

- 待编码数据序列：`"a b a c"`（简化为符号序列`[a, b, a, c]`，空格仅用于分隔）
- 符号集与预先统计的概率（假设已通过样本统计得到）：

|符号|出现概率|
|---|---|
|a|0.5|
|b|0.3|
|c|0.2|

### 步骤 1：初始化初始区间

算术编码的初始区间固定为`[low, high) = [0.0, 1.0)`，其中`low`是区间下限，`high`是区间上限，初始时`low=0.0`，`high=1.0`。

### 步骤 2：逐个处理符号，迭代收缩区间

区间收缩的核心公式（关键！）：对于当前区间`[cur_low, cur_high)`，处理符号`s`（对应区间`[s_low, s_high)`）时，新的区间`[new_low, new_high)`计算如下：

1. `interval_width = cur_high - cur_low`（当前区间的宽度）
2. `new_low = cur_low + interval_width * s_low`（新下限 = 当前下限 + 区间宽度 × 符号下限）
3. `new_high = cur_low + interval_width * s_high`（新上限 = 当前下限 + 区间宽度 × 符号上限）

下面我们按照这个公式，逐步处理`[a, b, a, c]`中的每个符号：

#### 第 1 步：处理第一个符号 `a`（对应区间`[0.0, 0.5)`）

- 当前区间：`cur_low=0.0`，`cur_high=1.0`
- 区间宽度：`interval_width = 1.0 - 0.0 = 1.0`
- 新下限：`new_low = 0.0 + 1.0 * 0.0 = 0.0`
- 新上限：`new_high = 0.0 + 1.0 * 0.5 = 0.5`
- 处理后区间：`[0.0, 0.5)`

#### 第 2 步：处理第二个符号 `b`（对应区间`[0.5, 0.8)`）

- 当前区间：`cur_low=0.0`，`cur_high=0.5`
- 区间宽度：`interval_width = 0.5 - 0.0 = 0.5`
- 新下限：`new_low = 0.0 + 0.5 * 0.5 = 0.25`
- 新上限：`new_high = 0.0 + 0.5 * 0.8 = 0.4`
- 处理后区间：`[0.25, 0.4)`

#### 第 3 步：处理第三个符号 `a`（对应区间`[0.0, 0.5)`）

- 当前区间：`cur_low=0.25`，`cur_high=0.4`
- 区间宽度：`interval_width = 0.4 - 0.25 = 0.15`
- 新下限：`new_low = 0.25 + 0.15 * 0.0 = 0.25`
- 新上限：`new_high = 0.25 + 0.15 * 0.5 = 0.25 + 0.075 = 0.325`
- 处理后区间：`[0.25, 0.325)`

#### 第 4 步：处理第四个符号 `c`（对应区间`[0.8, 1.0)`）

- 当前区间：`cur_low=0.25`，`cur_high=0.325`
- 区间宽度：`interval_width = 0.325 - 0.25 = 0.075`
- 新下限：`new_low = 0.25 + 0.075 * 0.8 = 0.25 + 0.06 = 0.31`
- 新上限：`new_high = 0.25 + 0.075 * 1.0 = 0.25 + 0.075 = 0.325`
- 处理后区间：`[0.31, 0.325)`

### 步骤 3：获取最终编码结果

所有符号处理完毕后，我们得到了最终的收缩区间`[0.31, 0.325)`。此时，**这个区间内的任意一个小数都可以作为算术编码的结果**（因为解码时可以通过这个小数唯一还原出原始符号序列）。

为了方便表示，我们可以选择区间的中点作为编码结果：`(0.31 + 0.325) / 2 = 0.3175`。

如果需要进一步压缩，还可以将`0.3175`转换为二进制表示（注：小数的二进制转换是 “乘 2 取整，顺序排列”），最终的二进制串就是更紧凑的压缩结果。

## 四、Python 实现简易算术编码与解码

下面我们通过 Python 代码实现上述示例的算术编码与解码，代码保持简洁易懂，聚焦核心逻辑（不考虑复杂的精度优化和二进制优化，适合原理验证）。

### 完整代码实现

```python
def calculate_cumulative_prob(prob_dict):
    """
    计算符号的累积概率，返回符号对应的[下限, 上限]区间字典
    :param prob_dict: 符号-概率字典
    :return: 符号-区间[low, high]字典
    """
    cumulative_prob = 0.0
    symbol_interval = {}
    
    # 按符号排序（保证结果可复现），计算累积区间
    for symbol in sorted(prob_dict.keys()):
        prob = prob_dict[symbol]
        symbol_interval[symbol] = (cumulative_prob, cumulative_prob + prob)
        cumulative_prob += prob
    
    return symbol_interval

def arithmetic_encode(symbol_sequence, prob_dict):
    """
    算术编码核心函数
    :param symbol_sequence: 待编码的符号序列（列表/元组）
    :param prob_dict: 符号-概率字典
    :return: 编码结果（区间内的小数）、最终区间[low, high]
    """
    # 1. 初始化区间
    low = 0.0
    high = 1.0
    
    # 2. 获取符号的累积区间
    symbol_interval = calculate_cumulative_prob(prob_dict)
    
    # 3. 迭代处理每个符号，收缩区间
    for symbol in symbol_sequence:
        if symbol not in symbol_interval:
            raise ValueError(f"符号 {symbol} 不在预定义的符号集中")
        
        # 获取当前符号对应的区间
        s_low, s_high = symbol_interval[symbol]
        # 计算当前区间宽度
        interval_width = high - low
        # 更新区间
        new_low = low + interval_width * s_low
        new_high = low + interval_width * s_high
        
        # 更新当前区间为新区间
        low, high = new_low, new_high
    
    # 4. 选择区间中点作为编码结果（也可选择low或high）
    encode_result = (low + high) / 2.0
    
    return encode_result, (low, high)

def arithmetic_decode(encode_value, prob_dict, sequence_length):
    """
    算术解码核心函数（根据编码值还原符号序列）
    :param encode_value: 算术编码结果（小数）
    :param prob_dict: 符号-概率字典
    :param sequence_length: 原始符号序列的长度（必须预先知晓）
    :return: 还原的符号序列
    """
    # 1. 初始化变量
    low = 0.0
    high = 1.0
    decoded_sequence = []
    symbol_interval = calculate_cumulative_prob(prob_dict)
    symbol_list = sorted(prob_dict.keys())
    
    # 2. 迭代解码每个符号（需知道序列长度，否则无法停止）
    for _ in range(sequence_length):
        interval_width = high - low
        # 计算编码值在当前区间内的相对位置
        relative_value = (encode_value - low) / interval_width
        
        # 3. 找到相对位置对应的符号
        decoded_symbol = None
        for symbol in symbol_list:
            s_low, s_high = symbol_interval[symbol]
            if s_low <= relative_value < s_high:
                decoded_symbol = symbol
                # 4. 收缩区间，为下一个符号解码做准备
                new_low = low + interval_width * s_low
                new_high = low + interval_width * s_high
                low, high = new_low, new_high
                break
        
        if decoded_symbol is None:
            raise ValueError("无法找到对应的符号，解码失败")
        
        decoded_sequence.append(decoded_symbol)
    
    return decoded_sequence

# ---------------------- 测试示例 ----------------------
if __name__ == "__main__":
    # 1. 定义示例参数
    prob_dict = {"a": 0.5, "b": 0.3, "c": 0.2}  # 符号-概率字典
    symbol_sequence = ["a", "b", "a", "c"]       # 待编码符号序列
    print(f"原始符号序列：{symbol_sequence}")
    print(f"符号概率分布：{prob_dict}\n")
    
    # 2. 执行算术编码
    encode_result, final_interval = arithmetic_encode(symbol_sequence, prob_dict)
    print(f"算术编码结果（区间中点）：{encode_result:.6f}")
    print(f"最终收缩区间：[{final_interval[0]:.6f}, {final_interval[1]:.6f})\n")
    
    # 3. 执行算术解码
    sequence_length = len(symbol_sequence)
    decoded_sequence = arithmetic_decode(encode_result, prob_dict, sequence_length)
    print(f"解码得到的符号序列：{decoded_sequence}")
    print(f"编码与解码是否一致：{symbol_sequence == decoded_sequence}")
```

### 代码运行结果

```plaintext
原始符号序列：['a', 'b', 'a', 'c']
符号概率分布：{'a': 0.5, 'b': 0.3, 'c': 0.2}

算术编码结果（区间中点）：0.317500
最终收缩区间：[0.310000, 0.325000)

解码得到的符号序列：['a', 'b', 'a', 'c']
编码与解码是否一致：True
```

### 代码关键说明

1. **累积概率计算**：`calculate_cumulative_prob`函数用于将符号概率转换为对应的区间，保证每个符号的区间不重叠且覆盖`[0,1)`。
2. **编码逻辑**：严格遵循前文的区间收缩公式，逐个处理符号，最终返回区间中点作为编码结果。
3. **解码逻辑**：解码是编码的逆过程，需要预先知晓原始序列长度（这是算术编码的一个特点，通常需要在压缩结果中记录该信息），通过相对位置找到对应的符号，并迭代收缩区间。
4. **可复现性**：对符号进行排序后处理，避免字典无序导致的结果差异。

## 五、算术编码的关键注意事项

1. **精度问题**：在实际应用中，随着符号序列长度增加，区间会无限收缩，普通浮点数（Python 中的`float`是 64 位双精度）会出现精度丢失，此时需要使用高精度数值类型或采用 “区间缩放 + 进位处理” 的策略。
2. **概率更新**：本文示例使用的是固定概率（静态算术编码），实际场景中更常用自适应算术编码，即编码过程中动态更新符号概率，无需预先统计。
3. **解码依赖**：解码时必须知晓原始符号的概率分布和序列长度，否则无法正确还原数据，这部分信息需要作为压缩头存储在编码结果中。
4. **压缩比优势**：当符号数量较多、概率分布较均匀时，算术编码的压缩比明显优于霍夫曼编码，因为它无需为单个符号分配整数长度码字，能更充分地利用概率信息。

## 六、总结

算术编码的核心是 “区间收缩” 与 “概率映射”，它将整个数据序列映射为一个小数，打破了传统编码的码字限制，实现了更高的压缩效率。本文通过一个简单的符号序列示例，逐步拆解了编码的核心步骤，并通过 Python 代码实现了简易的编码与解码，验证了算术编码的可行性。

需要注意的是，本文实现的是基础版本的算术编码，实际工业级应用（如 JPEG、PNG 压缩标准中）还需要解决精度丢失、自适应概率更新、二进制优化等问题，但理解本文的核心原理，是深入学习高级算术编码的基础。

算术编码的魅力在于它对概率的极致利用，这种 “整体映射” 的思路也为后续的无损压缩算法提供了重要启发，值得每一位对数据压缩感兴趣的开发者深入研究。

## 附录封装版代码

### pyae.py

```python
from decimal import Decimal


class ArithmeticEncoding:
    """
    ArithmeticEncoding is a class for building the arithmetic encoding.
    """

    def __init__(self, frequency_table, save_stages=False):
        """
        frequency_table: Frequency table as a dictionary where key is the symbol and value is the frequency.
        save_stages: If True, then the intervals of each stage are saved in a list. Note that setting save_stages=True may cause memory overflow if the message is large
        """

        self.save_stages = save_stages
        if (save_stages == True):
            print("WARNING: Setting save_stages=True may cause memory overflow if the message is large.")

        self.probability_table = self.get_probability_table(frequency_table)

    def get_probability_table(self, frequency_table):
        """
        Calculates the probability table out of the frequency table.

        frequency_table: A table of the term frequencies.

        Returns the probability table.
        """
        total_frequency = sum(list(frequency_table.values()))

        probability_table = {}
        for key, value in frequency_table.items():
            probability_table[key] = value / total_frequency

        return probability_table

    def get_encoded_value(self, last_stage_probs):
        """
        After encoding the entire message, this method returns the single value that represents the entire message.

        last_stage_probs: A list of the probabilities in the last stage.

        Returns the minimum and maximum probabilites in the last stage in addition to the value encoding the message.
        """
        last_stage_probs = list(last_stage_probs.values())
        last_stage_values = []
        for sublist in last_stage_probs:
            for element in sublist:
                last_stage_values.append(element)

        last_stage_min = min(last_stage_values)
        last_stage_max = max(last_stage_values)
        encoded_value = (last_stage_min + last_stage_max) / 2

        return last_stage_min, last_stage_max, encoded_value

    def process_stage(self, probability_table, stage_min, stage_max):
        """
        Processing a stage in the encoding/decoding process.

        probability_table: The probability table.
        stage_min: The minumim probability of the current stage.
        stage_max: The maximum probability of the current stage.

        Returns the probabilities in the stage.
        """

        stage_probs = {}
        stage_domain = stage_max - stage_min
        for term_idx in range(len(probability_table.items())):
            term = list(probability_table.keys())[term_idx]
            term_prob = Decimal(probability_table[term])
            cum_prob = term_prob * stage_domain + stage_min
            stage_probs[term] = [stage_min, cum_prob]
            stage_min = cum_prob
        return stage_probs

    def encode(self, msg, probability_table):
        """
        Encodes a message using arithmetic encoding.

        msg: The message to be encoded.
        probability_table: The probability table.

        Returns the encoder, the floating-point value representing the encoded message, and the maximum and minimum values of the interval in which the floating-point value falls.
        """

        msg = list(msg)

        encoder = []

        stage_min = Decimal(0.0)
        stage_max = Decimal(1.0)

        for msg_term_idx in range(len(msg)):
            stage_probs = self.process_stage(probability_table, stage_min, stage_max)

            msg_term = msg[msg_term_idx]
            stage_min = stage_probs[msg_term][0]
            stage_max = stage_probs[msg_term][1]

            if self.save_stages:
                encoder.append(stage_probs)

        last_stage_probs = self.process_stage(probability_table, stage_min, stage_max)

        if self.save_stages:
            encoder.append(last_stage_probs)

        interval_min_value, interval_max_value, encoded_msg = self.get_encoded_value(last_stage_probs)

        return encoded_msg, encoder, interval_min_value, interval_max_value

    def process_stage_binary(self, float_interval_min, float_interval_max, stage_min_bin, stage_max_bin):
        """
        Processing a stage in the encoding/decoding process.

        float_interval_min: The minimum floating-point value in the interval in which the floating-point value that encodes the message is located.
        float_interval_max: The maximum floating-point value in the interval in which the floating-point value that encodes the message is located.
        stage_min_bin: The minimum binary number in the current stage.
        stage_max_bin: The maximum binary number in the current stage.

        Returns the probabilities of the terms in this stage. There are only 2 terms.
        """

        stage_mid_bin = stage_min_bin + "1"
        stage_min_bin = stage_min_bin + "0"

        stage_probs = {}
        stage_probs[0] = [stage_min_bin, stage_mid_bin]
        stage_probs[1] = [stage_mid_bin, stage_max_bin]

        return stage_probs

    def encode_binary(self, float_interval_min, float_interval_max):
        """
        Calculates the binary code that represents the floating-point value that encodes the message.

        float_interval_min: The minimum floating-point value in the interval in which the floating-point value that encodes the message is located.
        float_interval_max: The maximum floating-point value in the interval in which the floating-point value that encodes the message is located.

        Returns the binary code representing the encoded message.
        """

        binary_encoder = []
        binary_code = None

        stage_min_bin = "0.0"
        stage_max_bin = "1.0"

        stage_probs = {}
        stage_probs[0] = [stage_min_bin, "0.1"]
        stage_probs[1] = ["0.1", stage_max_bin]

        while True:
            if float_interval_max < bin2float(stage_probs[0][1]):
                stage_min_bin = stage_probs[0][0]
                stage_max_bin = stage_probs[0][1]
            else:
                stage_min_bin = stage_probs[1][0]
                stage_max_bin = stage_probs[1][1]

            if self.save_stages:
                binary_encoder.append(stage_probs)

            stage_probs = self.process_stage_binary(float_interval_min,
                                                    float_interval_max,
                                                    stage_min_bin,
                                                    stage_max_bin)

            # print(stage_probs[0][0], bin2float(stage_probs[0][0]))
            # print(stage_probs[0][1], bin2float(stage_probs[0][1]))
            if (bin2float(stage_probs[0][0]) >= float_interval_min) and (
                    bin2float(stage_probs[0][1]) < float_interval_max):
                # The binary code is found.
                # print(stage_probs[0][0], bin2float(stage_probs[0][0]))
                # print(stage_probs[0][1], bin2float(stage_probs[0][1]))
                # print("The binary code is : ", stage_probs[0][0])
                binary_code = stage_probs[0][0]
                break
            elif (bin2float(stage_probs[1][0]) >= float_interval_min) and (
                    bin2float(stage_probs[1][1]) < float_interval_max):
                # The binary code is found.
                # print(stage_probs[1][0], bin2float(stage_probs[1][0]))
                # print(stage_probs[1][1], bin2float(stage_probs[1][1]))
                # print("The binary code is : ", stage_probs[1][0])
                binary_code = stage_probs[1][0]
                break

        if self.save_stages:
            binary_encoder.append(stage_probs)

        return binary_code, binary_encoder

    def decode(self, encoded_msg, msg_length, probability_table):
        """
        Decodes a message from a floating-point number.

        encoded_msg: The floating-point value that encodes the message.
        msg_length: Length of the message.
        probability_table: The probability table.

        Returns the decoded message.
        """

        decoder = []

        decoded_msg = []

        stage_min = Decimal(0.0)
        stage_max = Decimal(1.0)

        for idx in range(msg_length):
            stage_probs = self.process_stage(probability_table, stage_min, stage_max)

            for msg_term, value in stage_probs.items():
                if encoded_msg >= value[0] and encoded_msg <= value[1]:
                    break

            decoded_msg.append(msg_term)

            stage_min = stage_probs[msg_term][0]
            stage_max = stage_probs[msg_term][1]

            if self.save_stages:
                decoder.append(stage_probs)

        if self.save_stages:
            last_stage_probs = self.process_stage(probability_table, stage_min, stage_max)
            decoder.append(last_stage_probs)

        return decoded_msg, decoder


def float2bin(float_num, num_bits=None):
    """
    Converts a floating-point number into binary.

    float_num: The floating-point number.
    num_bits: The number of bits expected in the result. If None, then the number of bits depends on the number.

    Returns the binary representation of the number.
    """

    float_num = str(float_num)
    if float_num.find(".") == -1:
        # No decimals in the floating-point number.
        integers = float_num
        decimals = ""
    else:
        integers, decimals = float_num.split(".")
    decimals = "0." + decimals
    decimals = Decimal(decimals)
    integers = int(integers)

    result = ""
    num_used_bits = 0
    while True:
        mul = decimals * 2
        int_part = int(mul)
        result = result + str(int_part)
        num_used_bits = num_used_bits + 1

        decimals = mul - int(mul)
        if type(num_bits) is type(None):
            if decimals == 0:
                break
        elif num_used_bits >= num_bits:
            break
    if type(num_bits) is type(None):
        pass
    elif len(result) < num_bits:
        num_remaining_bits = num_bits - len(result)
        result = result + "0" * num_remaining_bits

    integers_bin = bin(integers)[2:]
    result = str(integers_bin) + "." + str(result)
    return result


def bin2float(bin_num):
    """
    Converts a binary number to a floating-point number.

    bin_num: The binary number as a string.

    Returns the floating-point representation.
    """

    if bin_num.find(".") == -1:
        # No decimals in the binary number.
        integers = bin_num
        decimals = ""
    else:
        integers, decimals = bin_num.split(".")
    result = Decimal(0.0)

    # Working with integers.
    for idx, bit in enumerate(integers):
        if bit == "0":
            continue
        mul = 2 ** idx
        result = result + Decimal(mul)

    # Working with decimals.
    for idx, bit in enumerate(decimals):
        if bit == "0":
            continue
        mul = Decimal(1.0) / Decimal((2 ** (idx + 1)))
        result = result + mul
    return result
```

### example.py

```python
import pyae

# Example for encoding a simple text message using the PyAE module.
# This example only returns the floating-point value that encodes the message.
# Check the example_binary.py to return the binary code of the floating-point value.

frequency_table = {"a": 2,
                   "b": 7,
                   "c": 1}

AE = pyae.ArithmeticEncoding(frequency_table=frequency_table,
                            save_stages=True)

original_msg = "abca"
print("Original Message: {msg}".format(msg=original_msg))

encoded_msg, encoder , interval_min_value, interval_max_value = AE.encode(msg=original_msg,
                                                                          probability_table=AE.probability_table)
print("Encoded Message: {msg}".format(msg=encoded_msg))

decoded_msg, decoder = AE.decode(encoded_msg=encoded_msg,
                                 msg_length=len(original_msg),
                                 probability_table=AE.probability_table)
print("Decoded Message: {msg}".format(msg=decoded_msg))

decoded_msg = "".join(decoded_msg)
print("Message Decoded Successfully? {result}".format(result=original_msg == decoded_msg))
```
## 参考资料

[什么是算术编码](https://zhuanlan.zhihu.com/p/390684936)

[GitHub-Arithmetic Encoding Python](https://github.com/ahmedfgad/ArithmeticEncodingPython)

[Arithmetic Encoding and Decoding with Python](https://mohammad-gholizadeh.github.io/Post-3-Arithmetic%20with%20Python.html)

[Arithmetic coding in Python](https://tommyodland.com/articles/2024/arithmetic-coding-in-python/)













