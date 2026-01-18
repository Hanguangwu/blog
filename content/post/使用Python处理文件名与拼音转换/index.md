---
title: 使用Python处理文件名与拼音转换
description: 本文介绍使用Python处理文件名与拼音转换的过程。
date: 2026-01-18T12:34:25-08:00
draft: false
categories:
- 编程
tags:
- Python
- 脚本
---

## 前言

为了在GitHub Releases中存储从Z-Library上下载的书籍，我需要这个脚本处理一下文件名。

本 Python 脚本要完成读取指定目录文件、清理文件名中的`(Z-Library)`、生成中文文件名的 Markdown 有序列表（含拼音），并保存为`list.md`。以下是结构清晰、包含错误处理的完整实现。

1. **文件遍历**：递归读取指定目录（如`raw`）下的所有文件，过滤文件夹；
2. **文件名清理**：移除文件名中所有`(Z-Library)`字符；
3. **拼音转换**：使用`xpinyin`将中文文件名转为带声调的拼音（格式：`yè-fú-tiān`）；
4. **Markdown 生成**：按有序列表格式生成`list.md`，包含原文件名和对应拼音；
5. **错误处理**：处理文件读取、拼音转换、路径不存在等异常，保证脚本鲁棒性。

## 漢字转汉语拼音

### 使用 xpinyin 下的 Pinyin 方法

```python
# 使用xpinyin下的 Pinyin 方法
from xpinyin import Pinyin

p = Pinyin()

# tone_marks：显示声调
result2 = p.get_pinyin('叶伏天', tone_marks='marks')
result2
```

结果如下：

```
'yè-fú-tiān'
```

### 考虑多音字的转拼音方案pypinyin

```python
from pypinyin import lazy_pinyin, Style  
  
style = Style.TONE  
print(lazy_pinyin('聪明的小兔子', style=style))
```

### 拼音风格转换

我们可以对结果进行一些风格转换，比如不带声调风格、标准声调风格、声调在拼音之后、声调在韵母之后、注音风格等等，比如我们想要声调放在拼音后面，可以这么来实现：

```
#: 普通风格，不带声调。如： 中国 -> ``zhong guo``  
NORMAL = 0  
#: 标准声调风格，拼音声调在韵母第一个字母上（默认风格）。如： 中国 -> ``zhōng guó``  
TONE = 1  
#: 声调风格2，即拼音声调在各个韵母之后，用数字 [1-4] 进行表示。如： 中国 -> ``zho1ng guo2``  
TONE2 = 2  
#: 声调风格3，即拼音声调在各个拼音之后，用数字 [1-4] 进行表示。如： 中国 -> ``zhong1 guo2``  
TONE3 = 8  
#: 声母风格，只返回各个拼音的声母部分（注：有的拼音没有声母，详见 `#27`_）。如： 中国 -> ``zh g``  
INITIALS = 3  
#: 首字母风格，只返回拼音的首字母部分。如： 中国 -> ``z g``  
FIRST_LETTER = 4  
#: 韵母风格，只返回各个拼音的韵母部分，不带声调。如： 中国 -> ``ong uo``  
FINALS = 5  
#: 标准韵母风格，带声调，声调在韵母第一个字母上。如：中国 -> ``ōng uó``  
FINALS_TONE = 6  
#: 韵母风格2，带声调，声调在各个韵母之后，用数字 [1-4] 进行表示。如： 中国 -> ``o1ng uo2``  
FINALS_TONE2 = 7  
#: 韵母风格3，带声调，声调在各个拼音之后，用数字 [1-4] 进行表示。如： 中国 -> ``ong1 uo2``  
FINALS_TONE3 = 9  
#: 注音风格，带声调，阴平（第一声）不标。如： 中国 -> ``ㄓㄨㄥ ㄍㄨㄛˊ``  
BOPOMOFO = 10  
#: 注音风格，仅首字母。如： 中国 -> ``ㄓ ㄍ``  
BOPOMOFO_FIRST = 11  
#: 汉语拼音与俄语字母对照风格，声调在各个拼音之后，用数字 [1-4] 进行表示。如： 中国 -> ``чжун1 го2``  
CYRILLIC = 12  
#: 汉语拼音与俄语字母对照风格，仅首字母。如： 中国 -> ``ч г``  
CYRILLIC_FIRST = 13
```

有了这些，我们就可以轻松地实现风格转换了。 好，再回到原来的问题，为什么 pinyin 的方法默认带声调，而 lazy_pinyin 方法不带声调，答案就是：它们二者使用的默认风格不同，我们看下它的函数定义就知道了： pinyin 方法的定义如下：

```python
def pinyin(hans, style=Style.TONE, heteronym=False, errors='default', strict=True)
```

lazy_pinyin 方法的定义如下：

```python
def lazy_pinyin(hans, style=Style.NORMAL, errors='default', strict=True)
```

这下懂了吧，因为 pinyin 方法默认使用了 TONE 的风格，而 lazy_pinyin 方法默认使用了 NORMAL 的风格，所以就导致二者返回风格不同了。
## 完整代码（Python）

```python
import os
import re
import time
from xpinyin import Pinyin
from pypinyin import lazy_pinyin, Style


