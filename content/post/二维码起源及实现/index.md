---
title: 从黑白方块到信息世界：二维码的原理、历史与 Python 实战
description: 本文从二维码的历史讲起，解释它如何把链接、文本及其他数据编码成黑白方块，再通过 Python 实现二维码的生成、识别、纠错和自定义美化。
date: 2026-10-23T14:34:25-08:00
draft: false
categories:
- 编程
- 技术科普
- AIGC
tags:
- QR Code
- 编码原理
- Python
---

# 从黑白方块到信息世界：二维码的原理、历史与 Python 实战

## 一、二维码是什么？

二维码（QR Code，Quick Response Code，快速响应码）是一种二维条码。它通过在平面上排列深色和浅色模块，将文本、网址、数字以及其他二进制数据编码为机器可以识别的图案。

我们每天都会遇到二维码：

- 扫描二维码，打开一个网页。
    
- 扫描付款码，发起支付。
    
- 扫描 Wi-Fi 二维码，自动填写网络名称和密码。
    
- 扫描电子名片，导入联系人。
    
- 扫描产品标签，查询商品信息。
    

这些二维码看起来很相似，但它们存储的内容可能完全不同。

例如，一个二维码的实际内容可以是：

```
https://example.com
```

也可以是：

```
你好，世界！
```

还可以是：

```
WIFI:T:WPA;S:MyHomeWiFi;P:MyPassword123;;
```

甚至可以是经过特定格式编码的联系人信息、设备配置或一段二进制数据。

最重要的认识：二维码本身通常不是一个网页，也不是一个数据库，而是数据的一种机器可读表示形式。

如果二维码存储的是网址，扫码后打开网页的过程通常是：

1. 摄像头获取二维码图像。
    
2. 解码程序定位并识别二维码。
    
3. 解析二维码中的数据，得到网址。
    
4. 手机浏览器根据网址访问互联网。
    

其中，第 1～3 步属于二维码识别与解码；第 4 步才是网络访问。

二维码也可以直接存储文本而不访问互联网。因此，扫码后是否需要联网，取决于二维码中存储的内容以及使用它的应用程序。

## 二、二维码的起源：一项来自日本汽车工业的发明

