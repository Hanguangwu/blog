---
title: 使用Python实现文本加密
description: 本文介绍使用Python的第三方库实现文件夹中文本加密。
date: 2026-02-27T12:34:25-08:00
draft: false
categories:
- 编程
tags:
- Python
- Cryptography
---

# 使用Python实现文本加密


## 前言

本脚本的核心功能如下所示：

1. 脚本通过PBKDF2HMAC+base64将易记密码转换为符合Fernet要求的密钥，比单纯hash更安全；
2. 支持对指定文件夹的.md文件批量加密/解密，加密文件带后缀区分，解密自动还原；
3. 包含完善的异常处理和防覆盖机制，新手也能安全使用，核心是**加密/解密必须使用相同密码+相同盐值**。

## 实现思路
1. **密钥生成**：
   - 用hashlib（如SHA-256）对原始密码进行哈希，得到固定长度的摘要
   - 将哈希结果编码为URL安全的base64格式，并截取前32字节（Fernet要求的密钥长度）
2. **文件加密/解密**：
   - 遍历指定文件夹，筛选目标文件（默认.md）
   - 使用生成的Fernet密钥对文件内容进行加密/解密
   - 加密后的文件可添加后缀（如.encrypted）区分，解密时还原
3. **健壮性设计**：
   - 异常处理（文件读写、加密解密错误）
   - 路径验证、文件覆盖提示
   - 支持指定目标文件类型、自定义加密后缀

## 完整代码

```python
import os
import base64
from cryptography.fernet import Fernet, InvalidToken
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC

def generate_fernet_key(password: str, salt: bytes = b'system_design_salt_123') -> bytes:
    """
    将易记密码转换为符合Fernet要求的32字节URL安全base64密钥
    
    Args:
        password: 原始密码（如"password"）
        salt: 盐值（增加安全性，建议自定义）
    
    Returns:
        符合Fernet要求的32字节URL安全base64密钥
    """
    # 1. 将密码转换为字节串
    password_bytes = password.encode('utf-8')
    
    # 2. 使用PBKDF2HMAC进行密钥派生（比单纯hash更安全）
    kdf = PBKDF2HMAC(
        algorithm=hashes.SHA256(),
        length=32,  # Fernet要求32字节
        salt=salt,
        iterations=100000,  # 迭代次数，越高越安全（但稍慢）
        backend=default_backend()
    )
    key = kdf.derive(password_bytes)
    
    # 3. 编码为URL安全的base64格式（Fernet要求的格式）
    fernet_key = base64.urlsafe_b64encode(key)
    
    return fernet_key

def process_files(
    folder_path: str,
    password: str,
    mode: str = 'encrypt',  # 'encrypt' 加密 / 'decrypt' 解密
    file_ext: str = '.md',  # 目标文件类型
    encrypt_suffix: str = '.encrypted'  # 加密文件后缀
):
    """
    加密/解密指定文件夹中的目标文件
    
    Args:
        folder_path: 目标文件夹路径
        password: 加密/解密密码
        mode: 操作模式（encrypt/decrypt）
        file_ext: 要处理的文件扩展名（如.md）
        encrypt_suffix: 加密文件的后缀（解密时会移除）
    """
    # 验证文件夹是否存在
    if not os.path.isdir(folder_path):
        print(f"错误：文件夹 '{folder_path}' 不存在！")
        return
    
    # 验证操作模式
    if mode not in ['encrypt', 'decrypt']:
        print("错误：mode只能是 'encrypt' 或 'decrypt'")
        return
    
    # 生成Fernet密钥
    try:
        key = generate_fernet_key(password)
        fernet = Fernet(key)
        print(f"✅ 密钥生成成功（基于密码：{password}）")
    except Exception as e:
        print(f"❌ 密钥生成失败：{str(e)}")
        return
    
    # 遍历文件夹中的文件
    processed_count = 0
    failed_files = []
    
    for filename in os.listdir(folder_path):
        file_path = os.path.join(folder_path, filename)
        
        # 跳过文件夹，只处理文件
        if os.path.isdir(file_path):
            continue
        
        # 筛选目标文件（加密：匹配扩展名；解密：匹配加密后缀）
        if mode == 'encrypt':
            if not filename.lower().endswith(file_ext.lower()):
                continue  # 只处理指定扩展名的文件
            target_file = file_path + encrypt_suffix  # 加密后的文件名
        else:
            if not filename.endswith(encrypt_suffix):
                continue  # 只处理带加密后缀的文件
            # 解密：移除加密后缀，还原原文件名
            target_file = filename[:-len(encrypt_suffix)]
            target_file = os.path.join(folder_path, target_file)
        
        # 跳过已存在的目标文件（避免覆盖）
        if os.path.exists(target_file):
            print(f"⚠️ 跳过：目标文件 '{target_file}' 已存在，避免覆盖")
            continue
        
        # 读取原文件内容
        try:
            with open(file_path, 'rb') as f:
                content = f.read()
        except Exception as e:
            print(f"❌ 读取文件失败 '{file_path}'：{str(e)}")
            failed_files.append(file_path)
            continue
        
        # 加密/解密处理
        try:
            if mode == 'encrypt':
                processed_content = fernet.encrypt(content)
                print(f"🔒 加密中：{filename} -> {os.path.basename(target_file)}")
            else:
                processed_content = fernet.decrypt(content)
                print(f"🔓 解密中：{filename} -> {os.path.basename(target_file)}")
        except InvalidToken:
            print(f"❌ 解密失败 '{file_path}'：密码错误或文件已被篡改")
            failed_files.append(file_path)
            continue
        except Exception as e:
            print(f"❌ 处理失败 '{file_path}'：{str(e)}")
            failed_files.append(file_path)
            continue
        
        # 写入处理后的内容
        try:
            with open(target_file, 'wb') as f:
                f.write(processed_content)
            processed_count += 1
        except Exception as e:
            print(f"❌ 写入文件失败 '{target_file}'：{str(e)}")
            failed_files.append(file_path)
            continue
    
    # 输出处理结果
    print("\n" + "="*50)
    print(f"处理完成！")
    print(f"✅ 成功处理文件数：{processed_count}")
    if failed_files:
        print(f"❌ 失败文件数：{len(failed_files)}")
        print(f"失败列表：{failed_files}")
    else:
        print(f"🎉 所有目标文件处理成功！")

if __name__ == "__main__":
    # -------------------------- 配置参数 --------------------------
    # 你需要根据实际情况修改以下参数
    FOLDER_PATH = "./System-Design"  # 要处理的文件夹路径
    PASSWORD = "password123"         # 易记密码（自定义）
    OPERATION_MODE = "encrypt"       # encrypt=加密 / decrypt=解密
    TARGET_FILE_EXT = ".md"          # 要处理的文件类型（如.md/.txt）
    ENCRYPT_SUFFIX = ".encrypted"    # 加密文件后缀（解密时自动移除）
    # --------------------------------------------------------------
    
    # 执行文件处理
    process_files(
        folder_path=FOLDER_PATH,
        password=PASSWORD,
        mode=OPERATION_MODE,
        file_ext=TARGET_FILE_EXT,
        encrypt_suffix=ENCRYPT_SUFFIX
    )
```

