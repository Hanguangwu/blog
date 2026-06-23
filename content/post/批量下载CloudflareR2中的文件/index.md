---
title: 批量下载 Cloudflare R2 中的文件
description: 本文介绍批量下载 Cloudflare R2 中文件的过程。
date: 2026-01-14T15:34:25-08:00
draft: false
categories:
- 编程
tags:
- CloudFlare R2
- Python
---

# 批量下载 Cloudflare R2 中的文件

## 前言

这里可以使用Python脚本，也可以使用r2clone工具。当然也可以使用CloudFlare R2第三方客户端。

本文资料来源如下所示：

[使用 Python 脚本批量下载 Cloudflare R2 中的文件](https://djdog.cc/blog/2025/r2-download)

[使用 rclone 批量导出 Cloudflare R2 存储文件](https://www.210k.cc/export-cloudflare-r2-files/)

[Cloudflare R2 文件管理器使用说明](http://bbs1.phpdisk.com/thread-372365-1-1.html)

## 使用 Python 脚本批量下载 Cloudflare R2 中的文件

这里给出使用 AWS SDK for Python（Boto3）的示例，以便后续使用。

### 准备工作

下载 Boto3 库，这里以 `uv` 作为 Python 的包管理器：

```sh
uv add boto3
```

### 运行脚本

```python
import boto3
import os

# 显式设置R2的访问凭证和配置
s3_client = boto3.client(
    's3',
    aws_access_key_id='YOUR_ACCESS_KEY',
    aws_secret_access_key='YOUR_SECRET_KEY',
    endpoint_url='https://YOUR_ACCOUNT_ID.r2.cloudflarestorage.com',
    region_name='auto'
)

# R2存储桶名称
bucket_name = 'YOUR_BUCKET_NAME'

# 本地存储路径
local_dir = './files'

def download_files(bucket_name, local_dir):
    # 列出存储桶中的所有对象
    response = s3_client.list_objects_v2(Bucket=bucket_name)

    if 'Contents' in response:
        for obj in response['Contents']:
            file_key = obj['Key']
            local_file_path = os.path.join(local_dir, file_key)

            # 创建本地目录
            os.makedirs(os.path.dirname(local_file_path), exist_ok=True)

            # 下载文件
            s3_client.download_file(bucket_name, file_key, local_file_path)
            print(f'Downloaded {file_key} to {local_file_path}')
    else:
        print('No files found in the bucket.')

# 调用下载函数
download_files(bucket_name, local_dir)
```

其中的 `YOUR_ACCESS_KEY` 等需要替换成自己所需要的。

注意： `YOUR_ACCESS_KEY`要有访问权限。

## 使用 rclone 批量导出 Cloudflare R2 存储文件

### 背景

Cloudflare R2 是一个兼容 S3 的对象存储服务。虽然 Cloudflare 提供了 wrangler CLI 工具，但它**不支持批量下载文件**，只能下载单个文件。要批量导出 R2 存储桶中的文件，需要使用第三方工具。

### 官方推荐方案：使用 rclone

根据 Cloudflare 官方文档，批量操作 R2 文件的推荐方式是使用 [rclone](https://rclone.org/)，一个开源的云存储同步工具。

### 完整操作步骤

#### 第一步：安装 rclone

**macOS:**

```
brew install rclone
```

**Linux:**

```
curl https://rclone.org/install.sh | sudo bash
```

**Windows:** 从 [rclone 官网](https://rclone.org/downloads/) 下载安装包

#### 第二步：创建 R2 API Token

1. 登录 Cloudflare Dashboard
2. 进入 R2 管理页面：`https://dash.cloudflare.com/{account_id}/r2/overview`
3. 点击右上角 **“Manage R2 API Tokens”**
4. 点击 **“Create API Token”**
5. 配置权限：
    - **Token name**: 随意命名（例如：`r2-export-token`）
    - **Permissions**:
        - Object Read（读取对象）
        - Object List（列出对象）
    - **Bucket scope**: 选择需要访问的存储桶，或选择 “All buckets”
6. 点击 **“Create API Token”**
7. **重要**：复制并保存以下信息（只会显示一次）：
    - Access Key ID
    - Secret Access Key
    - Endpoint URL（通常格式为：`https://{account_id}.r2.cloudflarestorage.com`）

#### 第三步：配置 rclone

运行配置命令：

```
rclone config
```

按照提示完成配置：

```
1. 选择操作
n) New remote
> n

2. 输入 remote 名称
name> r2prod

3. 选择存储类型
Storage> s3
（输入数字或直接输入 s3）

4. 选择 S3 提供商
provider> Cloudflare
（选择 Cloudflare 或输入对应数字）

5. 选择认证方式
env_auth> false
（选择手动输入凭据）

6. 输入 Access Key ID
access_key_id> [粘贴你的 Access Key ID]

7. 输入 Secret Access Key
secret_access_key> [粘贴你的 Secret Access Key]

8. 区域设置
region> auto
（Cloudflare R2 使用 auto）

9. Endpoint 设置
endpoint> https://{account_id}.r2.cloudflarestorage.com
（替换 {account_id} 为你的实际 Account ID）

10. Location constraint
location_constraint>
（留空，直接回车）

11. ACL 设置
acl> private

12. 服务端加密
server_side_encryption>
（留空，直接回车）

13. 存储类别
storage_class>
（留空，直接回车）

14. 高级配置
Edit advanced config? (y/n)
> n

15. 确认配置
y) Yes this is OK (default)
> y
```

#### 第四步：测试连接

验证 rclone 配置是否正确：

```
# 列出所有存储桶
rclone lsd r2prod:

# 列出特定存储桶的内容
rclone ls r2prod:your-bucket-name --max-depth 1
```

#### 第五步：下载文件

**基本用法**

下载整个存储桶：

```
rclone copy r2prod:bucket-name ./local-directory
```

下载特定前缀的文件：

```
rclone copy r2prod:bucket-name/prefix/ ./local-directory
```

**推荐参数**

使用以下参数以获得更好的性能和用户体验：

```
rclone copy r2prod:bucket-name/prefix/ ./output-directory \
  --progress \              # 显示进度
  --transfers 8 \           # 并行传输数（默认 4）
  --checkers 16 \           # 并行检查数（默认 8）
  --stats 2s \              # 每 2 秒更新统计
  --stats-one-line \        # 单行显示统计
  --s3-no-check-bucket      # 跳过存储桶检查（提升性能）
```

**常用场景**

**只下载新文件或已修改的文件（增量下载）：**

```
rclone copy r2prod:bucket/path/ ./local-path --progress
```

**同步（删除本地多余文件）：**

```
rclone sync r2prod:bucket/path/ ./local-path --progress
```

⚠️ 注意：sync 会删除本地目录中不存在于 R2 的文件！

**预览将要下载的文件（不实际下载）：**

```
rclone copy r2prod:bucket/path/ ./local-path --dry-run
```

### 自动化脚本示例

创建一个便捷的导出脚本 `export-r2-files.sh`：

```
#!/bin/bash
# 从 Cloudflare R2 批量导出文件

set -e

# 配置
REMOTE_NAME="r2prod"
BUCKET_NAME="your-bucket-name"
PREFIX="${1:-workflow-outputs/}"
OUTPUT_DIR="${2:-./r2-exports}"

echo "🚀 Starting R2 files export..."
echo "📦 Bucket: ${BUCKET_NAME}"
echo "📁 Prefix: ${PREFIX}"
echo "💾 Output: ${OUTPUT_DIR}"
echo ""

# 检查 rclone 是否安装
if ! command -v rclone &> /dev/null; then
    echo "❌ rclone is not installed!"
    echo "Install it with: brew install rclone"
    exit 1
fi

# 检查远程是否配置
if ! rclone listremotes | grep -q "^${REMOTE_NAME}:$"; then
    echo "❌ rclone remote '${REMOTE_NAME}' is not configured!"
    echo "Run: rclone config"
    exit 1
fi

# 创建输出目录
mkdir -p "${OUTPUT_DIR}"

# 测试连接
echo "🔗 Testing R2 connection..."
if ! rclone lsd "${REMOTE_NAME}:${BUCKET_NAME}" --max-depth 1 > /dev/null 2>&1; then
    echo "❌ Failed to connect to R2 bucket"
    exit 1
fi
echo "✅ Connection successful"
echo ""

# 下载文件
echo "⬇️  Downloading files from R2..."
rclone copy \
    "${REMOTE_NAME}:${BUCKET_NAME}/${PREFIX}" \
    "${OUTPUT_DIR}/${PREFIX}" \
    --progress \
    --transfers 8 \
    --checkers 16 \
    --stats 2s \
    --stats-one-line \
    --s3-no-check-bucket

echo ""
echo "✅ Export completed successfully!"
echo "📁 Files saved to: ${OUTPUT_DIR}/${PREFIX}"
```

使用方法：

```
chmod +x export-r2-files.sh

# 下载所有 workflow-outputs/ 文件
./export-r2-files.sh

# 下载特定月份的文件
./export-r2-files.sh workflow-outputs/2025-01/

# 指定输出目录
./export-r2-files.sh workflow-outputs/ ./my-exports
```

### 常见问题

#### Q1: 遇到 403 Access Denied 错误

**原因**：API Token 权限不足或配置错误

**解决方法**：

1. 确认 API Token 有 Object Read 和 Object List 权限
2. 在 rclone 命令中添加 `--s3-no-check-bucket` 参数
3. 或在 rclone config 中添加：`no_check_bucket = true`

#### Q2: Endpoint URL 格式不正确

**正确格式**：`https://{account_id}.r2.cloudflarestorage.com`

**错误示例**：

- ❌ `https://{account_id}.r2.cloudflarestorage.com/bucket-name`（不要包含存储桶名）
- ❌ `https://r2.cloudflarestorage.com`（缺少 account_id）

#### Q3: 如何找到 Account ID？

方法一：从 Cloudflare Dashboard URL 获取

- 登录后 URL 格式：`https://dash.cloudflare.com/{account_id}/...`

方法二：从 R2 页面查看

- 进入 R2 管理页面，查看 Endpoint URL

#### Q4: rclone 速度慢怎么办？

尝试调整并行参数：

```
rclone copy r2prod:bucket/path/ ./local \
  --transfers 16 \      # 增加并行传输数
  --checkers 32 \       # 增加并行检查数
  --buffer-size 128M    # 增加缓冲区大小
```

#### Q5: 如何只下载特定文件类型？

使用 `--include` 或 `--exclude` 参数：

```
# 只下载 .json 文件
rclone copy r2prod:bucket/ ./local --include "*.json"

# 排除 .tmp 文件
rclone copy r2prod:bucket/ ./local --exclude "*.tmp"

# 只下载图片
rclone copy r2prod:bucket/ ./local --include "*.{jpg,png,gif}"
```

### 其他可选方案

#### 1. AWS CLI

R2 兼容 S3 API，也可以使用 AWS CLI：

```
# 配置
aws configure --profile r2
# AWS Access Key ID: [R2 Access Key]
# AWS Secret Access Key: [R2 Secret Key]
# Default region name: auto
# Default output format: json

# 设置 endpoint
export AWS_ENDPOINT_URL=https://{account_id}.r2.cloudflarestorage.com

# 下载文件
aws s3 cp s3://bucket-name/path/ ./local-path/ --recursive --profile r2
```

#### 2. S3-compatible GUI 工具

- [Cyberduck](https://cyberduck.io/) - 免费，支持 S3
- [Mountain Duck](https://mountainduck.io/) - 付费，可将 R2 挂载为本地磁盘
- [S3 Browser](https://s3browser.com/) - Windows，免费

#### 3. 编程方式

使用 AWS SDK 访问 R2（以 Node.js 为例）：

```
import { S3Client, ListObjectsV2Command, GetObjectCommand } from '@aws-sdk/client-s3';

const s3 = new S3Client({
  region: 'auto',
  endpoint: 'https://{account_id}.r2.cloudflarestorage.com',
  credentials: {
    accessKeyId: 'your-access-key',
    secretAccessKey: 'your-secret-key',
  },
});

// 列出对象
const listCommand = new ListObjectsV2Command({
  Bucket: 'bucket-name',
  Prefix: 'workflow-outputs/',
});
const { Contents } = await s3.send(listCommand);

// 下载文件
for (const object of Contents) {
  const getCommand = new GetObjectCommand({
    Bucket: 'bucket-name',
    Key: object.Key,
  });
  const response = await s3.send(getCommand);
  // 处理 response.Body
}
```

### 总结

- ✅ **推荐方案**：rclone（官方推荐，稳定可靠）
- ✅ **备选方案**：AWS CLI、GUI 工具、编程 SDK
- ❌ **不推荐**：wrangler CLI（不支持批量操作）

## 其他工具

### Cloudflare R2 CLI

[cloudflare-r2-cli](https://github.com/pxgo/cloudflare-r2-cli)

Cloudflare R2 CLI 是一个简单的命令行工具，旨在帮助用户通过命令行轻松管理 Cloudflare R2 存储。该工具使用 Node.js 和 AWS SDK 构建，为 Cloudflare R2 提供上传、下载、重命名、删除等基本的文件操作功能。

#### 特性

- **文件上传**：将本地文件上传到指定的 Cloudflare R2 存储桶
- **文件下载**：从 Cloudflare R2 存储桶下载文件到本地
- **列出文件**：显示存储桶中的所有文件及其详细信息
- **文件信息**：查看指定文件的元数据和访问链接
- **文件重命名/移动**：支持文件重命名或在存储桶内移动文件
- **文件删除**：从存储桶中删除指定文件
- **列出存储桶**：显示账户中所有存储桶列表

#### 安装

1. 克隆仓库：

```shell
git clone https://github.com/pxgo/cloudflare-r2-cli.git
cd cloudflare-r2-cli
```

2. 安装依赖项：

```shell
npm install
```

3. 设置环境变量：

```shell
export R2_ACCOUNT_ID="your_account_id"
export R2_ACCESS_KEY="your_access_key"
export R2_SECRET_KEY="your_secret_key"
```

4. 编译

```shell
bun build index.ts --compile --outfile=build/r2
```

#### 使用说明

可用命令:

- `upload <bucketName> <filePath> <key>` - 上传文件到 R2
- `download <bucketName> <key> <downloadPath>` - 从 R2 下载文件
- `list <bucketName>` - 显示存储桶中的文件列表
- `info <bucketName> <key>` - 显示文件的信息
- `rename <bucketName> <oldKey> <newKey>` - 修改文件的目录或文件名
- `delete <bucketName> <key>` - 删除文件
- `buckets` - 列出所有存储桶

例子：

```shell
r2 upload books ~/JavaScript从入门到入狱.pdf 武林秘籍.pdf
```

### R2 文件管理系统

[一个基于 Cloudflare Workers 和 R2 的现代化文件管理系统，支持文件上传、下载、移动、重命名等操作。](https://github.com/zhao-zg/Cloudflare-Workers-R2)

基于 **Cloudflare Workers + R2** 的轻量文件管理面板。  

未登录：浏览 / 下载；登录：上传、删除、移动、重命名、批量操作。

---

#### ✨ 功能

- 游客：浏览文件 & 文件夹、下载、（可选）复制直链
- 管理员：以上全部 + 上传（多文件）、创建文件夹、批量选择、移动、重命名、删除
- 自动重试上传（指数退避 1s/2s/4s/8s/16s）
- 纯前端界面 + Worker 代理后端（不暴露密钥）
- 零构建依赖，直接粘贴部署
### [R2Client](https://r2client.com/zh)

[让 Cloudflare R2 像本地磁盘一样好用](https://r2client.com/zh)

### 参考资料

- [Cloudflare R2 官方文档 - rclone](https://developers.cloudflare.com/r2/examples/rclone/)
- [rclone 官方文档](https://rclone.org/docs/)
- [Cloudflare R2 API 文档](https://developers.cloudflare.com/r2/api/)










