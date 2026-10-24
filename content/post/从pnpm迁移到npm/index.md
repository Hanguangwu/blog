---
title: 如何从 pnpm 还原使用 npm ？
description: 本文介绍pnpm到npm的切换。
date: 2025-12-28T12:34:25+08:00
draft: false
categories:
- 开发
tags:
- nodejs
---

# 如何从 pnpm 还原使用 npm ？

[如何从 pnpm 还原使用 npm ？](https://www.levenx.com/issues/how-to-revert-from-pnpm-to-npm/531241811290)

理解各种包管理器及其互操作性是很重要的。从 pnpm 还原到 npm，主要步骤如下：
### 步骤1: 清理 `pnpm` 环境

首先，确保删除或清理由 pnpm 创建的 `node_modules` 文件夹和 `pnpm-lock.yaml` 文件。这是因为 pnpm 和 npm 在处理依赖和锁文件方面存在差异，直接使用可能会导致冲突或错误。

`rm -rf node_modules rm pnpm-lock.yaml`

### 步骤2: 初始化 `npm`

如果项目中不存在 `package.json` 文件（一般情况下不会发生，除非是全新项目），你需要运行 `npm init` 来创建一个。如果已经有 `package.json`，则可以直接跳至下一步。

`npm init`

### 步骤3: 安装依赖

使用 `npm` 安装项目所需要的依赖。如果你有 `package.json` 文件，npm 会根据此文件安装所有列出的依赖。

`npm install`

这个命令会创建一个 `node_modules` 文件夹和一个 `package-lock.json` 文件，这是 npm 用来锁定依赖版本的。

### 步骤4: 测试项目

在转换完成后，确保运行项目的测试，以验证所有依赖都正确安装，项目可以正常运行。

`npm test`

### 示例

假设我之前使用 pnpm 管理一个 Node.js 项目，该项目依赖于 Express 和 React。转换过程中，我会首先删除由 pnpm 创建的锁文件和 `node_modules` 文件夹，然后使用 npm 重新安装依赖，并确保通过所有测试。

这个过程保证了从一个包管理器到另一个包管理器的平滑过渡，同时确保项目的稳定性和一致性。

### 结论

虽然 pnpm 提供了优异的性能和磁盘空间优化，但在某些团队或项目中可能需要统一使用 npm。以上步骤可以帮助实现从 pnpm 到 npm 的无缝过渡。