![QRコードを生み出した技術者、原昌宏。ヒントは囲碁だった【世界が尊敬する日本人】 | ニューズウィーク日本版 オフィシャルサイト](https://images.openai.com/static-rsc-4/dKk2Z9VkjKYsb-SaIRLC1ULrbd5_r8KvuF_bTIP9ow973ASONQ2Hr9OBirIqj4xvdnQtv3KmdyU1o3LdcR-5Qy5oaAv9D8gCDg1jDf2qNq-oVqPhYbm-9RNNKB0WikNpPrhmUfY8XekN2mFsJEIlrYsyX0vTJoETZK-kvQrn8UQ?purpose=inline)

[newsweekjapan.jp](https://www.newsweekjapan.jp/stories/technology/2021/08/post-96818.php)

二维码并不是互联网时代才出现的技术。我们今天熟悉的 QR Code，诞生于日本制造业对生产效率的实际需求。

### 2.1 从条形码的局限说起

20 世纪后期，条形码已经广泛用于零售、物流和工业生产。传统条形码通过不同宽度的线条及间隔来表示数据，读取设备通常沿着一个方向扫描。

但随着制造业需要管理的零部件、型号和生产记录越来越多，传统条形码的容量和读取效率开始显得不足。

当时的日本汽车零部件生产现场，需要处理大量产品信息。开发人员希望设计一种编码，能够：

- 在有限空间中存储更多信息。
    
- 同时支持数字、字母和日文字符。
    
- 能够快速识别，减少重复扫描。
    
- 即使部分区域受到污损，也尽可能恢复数据。
    

这些需求推动了新型二维编码的研发。

### 2.2 1994 年：QR Code 正式问世

1992 年前后，DENSO 的工程师开始探索新的二维编码方案。以原昌宏（Masahiro Hara）为代表的开发团队，最终设计出了今天广泛使用的 QR Code。

1994 年，DENSO WAVE（当时属于 DENSO 的一个事业部门）正式发布 QR Code。

QR 是英文 Quick Response 的缩写，强调快速读取和响应。

二维码设计中的一个关键创新，是三个角上的大型定位图案。

![What is QR Code? 2026 Guide to Uses and Security](https://images.openai.com/static-rsc-4/UXCL5hMDPZQgQ2LeXZhI6cqLlOi6tdJDy0St5r5dRDTX4swC3DfZW8zYg4-h_qqzsRqJ5V6FIEVodMZ1Gt21Pvt7a3fBaP_L2ZJx_s22kXHpmZvxJ4AgisU_Ih-XcG6kG6y8o5eOLsWpZMqb29kBawjYWYP-CzO9CwjRarSigzE?purpose=inline)

![](https://cdn.jsdelivr.net/gh/Hanguangwu/MyImageBed01/img/20261023192914439.png)

![QR code's structure 1. Finder Pattern: It is detecting position of QR... | Download Scientific Diagram](https://images.openai.com/static-rsc-4/gMZM32uziCQVSzsERzpiypr6zejuNT5VrUOmqLTwK5pHzFPnSu-nozvhF3_iklycR61qcg25xJXRrkdbI_BCbbRbJA2ypcPkghPKl4aRsgsJWM0xz48Z2IfPQbm4P2W9CElTcXo6OlPtOuXd9ZFhluU0cWWaNFoxjtSlAhtZT4M?purpose=inline)

这些图案帮助扫描器判断二维码的位置、方向和尺度，使设备即使在二维码旋转或倾斜时，也能更可靠地识别它。

另一个重要特点是纠错能力：二维码不只记录原始数据，还包含用于恢复数据的冗余信息。即使一部分图案被污损，解码器仍可能恢复原始内容。

### 2.3 从工厂走向全世界

二维码最初主要服务于汽车制造和生产管理。随着标准化和手机摄像头识别能力的发展，它逐渐进入商品追踪、票务、广告、支付和个人信息交换等场景。

2000 年，QR Code 被纳入国际标准 ISO/IEC 18004，进一步促进了它的广泛应用。

二维码的成功不仅在于技术本身，也在于它是一种可广泛使用的开放编码技术。任何开发者都可以利用兼容的规范和工具生成、读取二维码，而不必依赖某家公司的专有扫码系统。

## 三、二维码究竟如何存储数据？

这是理解二维码最重要的一部分。

### 3.1 黑白方块如何表示信息？

二维码由许多小方格组成，这些方格称为模块（module）。在常见的黑白二维码中，每个模块呈现为深色或浅色。

但需要注意：一个黑色方块不一定代表数字 1，一个白色方块也不一定代表数字 0。

这是因为二维码的图案中包含多种结构：

- 定位图案：帮助识别二维码的位置和方向。
    
- 校正与同步图案：帮助读取器判断模块排列、倾斜和尺度。
    
- 格式及版本信息：说明如何解读二维码。
    
- 数据区域：承载编码后的数据。
    
- 纠错信息：帮助恢复因污损或遮挡而丢失的数据。
    

因此，不能简单地把整张二维码当成一串从左到右、从上到下的二进制数字直接阅读。

### 3.2 数据编码的完整流程

以生成一个网址二维码为例，整个过程可以抽象成以下流水线。

① 原始数据

[https://example.com](https://example.com/?utm_source=chatgpt.com)

② 字符编码与数据模式选择

将文本转换成适合 QR Code 规则的位序列

③ 生成纠错数据

使用 Reed–Solomon 纠错编码增加冗余信息

④ 构建二维码矩阵

放置定位图案、格式信息、数据模块等

⑤ 输出图像

PNG、SVG 或其他适合显示和打印的格式

不同编码模式的容量不同。标准 QR Code 可以使用数字模式、字母数字模式、字节模式和汉字模式等来存储数据。实际容量还取决于二维码版本、纠错等级以及数据内容。

### 3.3 二维码可以存储哪些类型的数据？

|数据类型|存储示例|扫码后的典型行为|
|---|---|---|
|网址|`https://example.com`|浏览器打开网页|
|普通文本|`你好，世界！`|显示文本|
|联系人|`BEGIN:VCARD...`|提示添加联系人|
|Wi-Fi 配置|`WIFI:T:WPA;S:Home;P:...;;`|提示连接网络|
|邮件信息|`mailto:hello@example.com`|打开邮件应用|
|电话号码|`tel:+8613800000000`|提示拨打电话|
|二进制数据|文件片段或应用专用数据|需要对应程序解读|

表中的内容并不都属于 QR Code 标准自带的特殊功能。例如，Wi-Fi 配置和联系人信息通常是按照约定的文本格式存入二维码，再由手机识别这些格式。

因此，二维码负责存储和传递数据，应用程序负责理解数据的具体含义。

### 3.4 二维码最多能存储多少数据？

标准 QR Code 的最大容量取决于编码模式。以最高版本、最低纠错等级等适合获得最大容量的条件为例，官方资料列出的典型上限如下。

|编码模式|最大容量（约）|
|---|---|
|纯数字|7,089 个数字字符|
|字母数字|4,296 个字符|
|8-bit 字节|2,953 个字节|
|日文汉字模式|1,817 个字符|

这些是特定条件下的理论上限，并不意味着任何二维码都能存储这么多内容。

例如，中文文本通常需要按 UTF-8 等编码转换成多个字节，一个汉字往往占用 3 个 UTF-8 字节。二维码版本越大，图案中的模块越多，打印和识别时对清晰度的要求也可能更高。

如果希望分享一篇很长的文章，通常存储文章网址比把全文塞进二维码更方便。

## 四、二维码为什么被遮挡后仍然可以识别？

二维码最值得注意的技术之一，是纠错编码。

### 4.1 Reed–Solomon 纠错

QR Code 使用 Reed–Solomon（里德–所罗门）纠错码，在原始数据之外添加冗余信息。

可以把它理解为给数据增加一份有数学规律的校验信息。即使部分数据受到破坏，解码器仍然可以根据剩余数据和纠错信息，恢复一定范围内的错误。

二维码通常提供四种纠错等级：

|等级|常见标记|大致可恢复的码字比例|特点|
|---|---|---|---|
|L|Low|约 7%|容量相对较大|
|M|Medium|约 15%|常见的折中选择|
|Q|Quartile|约 25%|更强的纠错能力|
|H|High|约 30%|纠错能力更强|

这些百分比是标准资料给出的近似值，并不是二维码图像允许随意遮挡的面积比例。实际可恢复程度取决于损坏的位置、分布和解码条件。

### 4.2 为什么二维码中间可以放一个 Logo？

![브런치](https://images.openai.com/static-rsc-4/HV1lbgBHmdVIR2A8LpDCQS12oWF6ONMjlWB8pm-hL-dBf4-ZLnE7AlYsUI2b0Cifcx9rT5SEQ2wo0as_ibvKYz0P_y7YyDHtn-vjqCSu7mhpEr4OB7EgDVa2lnpgj_gHAyEVj3XmR460u8Hzdra3mvo9WDVAyIK85CpWi2z0nvo?purpose=inline)

有些品牌二维码会在中央放置 Logo。这并不是因为 Logo 本身是二维码的一部分，而是因为纠错能力允许部分数据受损时仍有机会完成解码。

但这不意味着可以随意覆盖二维码：

- Logo 可能遮住数据模块或关键定位结构。
    
- 图像压缩、低分辨率和印刷误差也会消耗纠错余量。
    
- 即使设置 H 级纠错，也无法保证任意遮挡下都能识别。

实用原则是：尽量保持三个角的定位图案完整，Logo 尺寸适中，并使用真实手机或扫码器反复测试。

## 五、使用 Python 自制二维码

接下来，我们从零搭建一个可以在本地运行的二维码生成程序。

整个过程不需要连接二维码生成网站。生成的内容可以保留在本地电脑上，适合制作个人博客链接、项目主页、联系方式和 Wi-Fi 配置码。

### 5.1 安装 Python 依赖

建议使用 Python 3.10 或更新版本。在 Windows PowerShell 或 WSL2 终端中执行：

```
python -m venv .venv
```

激活虚拟环境。

Windows PowerShell：

```
.\.venv\Scripts\Activate.ps1
```

Linux / WSL2：

```
source .venv/bin/activate
```

安装二维码生成和图像处理库：

```
python -m pip install "qrcode[pil]" pillow
```

如果还希望用 Python 直接识别二维码，可以额外安装 OpenCV：

```
python -m pip install opencv-python
```

其中：

- `qrcode`：生成符合 QR Code 规范的二维码。
    
- `Pillow`：处理和保存图像。
    
- `opencv-python`：读取图像、检测二维码并解码内容。
    

### 5.2 示例一：生成最简单的网址二维码

新建文件 `generate_qr.py`：

```python
import qrcode

# 二维码中真正存储的内容
data = "https://example.com"

# 生成二维码
qr = qrcode.QRCode(
    version=None,  # 自动选择合适的二维码版本
    error_correction=qrcode.constants.ERROR_CORRECT_M,
    box_size=10,   # 每个模块对应的像素大小
    border=4,      # 保留四个模块宽度的边框
)

qr.add_data(data)
qr.make(fit=True)

# 转换为黑白图像并保存
img = qr.make_image(
    fill_color="black",
    back_color="white",
)

img.save("my_qr.png")

print("二维码已保存为 my_qr.png")
```

运行：

```
python generate_qr.py
```

程序会在当前目录生成 `my_qr.png`。

你可以用手机扫描图片，检查是否得到 `https://example.com`。

这里有几个值得理解的参数：

- `version=None`：让库根据数据量自动选择版本。
    
- `ERROR_CORRECT_M`：使用 M 级纠错。
    
- `box_size=10`：每个模块绘制成约 10 × 10 像素的方块。
    
- `border=4`：保留四个模块宽度的静区，帮助扫码器区分二维码与周围背景。
    

`qr.add_data()` 接收的是待编码内容。对于普通网址，不需要自己手动将每个字符转换为二进制，也不需要自行绘制定位图案；这些工作由二维码库完成。

### 5.3 示例二：自定义颜色、尺寸和纠错等级

二维码不一定只能是纯黑白色。只要对比度足够，通常也可以使用其他前景色和背景色。

新建 `custom_qr.py`：

```python
import qrcode
from qrcode.constants import (
    ERROR_CORRECT_H,
)

data = "https://example.com/my-project"

qr = qrcode.QRCode(
    version=None,
    error_correction=ERROR_CORRECT_H,
    box_size=12,
    border=4,
)

qr.add_data(data)
qr.make(fit=True)

img = qr.make_image(
    fill_color="#17324D",
    back_color="#FFFFFF",
)

img.save("custom_qr.png")

print("已生成 custom_qr.png")
```

这里使用 H 级纠错，适合希望后续尝试添加小型 Logo 的场景，但它会带来容量方面的权衡：相同版本下，可用于原始数据的空间会受到纠错数据的影响。

实际应用中，深色前景与浅色背景通常更可靠。不要随意使用低对比度配色、渐变、复杂背景或过度圆角化的模块。

### 5.4 示例三：生成包含中文的二维码

二维码也可以直接存储中文文本。

```python
import qrcode

text = """你好，世界！

这是我使用 Python 生成的二维码。
它可以直接存储文本，不需要访问网站。"""

qr = qrcode.QRCode(
    version=None,
    error_correction=qrcode.constants.ERROR_CORRECT_M,
    box_size=10,
    border=4,
)

qr.add_data(text)
qr.make(fit=True)

img = qr.make_image(
    fill_color="black",
    back_color="white",
)

img.save("chinese_text_qr.png")

print("已生成 chinese_text_qr.png")
```

扫码后，兼容的扫码器可以直接显示中文内容。

这里需要区分两件事：

1. 二维码的数据编码：将字符转换成二维码协议规定的位序列。
    
2. 字符集解码：将恢复出来的字节还原成中文字符串。
    

如果你要存储的是中文、Emoji 或其他 Unicode 字符，使用成熟的二维码库通常比手动拼接二进制数据可靠得多。

### 5.5 示例四：生成 Wi-Fi 二维码

手机可以通过约定的文本格式读取 Wi-Fi 名称、加密方式和密码。

例如：

```python
import qrcode

ssid = "MyHomeWiFi"
password = "MyPassword123"

wifi_data = (
    f"WIFI:T:WPA;S:{ssid};P:{password};;"
)

qr = qrcode.QRCode(
    error_correction=qrcode.constants.ERROR_CORRECT_M,
    box_size=10,
    border=4,
)

qr.add_data(wifi_data)
qr.make(fit=True)

qr.make_image(
    fill_color="black",
    back_color="white",
).save("wifi_qr.png")

print("已生成 wifi_qr.png")
```

这类格式通常适用于常见的 WPA/WPA2 等 Wi-Fi 配置，但具体识别行为取决于手机系统和扫码应用。

安全提醒： Wi-Fi 二维码可能直接包含明文密码。不要把它发布到公开网站、社交媒体或容易被他人获取的场所；生成的图片也应按敏感信息妥善保管。

如果 SSID 或密码中包含分号、反斜杠等特殊字符，还需要根据对应的 Wi-Fi 二维码格式规则进行转义，不能总是直接拼接字符串。

### 5.6 示例五：读取二维码并还原原始内容

只会生成二维码还不够。我们还可以用 Python 验证二维码能否被读取，以及解码后是否与原始内容一致。

新建 `read_qr.py`：

```python
import cv2

image_path = "my_qr.png"

# 读取二维码图片
image = cv2.imread(image_path)

if image is None:
    raise FileNotFoundError(
        f"找不到图片文件：{image_path}"
    )

# 创建二维码检测器
detector = cv2.QRCodeDetector()

# 检测并解码
data, points, _ = detector.detectAndDecode(image)

if data:
    print("二维码解码成功！")
    print("原始内容：", data)
else:
    print("未能识别二维码。")
```

运行：

```
python read_qr.py
```

如果读取的是前面生成的网址二维码，终端应该输出类似结果：

```
二维码解码成功！
原始内容： https://example.com
```

这里使用 OpenCV 的 `QRCodeDetector`，它负责定位二维码、识别图案并解码数据。

注意：如果二维码中存储的是网址，OpenCV 通常只返回网址字符串，不会自动访问网页。如果二维码存储的是 Wi-Fi 配置，也不会自动替你连接网络。

### 5.7 示例六：把生成、保存和验证整合起来

实际项目中，我们可以把生成和验证写在同一个脚本里，减少手动检查的工作。

```python
from pathlib import Path

import cv2
import qrcode
from qrcode.constants import ERROR_CORRECT_M


def generate_qr(data: str, output_path: str) -> None:
    """生成二维码并保存为 PNG 图片。"""
    qr = qrcode.QRCode(
        version=None,
        error_correction=ERROR_CORRECT_M,
        box_size=10,
        border=4,
    )

    qr.add_data(data)
    qr.make(fit=True)

    img = qr.make_image(
        fill_color="black",
        back_color="white",
    )

    img.save(output_path)


def decode_qr(image_path: str) -> str:
    """读取二维码图片并返回解码内容。"""
    image = cv2.imread(image_path)

    if image is None:
        raise FileNotFoundError(image_path)

    detector = cv2.QRCodeDetector()
    data, points, _ = detector.detectAndDecode(image)

    if not data:
        raise ValueError("二维码解码失败")

    return data


def main():
    original_data = "https://example.com/my-project"
    output_path = "my_project_qr.png"

    generate_qr(original_data, output_path)

    decoded_data = decode_qr(output_path)

    print(f"图片位置：{Path(output_path).resolve()}")
    print(f"原始数据：{original_data}")
    print(f"解码数据：{decoded_data}")

    if decoded_data == original_data:
        print("验证通过：解码结果与原始数据一致。")
    else:
        print("验证失败：解码结果不一致。")


if __name__ == "__main__":
    main()
```

这个小程序实现了一个完整的闭环：

>输入原始数据（网址、文本或其他适合编码的字符串）——>生成二维码（编码、纠错、构建矩阵、保存图像）——>重新读取（通过图像识别恢复编码内容）——>自动校验（比较原始数据与解码数据是否一致）

## 六、进一步理解：二维码不等于链接，链接也不等于网页

二维码的应用中，有三个经常被混淆的概念。

|概念|实际含义|
|---|---|
|二维码|数据的图形化编码|
|URL（网址）|标识网络资源的地址|
|网页或服务|URL 所指向的实际资源|

例如，你可以生成一个存储 `https://example.com` 的二维码，但这不代表二维码内部已经保存了该网站的全部 HTML、图片和数据库。

它通常只存储网址文本。手机扫描后再通过网络请求取得网页。

反过来，如果你把一段 500 字的文章直接编码进二维码，扫码器就可以在不访问网站的情况下恢复这段文章，前提是数据量在容量限制内，且扫码器能正确解码对应内容。

这也解释了为什么二维码生成网站经常会区分两类二维码：

- 静态二维码：直接存储最终内容。内容通常无法在不重新生成二维码的情况下修改。
    
- 动态二维码：存储一个可控的短链接或重定向地址。服务端可以改变它最终指向的资源。
    

所谓动态二维码并不是一种会自动变化的特殊图案。它的动态性通常来自二维码背后的服务器服务。

## 七、二维码有哪些安全风险？

二维码本身并不等于安全或危险。风险主要取决于其中存储的数据，以及扫码之后发生的操作。

### 7.1 恶意链接

攻击者可以把钓鱼网站的网址编码成二维码，诱导用户输入账号、密码或支付信息。

防范方式：扫描后先检查域名，不要因为二维码看起来正规就直接相信其内容。

### 7.2 敏感信息泄露

二维码可能直接存储 Wi-Fi 密码、联系人信息、个人资料，甚至某些业务系统的访问令牌。

防范方式：将二维码当作承载原始数据的文件管理。只要别人能够读取二维码，就可能获得其中存储的信息。

### 7.3 二维码不是加密技术

普通二维码的编码过程不提供保密性。任何兼容的扫码器都可能读取其中的内容。

如果需要保护机密数据，应当先使用适当的加密方案，再将加密后的数据编码进二维码；同时考虑密钥管理、数据长度和扫码环境等限制。

### 7.4 不要盲目相信付款码

二维码可以表示收款地址、付款链接或其他交易参数，但它本身不证明交易对象真实可信。

付款前应确认收款人、金额及支付应用显示的交易信息。

## 八、常见问题

Q1：二维码为什么是正方形？

正方形矩阵便于将数据模块排列在两个维度中，并放置定位、同步和校正结构。QR Code 的标准版本也通过增加模块数量来扩展容量。

Q2：二维码能不能存储图片、PDF 或整个文件？

从编码原理上说，二维码可以存储二进制数据，但单个标准 QR Code 的容量有限。完整图片或 PDF 通常更适合存放在服务器或文件中，再通过二维码提供访问地址。也可以采用分片或多二维码方案，但需要专门的重组程序。

Q3：二维码破损后为什么还能识别？

因为 QR Code 添加了纠错冗余，可以恢复一定范围内丢失或错误的数据。不过，破损位置、程度和图像质量都会影响识别结果。

Q4：自己生成的二维码会过期吗？

静态二维码本身通常没有内置过期时间。只要图案完好且解码器兼容，它就可以继续被读取。但其中的网址可能失效，服务器可能关闭，或者动态二维码背后的服务可能停止运行。

Q5：二维码中存储的数据会自动上传到生成网站吗？

不一定。本文的 Python 程序在本地生成图像，不需要上传数据到二维码生成网站。但在线二维码服务可能处理你提交的内容，使用之前应检查其隐私政策。

## 九、总结

二维码看起来只是由黑白方块组成的图案，但它背后包含字符编码、二进制表示、纠错编码、矩阵布局和图像识别等多个技术环节。

回顾全文，可以记住以下五点：

1. QR Code 于 1994 年由日本 DENSO WAVE 发布，其研发源于工业生产中对更高容量、更快读取速度的需求。
    
2. 二维码通过二维模块布局表示编码数据，而不是简单地把每个黑白方块对应成一个独立的字符。
    
3. 二维码可以存储网址、普通文本、二进制数据，以及按照约定格式组织的联系人和 Wi-Fi 信息。
    
4. Reed–Solomon 纠错技术使二维码具备一定的数据恢复能力，但不能保证任意遮挡后仍然可读。
    
5. 借助 Python 的 `qrcode` 和 OpenCV 库，我们可以在本地完成二维码生成、保存、解码和校验。
    

从程序员的视角来看，二维码是一个很好的学习案例：它把抽象的数据编码与现实世界的图像识别连接起来，也展示了一个成熟技术标准如何在容量、可靠性、读取速度和使用便利性之间取得平衡。

如果你想进一步学习，可以尝试自己实现一个更完整的二维码工具：支持网址、文本、Wi-Fi 和联系人等模板，提供纠错等级和颜色选择，并在生成后自动解码验证。

## 参考资料

1. DENSO WAVE，[QR 码的成功之路：二维码的起源与发展](https://www.qrcode.com/zh/history/?utm_source=chatgpt.com)。
    
2. DENSO WAVE，[QR Code Standardization：二维码规范与纠错能力](https://www.qrcode.com/en/about/standards.html/error_correction.html?utm_source=chatgpt.com)。
    
3. DENSO，[QR Code® invented by DENSO：二维码的发明与标准化](https://www.denso.com/global/en/technology/qrcode/?utm_source=chatgpt.com)。
    
4. Python `qrcode` 项目，[python-qrcode：二维码生成库的源代码与使用说明](https://github.com/lincolnloop/python-qrcode?utm_source=chatgpt.com)。
    
5. OpenCV 官方文档，[OpenCV：计算机视觉与图像识别](https://docs.opencv.org/?utm_source=chatgpt.com)。