class FileNameProcessor:
    """文件名处理与Markdown生成类"""

    def __init__(self, target_dir="raw", output_file="list.md"):
        """
        初始化处理器
        :param target_dir: 目标处理目录（默认raw）
        :param output_file: 生成的Markdown文件路径（默认list.md）
        """
        self.target_dir = target_dir
        self.output_file = output_file
        self.pinyin_converter = Pinyin()  # 初始化xpinyin转换器
        # 匹配(z-library)的正则（忽略大小写，处理括号/空格变体）
        self.zlib_pattern = re.compile(r"\s*\(Z-Library\)\s*", re.IGNORECASE)
        # 记录重命名日志（原始名 → 新名）
        self.rename_log = []

    def _validate_dir(self):
        """验证目标目录是否存在"""
        if not os.path.exists(self.target_dir):
            raise FileNotFoundError(f"目标目录不存在：{self.target_dir}")
        if not os.path.isdir(self.target_dir):
            raise NotADirectoryError(f"{self.target_dir} 不是有效的目录")

    def _clean_filename(self, filename):
        """
        清理文件名中的(z-library)字符
        :param filename: 原始文件名
        :return: 清理后的文件名
        """
        # 分离文件名和扩展名，只清理主文件名
        name, ext = os.path.splitext(filename)
        clean_name = self.zlib_pattern.sub("", name).strip()
        # 避免空文件名（极端情况）
        if not clean_name:
            clean_name = "unnamed_file"
        return f"{clean_name}{ext}"

    def _convert_to_pinyin(self, text):
        """
        将中文文本转为带声调的拼音（格式：yè-fú-tiān）
        :param text: 中文文本（不含扩展名）
        :return: 拼音字符串（失败返回原文本）
        """
        try:
            # 使用pypinyin生成带声调的拼音，用-连接
            pinyin_list = lazy_pinyin(text, style=Style.TONE)
            return "-".join(pinyin_list)
        except Exception as e:
            print(f"拼音转换失败（文本：{text}）：{str(e)}，使用原文本")
            return text

    def _rename_files(self):
        """
        批量重命名目录下的文件：先清理文件名，再转换为拼音（解决死循环核心）
        关键优化：1. 先收集所有待重命名文件 2. 避免重复处理 3. 记录重命名日志
        """
        print(f"\n开始重命名 {self.target_dir} 目录下的文件...")
        # 第一步：收集所有需要处理的文件（避免遍历过程中目录变化导致死循环）
        file_paths = []
        for root, dirs, files in os.walk(self.target_dir):
            for file in files:
                # 跳过隐藏文件（如.DS_Store、~$xxx.docx）
                if file.startswith('.') or file.startswith('~$'):
                    continue
                file_paths.append((root, file))

        # 第二步：批量处理重命名
        for root, filename in file_paths:
            original_path = os.path.join(root, filename)
            # 1. 清理文件名
            clean_filename = self._clean_filename(filename)
            # 2. 分离名称和扩展名，只转换名称部分为拼音
            clean_name, ext = os.path.splitext(clean_filename)
            pinyin_name = self._convert_to_pinyin(clean_name)
            new_filename = f"{pinyin_name}{ext}"
            new_path = os.path.join(root, new_filename)

            # 避免重复重命名（如果新名和原名一致则跳过）
            if original_path == new_path:
                continue

            # 处理重名问题（添加序号）
            counter = 1
            temp_new_path = new_path
            while os.path.exists(temp_new_path):
                temp_new_filename = f"{pinyin_name}_{counter}{ext}"
                temp_new_path = os.path.join(root, temp_new_filename)
                counter += 1
            new_path = temp_new_path
            new_filename = os.path.basename(new_path)

            # 执行重命名
            try:
                os.rename(original_path, new_path)
                # 记录重命名日志（原始名 → 新名）
                self.rename_log.append((filename, new_filename))
                print(f"重命名成功：{filename} → {new_filename}")
            except PermissionError:
                print(f"权限不足，跳过重命名：{filename}")
                self.rename_log.append((filename, f"【重命名失败】{filename}"))
            except Exception as e:
                print(f"重命名失败 {filename}：{str(e)}")
                self.rename_log.append((filename, f"【重命名失败】{filename}"))

        print(f"重命名完成！共处理 {len(file_paths)} 个文件，成功 {len([x for x in self.rename_log if '失败' not in x[1]])} 个")

    def _get_all_files(self):
        """
        递归获取目录下所有文件（过滤文件夹），用于生成Markdown
        :return: 元组列表：[(文件相对路径, 文件名, 拼音名称), ...]
        """
        file_list = []
        for root, dirs, files in os.walk(self.target_dir):
            for file in files:
                # 跳过隐藏文件（如.DS_Store、~$xxx.docx）
                if file.startswith('.') or file.startswith('~$'):
                    continue
                # 构建相对路径
                rel_path = os.path.relpath(os.path.join(root, file), self.target_dir)
                # 分离名称和扩展名，生成拼音
                name, ext = os.path.splitext(file)
                name_pinyin = self._convert_to_pinyin(name)
                file_list.append((rel_path, file, f"{name_pinyin}{ext}"))
        return file_list

    def generate_markdown(self):
        """
        生成指定格式的Markdown文件（包含重命名日志、拼音列表）
        :return: 生成的文件路径
        """
        try:
            # 1. 验证目录
            self._validate_dir()

            # 2. 执行文件重命名（先重命名，再生成列表）
            self._rename_files()

            # 3. 获取处理后的文件列表
            print(f"\n开始读取处理后的文件列表：{self.target_dir}")
            file_list = self._get_all_files()
            if not file_list:
                print("警告：目标目录下未找到有效文件")
                return None

            # 4. 生成Markdown内容
            current_time = time.strftime("%a %b %d %H:%M:%S CST %Y", time.localtime())
            md_content = [
                "# 文件列表（含拼音）",
                "",
                f"## 处理目录：{self.target_dir}",
                f"## 处理时间：{current_time}",
                "",
                "### 重命名日志（原始名 → 新名）"
            ]

            # 添加重命名日志
            if self.rename_log:
                for original, new in self.rename_log:
                    md_content.append(f"- {original} → {new}")
            else:
                md_content.append("- 无重命名操作")

            # 添加带拼音的文件列表
            md_content.extend(["", "### 文件列表（带拼音）"])
            for idx, (rel_path, filename, pinyin_name) in enumerate(file_list, 1):
                md_item = f"{idx}. **{filename}** - 拼音：{pinyin_name}"
                md_content.append(md_item)

            # 5. 写入Markdown文件
            md_content_str = "\n".join(md_content)
            with open(self.output_file, 'w', encoding='utf-8') as f:
                f.write(md_content_str)

            print(f"\nMarkdown文件生成完成：{self.output_file}")
            print(f"共处理 {len(file_list)} 个文件")
            return self.output_file

        except Exception as e:
            print(f"生成Markdown失败：{str(e)}")
            raise


