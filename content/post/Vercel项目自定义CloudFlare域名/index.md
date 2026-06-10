---
title: Vercel项目部署+CloudFlare自定义域名配置｜解决国内访问卡顿无法访问问题
description: 本文介绍Vercel项目部署+CloudFlare自定义域名配置流程。
date: 2026-06-11T12:34:25-08:00
draft: false
categories:
- APP
tags:
- GitHub
- Vercel
- CloudFlare
---

# Vercel项目部署+CloudFlare自定义域名配置｜解决国内访问卡顿无法访问问题


## 前言

只要有一个自己的域名，我们就可以充分利用两大赛博善人资源搭建自己的新天地！

依托Github Education福利可白嫖name.com一年免费域名，搭配Vercel免费部署、CloudFlare免费CDN，零成本搭建个人站点、AI前端项目。

原生Vercel节点国内直连延迟高、时常打不开站点，官方解析方案适配海外网络，本文整理了一套完整实操方案：一套**专属国内加速优化方案**（适配ChatGPT-Next-Web/NextChat类AI项目）、一套**通用CloudFlare域名接入方案**，全程免费、新手可复刻。

## 详细步骤

### 第一步：将你的域名添加到 CloudFlare

1. 登录你的 CloudFlare 账户。
2. 点击 **Add a site** 按钮，输入你的根域名（如 `your-domain.com`），然后点击 **Add site**。
3. ![添加站点到CloudFlare](https://www.wyattyuan.studio/_astro/cloudflare-add-site.CwHEqj6Q_Z1r2xho.webp)
4. 选择 **Free（免费）** 套餐，点击 **Continue**。
5. CloudFlare 会自动扫描你域名现有的 DNS 记录，等待扫描完成后点击 **Continue**。

### 第二步：修改域名的 Nameservers (NS 记录)

这是最关键的一步，目的是将你域名的解析管理权正式交给 CloudFlare。

1. CloudFlare 会显示你域名当前的 Nameservers，并提供两个新的 CloudFlare Nameservers，例如：  
    `melissa.ns.cloudclare.com` 和 `skip.ns.cloudflare.com` ![CloudFlare-1](https://www.wyattyuan.studio/_astro/cloudflare-1.H_ABvNi__yAuns.webp)
2. 登录你购买域名的平台（如 GoDaddy、NameSilo、阿里云/万网），这里以 name.com 为例。
3. 找到你域名的 DNS 管理或 Nameserver 管理设置。
4. 删除原有的 Nameserver 地址，填入 CloudFlare 提供的两个新地址。
5. 保存更改。此更改全球生效可能需要几分钟到数小时。你可以回到 CloudFlare 页面，点击 **Done, check nameservers** 让 CloudFlare 定期检查。生效后 CloudFlare 会邮件通知你。

### 第三步：在 CloudFlare 中配置 DNS 记录（并开启代理）

在等待 NS 记录生效的同时，可以先配置好 DNS 记录：

1. 在 CloudFlare 网站左侧菜单选择 **DNS**。
2. 根据 Vercel 要求，添加一条 **A 记录** 或 **CNAME 记录** 指向 Vercel：
    **推荐：A 记录（用于根域名）**
    
    ```
    Type: A
    Name: @ （代表你的根域名 your-domain.com）
    IPv4 address: 76.223.126.88
    Proxy status: 确保云朵图标是橙色的（Proxied），表示流量会经过 CloudFlare 的 CDN。
    ```
    
    点击 **Save**。
    
    **如果你想用 www 子域名（如 [www.your-domain.com](http://www.your-domain.com/) ）：**
    
    - Type: CNAME
    - Name: www
    - Target: cname.vercel-dns.com
    - Proxy status: 确保云朵图标是橙色的（Proxied）。
    
    点击 **Save**。
    

### 第四步：配置 SSL/TLS 加密模式（非常重要）

这一步配置错误会导致网站无法访问或出现重定向循环。

1. 在 CloudFlare 网站左侧菜单选择 **SSL/TLS**。
2. 在 **Overview** 选项卡中，选择 **Full (Strict)** 模式。

> **为什么？**  
> Vercel 会自动为你的自定义域名部署 SSL 证书，实现 HTTPS 加密。Full (Strict) 模式要求从用户浏览器到 CloudFlare，以及从 CloudFlare 到 Vercel 源服务器的全程都使用严格的、受信任的证书进行加密，这是最安全且与 Vercel 兼容性最好的模式。

### 第五步：在 Vercel 中添加你的自定义域名

1. 回到你的 Vercel 项目控制台。
2. 进入项目的 **Settings → Domains**。
3. 输入你的自定义域名（如 your-domain.com 或 [www.your-domain.com](http://www.your-domain.com/) ），点击 **Add**。
4. 由于你已在 CloudFlare 配置好 DNS，Vercel 应能很快检测到并显示 **Valid Configuration**，并自动处理 SSL 证书签发。

---

完成以上所有步骤后，等待几分钟，你的网站应该就可以通过自定义域名在国内顺畅访问了。

## 完整流程示例

下面以在Vercel商部署NextChat项目为例展示具体过程。

![w](https://i-blog.csdnimg.cn/direct/ca7b2f1f90e64bee95f258b1bbde3249.png)

在Vercel上部署项目后，我们需要在CloudFlare的DNS记录中添加一条CNAME记录，指向cname-china.vercel-dns.com，但不要开启代理。

如果CloudFlare绑定的域名是chwz.us.kg，我们想用 gpt.chwz.us.kg 访问上面的Vercel项目，就应该如下配置：

![image](https://i-blog.csdnimg.cn/direct/a647b8d9797c4e7baa5753bb6e669edb.png)

配置好后，回到Vercel页面，找到绑定域名设置，加入你要绑定的域名，例如一开始的gpt.chwz.us.kg：

![image3](https://i-blog.csdnimg.cn/direct/fb5b5c453ccf4d2ba5a7532a73c43563.png)

等到出现两个蓝勾后，再次回到CloudFlare页面，将CNAME中一开始填入的cname-china.vercel-dns.com改为vercel-cname.xingpingcn.top，保存后回到Vercel等待两个蓝勾出现。

这时候如果访问gpt.chwz.us.kg，就会发现速度非常快，加速的目的也就实现了。

## 参考资料

[CloudFlare + Vercel 实现网站中国访问加速](https://blog.csdn.net/2402_88233879/article/details/145651455)

[Cloudfare解决Vercel部署后无法访问的问题](https://www.wyattyuan.studio/blog/web-dev/CloudFlare-vercel/)

[使用Vercel免费部署上线你的前端项目，并使用Cloudfare解决 Vercel 部署网站在国内被墙的问题](https://juejin.cn/post/7383894687302434825)
[Vercel域名加速](https://docs.tangly1024.com/article/vercel-accelerate)

[enhanced-FaaS-in-China](https://github.com/xingpingcn/enhanced-FaaS-in-China)

[推一下 Vercel 加速节点](https://www.yt-blog.top/9952/)


