## 代码关键部分解释
### 1. 密钥生成（核心）
```python
def generate_fernet_key(password: str, salt: bytes = b'system_design_salt_123') -> bytes:
```
- **PBKDF2HMAC**：比单纯用hashlib更安全的密钥派生方式，通过盐值（salt）和迭代次数增加破解难度
- **盐值（salt）**：建议你自定义（如替换`b'system_design_salt_123'`），相同密码+不同盐值会生成不同密钥
- **URL安全base64**：Fernet要求密钥必须是URL安全的base64编码，`base64.urlsafe_b64encode()` 专门处理这种场景
- **32字节长度**：Fernet密钥固定要求32字节，PBKDF2HMAC直接指定`length=32`保证符合要求

### 2. 文件处理逻辑
- **加密模式**：筛选`.md`文件，加密后生成`文件名.md.encrypted`（可自定义后缀）
- **解密模式**：筛选带`.encrypted`后缀的文件，解密后还原为原文件名（如`文件名.md`）
- **防覆盖**：检测目标文件是否存在，避免误覆盖已有文件
- **异常处理**：捕获文件读写、加密解密（如密码错误）等异常，输出清晰的错误提示

### 3. 使用方法
1. **安装依赖**（cryptography不是Python内置库）：
   ```bash
   pip install cryptography
   ```
2. **修改配置参数**：
   - `FOLDER_PATH`：改为你要处理的文件夹路径（如`./System-Design`）
   - `PASSWORD`：改为你的易记密码（如`my_secure_password`）
   - `OPERATION_MODE`：`encrypt`（加密）/ `decrypt`（解密）
3. **运行脚本**：
   ```bash
   python file_encryptor.py
   ```

## 示例效果
假设`System-Design`文件夹下有`chapter_1.md`：
- **加密后**：生成`chapter_1.md.encrypted`（原文件保留，加密文件为二进制密文）
- **解密时**：将`chapter_1.md.encrypted`解密为`chapter_1.md`（需使用相同密码）

## 参考资料

https://geek-blogs.com/blog/fernet-python/

https://www.cnblogs.com/tian-jiang-ming/p/8313397.html

https://deepinout.com/python/python-top-articles/1695265611_tr_fernet-symmetric-encryption-using-a-cryptography-module-in-python.html

https://geek-docs.com/python/python-ask-answer/686_python_how_to_encrypt_text_with_a_password_in_python.html

https://www.runoob.com/python3/python-hashlib.html

https://cryptools.github.io/