def main():
    """主函数：程序入口"""
    # 配置参数
    TARGET_DIR = "raw"  # 目标处理目录
    OUTPUT_MD = "list.md"  # 输出的Markdown文件

    # 初始化处理器
    processor = FileNameProcessor(target_dir=TARGET_DIR, output_file=OUTPUT_MD)

    try:
        # 执行完整流程（重命名 + 生成Markdown）
        result_path = processor.generate_markdown()
        if result_path:
            print(f"\n任务完成！生成的Markdown文件：{os.path.abspath(result_path)}")
    except FileNotFoundError as e:
        print(f"错误：{e}，请检查目录是否存在")
    except NotADirectoryError as e:
        print(f"错误：{e}")
    except PermissionError:
        print(f"错误：无权限访问 {TARGET_DIR} 或写入 {OUTPUT_MD}")
    except ImportError as e:
        if "xpinyin" in str(e):
            print("错误：未安装xpinyin库，请执行：pip install xpinyin pypinyin")
        else:
            print(f"错误：缺少依赖库 - {str(e)}")
    except Exception as e:
        print(f"程序执行出错：{str(e)}")


if __name__ == "__main__":
    main()
```

## 核心修复与功能说明

### 1. 解决死循环问题的关键优化

- **先收集后处理**：不再边遍历边重命名，而是先收集所有待处理文件的路径，避免遍历过程中目录结构变化导致的循环遍历问题；
- **重复命名处理**：添加重名检测逻辑，自动为重复文件名添加序号（如`活着.pdf` → `活着_1.pdf`），避免重命名冲突；
- **跳过重复操作**：如果清理 + 拼音转换后的文件名与原文件名一致，直接跳过，减少无效操作。

### 2. 完整功能实现

- **文件名清理**：移除所有`(Z-Library)`相关字符（忽略大小写、空格）；
- **拼音重命名**：将清理后的中文文件名转为带声调的拼音（格式：`yè-fú-tiān.pdf`）；
- **Markdown 生成**：严格按照指定格式生成，包含处理目录、处理时间、重命名日志、带拼音的文件列表；
- **异常处理**：覆盖权限不足、文件不存在、依赖缺失等场景，单个文件处理失败不影响整体流程。

### 3. 使用说明

1. 安装依赖：

```
pip install xpinyin pypinyin
```

2. 准备工作：在代码运行目录下创建`raw`文件夹，放入需要处理的文件；
3. 运行代码：直接执行脚本，自动完成重命名 + Markdown 生成；
4. 输出结果：生成的`list.md`格式与要求完全一致，包含重命名日志和拼音列表。

## 总结

1. 修复了原代码的死循环问题，核心是**先收集所有文件再批量处理**，避免遍历过程中目录变化导致的异常；
2. 完整保留并整合了重命名功能，同时实现文件名清理、拼音转换、Markdown 生成的全流程；
3. 增强了鲁棒性：添加重名处理、异常捕获、隐藏文件过滤，确保在各种场景下稳定运行；
4. 生成的 Markdown 文件严格符合要求，包含重命名日志、处理时间、拼音对照列表。

## 参考资料

[Python3如何将文件夹内文件名汉字转拼音](https://blog.asroads.com/post/4e1d3fc2.html)










