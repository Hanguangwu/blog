---
title: 自定义GitHub Pages域名
description: 本文介绍如何为 GitHub Pages 配置 CloudFlare 域名。
date: 2026-06-05T12:34:25-08:00
draft: false
categories:
- GitHub
tags:
- 网站开发
- 域名设置
---



# 为 GitHub Pages 配置 CloudFlare 域名指南


## 关键步骤

### 第二步｜在 Cloudflare 添加域名

**目标：** 在 Cloudflare 中接管你的域名

#### 操作步骤：

**1. 登录 Cloudflare Dashboard**

访问 [https://dash.cloudflare.com/](https://dash.cloudflare.com/) 并登录。

**2. 点击添加网站**

在 Dashboard 首页，点击 **Add a Site**（添加网站）按钮。

**3. 输入你的域名**

在输入框中输入你的域名（只输入根域名，不带 www）：

```
example.com
```

点击 **Add Site** 继续。

**4. 选择套餐**

选择 **Free** 套餐（个人博客/网站完全够用），点击 **Continue**。

**5. 选择导入现有 DNS 记录（可选）**

Cloudflare 会尝试扫描你域名现有的 DNS 记录。

> 💡 如果这是新域名，没有任何重要记录，可以选择 **Skip** 跳过这一步。

**6. 记录 Cloudflare 分配的 Nameservers**

⚠️ **这是关键步骤！**

Cloudflare 页面会显示类似下面的信息：

```
We have assigned the following nameservers to your zone:

amir.ns.cloudflare.com
violet.ns.cloudflare.com
```

> 💡 **重要**：Spaceship 和 Cloudflare 都显示这两组 Nameserver 信息是**正常的**。只要你第一步在 Spaceship 填的是 Cloudflare 的 Nameserver，现在两边显示相同就对了。

**7. 完成设置**

点击 **Done, take me to DNS** 进入 DNS 设置页面。

---

### 第三步｜在 Cloudflare 配置 DNS 记录

**目标：** 将域名指向 GitHub Pages

#### 基础知识｜DNS 记录类型

|记录类型|用途|示例|
|---|---|---|
|**A 记录**|将域名指向一个 IPv4 地址|`example.com` → `185.199.108.153`|
|**CNAME**|将子域名指向另一个域名|`www.example.com` → `example.com`|
|**TXT**|用于验证、SPF 邮件安全等|验证域名所有权|
|**AAAA**|将域名指向 IPv6 地址|较少使用|

> ⚠️ **CNAME 记录不能用于根域名（@）！**
> 
> DNS 规范规定 CNAME 记录不能与其他记录类型共存。根域名通常需要同时存在 SOA、NS、A 等记录，所以**根域名不能使用 CNAME**。这是 DNS 协议的限制，不是 Cloudflare 或 GitHub 的限制。

#### GitHub Pages 的 IP 地址

GitHub Pages 使用的固定 IP 地址：

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

#### 配置 DNS 记录

进入 Cloudflare Dashboard → 你的域名 → **DNS** → **Records**。

点击 **Add record**，按以下方式配置：

**记录 1：根域名（A 记录）**

|设置项|值|说明|
|---|---|---|
|Type|`A`|A 记录类型|
|Name|`@`|表示根域名（你的域名本身）|
|IPv4 address|`185.199.108.153`|GitHub Pages 的第一个 IP|
|Proxy status|**DNS only**（⚪ 灰色云）|⚠️ GitHub Pages 必须用灰色云！|

点击 **Save**。

**记录 2-4：备用 A 记录**

重复上述步骤，添加另外三个 IP 地址：

```
185.199.109.153
185.199.110.153
185.199.111.153
```

> 💡 四个 A 记录同时存在，GitHub 会自动选择最快响应的 IP。

**记录 5：www 子域名（CNAME）**

|设置项|值|说明|
|---|---|---|
|Type|`CNAME`|CNAME 记录类型|
|Name|`www`|www 子域名|
|Target|`username.github.io`|⚠️ 把 `username` 换成你的 GitHub 用户名|
|Proxy status|**DNS only**（⚪ 灰色云）|⚠️ GitHub Pages 必须用灰色云！|

点击 **Save**。

#### DNS 记录配置完成后的样子

```
| 类型 | 名称 | IPv4 地址 / 目标           | 代理状态 |
|------|------|---------------------------|----------|
| A    | @    | 185.199.108.153          | DNS only |
| A    | @    | 185.199.109.153          | DNS only |
| A    | @    | 185.199.110.153          | DNS only |
| A    | @    | 185.199.111.153          | DNS only |
| CNAME| www  | username.github.io        | DNS only |
```

> ⚠️ **代理状态（Proxy Status）必须是灰色云！**
> 
> 橙色云 ☁️ = Cloudflare CDN 代理 灰色云 ⚪ = 仅 DNS 解析
> 
> **GitHub Pages 的 DNS 记录必须设为灰色云**，因为：
> 
> 1. Cloudflare 的 CDN 代理会干扰 GitHub 的 SSL 证书自动验证
> 2. GitHub 要求直接连接到他们的服务器
> 
> 你可以为其他子域名（如 `api.example.com`）使用橙色云，但 **@ 和 www** 这两条 GitHub Pages 的记录必须用灰色云。

---

### 第四步｜在 GitHub 配置自定义域名

**目标：** 告诉 GitHub：“这个域名是我的，请用这个域名来托管我的网站”。

每次发表新博客文章后，触发GitHub的自动构建时，自定义域名就没了，需要重新去GitHub保存，很麻烦。只需要在Hugo的静态目录增加一个CNAME文件，指向自定义域名即可。

#### 前置条件

确保你的仓库已经启用了 GitHub Pages：

1. 进入你的仓库（如 `https://github.com/username/text-matrix`）
2. 点击 **Settings** → **Pages**
3. 确认 **Source** 设置为 **Deploy from a branch**
4. 选择 **main** 分支和 **/ (root)** 文件夹
5. 点击 **Save**

#### 操作步骤：

**1. 进入仓库设置**

登录 GitHub，进入你的仓库页面。

**2. 打开 Pages 设置**

左侧菜单点击 **Pages**。

**3. 配置自定义域名**

在 **Custom domain** 输入框中输入你的域名：

```
example.com
```

> 💡 输入后 GitHub 会立即开始 DNS 检查

**4. 保存设置**

点击 **Save**。

**5. 等待 DNS 验证**

GitHub 会自动检查你的 DNS 配置是否正确。检查项目：

```
✓ DNS check passed
```

或错误信息：

```
✗ DNS check failed
 原因1：DNS 记录尚未传播（等待 5 分钟到 24 小时）
 原因2：DNS 记录配置错误（检查第三步）
 原因3：使用了 Cloudflare 橙色云代理（改为灰色云）
```

> 💡 **如何加速验证？**
> 
> - 等待 5-10 分钟后再试
> - 使用 `dig www.example.com` 命令检查 DNS 是否已生效
> - 清除浏览器缓存或使用无痕模式

**6. 勾选 Enforce HTTPS**

当 DNS 验证通过后，勾选 **Enforce HTTPS**（强制使用 HTTPS）。

> 💡 GitHub 会使用 Let’s Encrypt 自动为你的域名申请 SSL 证书。这个过程可能需要 **24-48 小时**。在证书生效前，HTTPS 可能会有警告，这是正常现象。


## 参考资料

[Cloudflare + GitHub Pages 自定义域名配置指南](https://txtmix.com/posts/tech/cloudflare-github-pages-custom-domain-guide/)

[Cloudflare DNS 托管+Github Pages HTTPS加密避坑](https://me.onlyra1n.top/posts/a72e)

[小白学网站部署，手把手教你，Github Pages 如何自定义域名](https://zhuanlan.zhihu.com/p/1886365022571176637)

[我的「个人Python网站制作」第三部曲：Cloudflare完美加速GitHub Pages国内访问速度，嘎嘎快那种](https://cloud.tencent.com/developer/article/2654949)

[Hugo构建时自动CNAME到自定义域名](https://bingkaer.github.io/p/github-pages-cname/)









