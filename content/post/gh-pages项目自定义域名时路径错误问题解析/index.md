---
title: gh-pages项目自定义域名时路径错误问题解析
description: 本文介绍解决 gh-pages 项目自定义域名时路径错误问题的方法。
date: 2026-07-04T12:34:25-08:00
draft: false
categories:
- GitHub
tags:
- gh-pages
- vite
---


# GitHub Pages 自定义域名下 Vite 项目 JS 模块加载失败：MIME 类型错误

## 问题现象

通过 GitHub Actions 将 Vite 项目部署到 GitHub Pages 后，打开网站发现页面空白，浏览器控制台报错：

```
模块“https://your-custom-domain.com/RepoName/assets/index-abc123.js” 
被阻止加载，因为它使用了不允许的 MIME 类型（“text/html”）。
```

页面无法正常渲染，所有 JavaScript 逻辑均未执行。

## 环境背景

- **前端框架**：Vue 3 + Vite
- **部署方式**：GitHub Actions 构建并部署到 GitHub Pages
- **域名**：配置了自定义域名（如 `example.com`），而非使用 `username.github.io` 默认域名
- **仓库名称**：假设为 `RepoName`

## 问题复现步骤

1. 使用 `npm create vite@latest` 创建 Vite 项目
2. 配置 `vite.config.ts`，设置 `base: '/RepoName/'`
3. 编写 GitHub Actions 工作流，构建并部署到 GitHub Pages
4. 在仓库 Settings → Pages 中配置自定义域名
5. 访问自定义域名，打开浏览器开发者工具，发现 JS 模块加载失败

## 根因分析

### Vite 的 base 配置

Vite 的 `base` 配置决定了构建产物中所有静态资源的引用路径。例如：

```ts
// vite.config.ts
export default defineConfig({
  base: '/RepoName/',  // 所有资源路径将以此开头
})
```

构建后，`index.html` 中引用的 JS 文件路径会变成：

```html
<script type="module" src="/RepoName/assets/index-abc123.js"></script>
```

这设计是为了兼容 GitHub Pages **项目页面** 的路径结构。项目页面的访问地址格式为 `https://username.github.io/RepoName/`，因此资源路径 `/RepoName/assets/...` 恰好能正确映射到文件系统。

### GitHub Pages 自定义域名的路径行为

问题出在**配置了自定义域名之后**。

GitHub Pages 项目页面有两种访问方式：

| 访问方式   | URL                                    | 资源根路径   |
| ---------- | -------------------------------------- | ------------ |
| 默认域名   | `https://username.github.io/RepoName/` | `/RepoName/` |
| 自定义域名 | `https://example.com/`                 | `/`          |

当你在仓库 Settings → Pages 中配置了自定义域名（如 `example.com`）后，GitHub Pages 会将这个域名的根路径（`/`）映射到你的项目内容。此时：

- 访问 `https://example.com/` → GitHub Pages 返回 `index.html` ✅
- 浏览器解析到 `<script src="/RepoName/assets/index-abc123.js">` → 请求 `https://example.com/RepoName/assets/index-abc123.js`
- GitHub Pages 在根路径（`/`）下找不到 `/RepoName/` 目录 → 返回 404 页面（HTML）
- 浏览器期望收到 JavaScript，却收到 `text/html` → **拒绝执行，报 MIME 类型错误** ❌

### 关键结论

**`base` 路径与服务器实际文件路径不一致，是导致问题的根本原因。** 自定义域名下，GitHub Pages 将域名根路径作为服务根路径，但 Vite 构建的资源路径仍按项目子路径生成，两者不匹配。

## 解决方案

### 方案一：修改 base 为根路径（推荐）

将 `vite.config.ts` 中的 `base` 改为 `'/'`：

```ts
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  base: '/',  // 改为根路径，适配自定义域名
})
```

构建后，资源路径变为：

```html
<script type="module" src="/assets/index-abc123.js"></script>
```

浏览器请求 `https://example.com/assets/index-abc123.js`，GitHub Pages 能在根路径下找到该文件，正确返回 `application/javascript` MIME 类型。

### 方案二：动态判断 base（双域名兼容）

如果你需要同时支持默认域名和自定义域名（例如过渡期），可以在构建时动态设置 `base`：

```ts
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  // 通过环境变量动态设置 base
  base: process.env.BASE_PATH || '/',
})
```

在 GitHub Actions 中，如果需要部署到项目页面路径，可以传环境变量：

```yaml
- name: Build
  run: npm run build
  env:
    BASE_PATH: /RepoName/
```

但通常情况下，**推荐直接使用方案一**，因为 GitHub Pages 配置自定义域名后会 301 重定向默认域名到自定义域名，用户访问默认域名的流量会自动转到自定义域名。

### 方案三：使用 `actions/deploy-pages`（官方推荐部署方式）

确保 GitHub Actions 工作流使用官方推荐方式部署：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
      - name: Install dependencies
        run: npm ci
      - name: Build
        run: npm run build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

  deploy:
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

这种方式由 GitHub 官方处理部署逻辑，与自定义域名兼容性更好。

## 验证方法

修复后，按以下步骤验证：

1. **本地构建验证**：运行 `npm run build`，检查 `dist/index.html` 中的资源路径是否为绝对根路径：

   ```html
   <script type="module" src="/assets/index-abc123.js"></script>
   ```

   确认路径中**不包含**仓库名称前缀。

2. **部署后验证**：推送代码触发 GitHub Actions 部署，部署完成后：

   - 打开浏览器开发者工具 → Network 面板
   - 刷新页面，检查 JS 文件的请求 URL 和响应头
   - 确认 `Content-Type` 为 `application/javascript`，而非 `text/html`
   - 确认 HTTP 状态码为 200，而非 404

3. **功能验证**：确认页面正常渲染，所有交互功能可用。

## 常见变体

### 问题一：使用 `history` 路由模式（SPA 路由）

如果使用 Vue Router / React Router 的 `history` 模式，除了上述 `base` 配置外，还需要在 `public/` 目录下添加 `404.html` 或配置 GitHub Pages 的自定义 404 页面，因为 GitHub Pages 不支持服务端回退。

最简单的做法是使用 `hash` 路由模式：

```ts
// Vue Router
const router = createRouter({
  history: createWebHashHistory(),  // 使用 hash 模式
  routes: [...],
})
```

### 问题二：静态资源（图片、字体）也 404

与 JS 模块同理，`base` 配置也影响图片、字体等静态资源的加载路径。确保 `base` 配置正确，即可一并解决。

## 总结

| 关键点   | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| 错误原因 | Vite 的 `base` 路径与 GitHub Pages 自定义域名下的实际文件路径不一致 |
| 核心修复 | `base: '/RepoName/'` → `base: '/'`                           |
| 根本原理 | 自定义域名下 GitHub Pages 以根路径 `/` 服务，而非项目子路径  |
| 验证方式 | 检查构建产物 `index.html` 中的资源路径 + 浏览器 Network 面板确认 200 |

GitHub Pages 配合自定义域名是一个很常见的部署方案，而 Vite 的 `base` 配置则是这个组合中最容易踩坑的地方。理解路径映射关系后，问题其实很简单——**确保构建产物的资源引用路径与服务器上的实际文件路径一致即可**。







