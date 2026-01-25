---
title: Windows 11 双 GitHub 账号使用指南
description: 本文介绍如何在Windows 11上使用双 GitHub 账号。
date: 2026-01-25T20:34:25-08:00
draft: false
categories:
- 开发
tags:
- GitHub
- Windows
---

# Windows 11 双GitHub账号使用指南

## 前言

你是否和我一样，有两个 GitHub 账号？一个用于协作团队项目，另一个用于个人创作。想让它们在本地和远端完全隔离，互不干扰？

问题是，Git 默认用全局 `user.name` 和 `user.email`，这会让提交记录看起来混淆身份。更复杂的是，SSH 密钥是权限的关键，每个密钥只能归属于一个账号。电脑默认使用一个 SSH key 连接 github.com，也导致只能无密码访问其中一个账号。

## 双账号配置

分别下载一个 GitHub Desktop 和 UGit，然后登录相应的账号，再通过 batch 脚本切换账号。

### 通过.bat文件实现手动切换全局配置

下面这个简单的批处理脚本可以让用户输入账户名，然后根据不同的账户名设置对应的全局配置。例如：

```bat
@echo off
setlocal

:: 提示输入账户名称
set /p account=请输入账户名（比如：Account1 或 Account2）:

:: 先清除已有的 user.name 和 user.email
git config --global --unset user.name
git config --global --unset user.email

:: 根据不同账户设置
if /i "%account%"=="Account1" (
    git config --global user.name "你的账户1名字"
    git config --global user.email "youraccount1@example.com"
) else if /i "%account%"=="Account2" (
    git config --global user.name "你的账户2名字"
    git config --global user.email "youraccount2@example.com"
) else (
    echo 未识别的账户名，请确保输入正确。
)

pause
endlocal
```

这样运行脚本后，你会看到提示输入账户名，然后它会根据你的输入设置对应的全局git配置。你可以根据需要调整账户名和邮箱。
## 使用批处理文件自动提交git代码

我需要一个Windows 11下的批处理（batch）脚本，实现清空并重新设置Git提交身份（用户名/邮箱）、切换到指定目录、自动以时间戳为提交信息完成Git代码提交，同时补充完善的异常处理和操作反馈。

### 实现思路
1. 先清空Git全局用户名/邮箱配置
2. 设置新的Git提交身份信息
3. 切换到指定工作目录并做合法性检查
4. 执行Git拉取（避免冲突）、添加、提交、推送操作
5. 用系统时间生成时间戳作为提交信息
6. 增加完善的错误处理和操作反馈
7. 脚本执行完成后暂停，方便查看执行结果

### 完整Batch脚本
```batch
@echo off
chcp 65001 >nul 2>&1  :: 设置编码为UTF-8，避免中文乱码
setlocal enabledelayedexpansion

:: ===================== 配置区域（请根据实际需求修改）=====================
:: 新的Git用户名
set "GIT_USER_NAME=your_username"
:: 新的Git邮箱
set "GIT_USER_EMAIL=your_email@example.com"
:: 目标Git仓库目录（请替换为你的实际目录）
set "GIT_WORK_DIR=D:\your_project_directory"
:: 可选：Git远程仓库名称（默认origin）
set "GIT_REMOTE=origin"
:: 可选：Git分支名称（默认main，也可改为master）
set "GIT_BRANCH=main"
:: =========================================================================

echo ===================== Git自动提交脚本 =====================
echo 运行环境：Windows 11
echo 执行时间：%date% %time%
echo ==========================================================
echo.

:: 1. 清空Git全局用户名和邮箱配置
echo [1/6] 清空Git全局身份配置...
git config --global --unset user.name
git config --global --unset user.email
if !errorlevel! equ 0 (
    echo ✅ 已清空Git全局用户名和邮箱配置
) else (
    echo ⚠️  清空配置时出现警告（可能无原有配置），继续执行...
)
echo.

:: 2. 设置新的Git用户名和邮箱
echo [2/6] 设置新的Git身份信息...
git config --global user.name "%GIT_USER_NAME%"
if !errorlevel! equ 0 (
    echo ✅ 已设置Git用户名：%GIT_USER_NAME%
) else (
    echo ❌ 设置用户名失败，脚本终止！
    pause
    exit /b 1
)

git config --global user.email "%GIT_USER_EMAIL%"
if !errorlevel! equ 0 (
    echo ✅ 已设置Git邮箱：%GIT_USER_EMAIL%
) else (
    echo ❌ 设置邮箱失败，脚本终止！
    pause
    exit /b 1
)
echo.

:: 3. 切换到目标Git仓库目录并检查合法性
echo [3/6] 切换到Git仓库目录...
if not exist "%GIT_WORK_DIR%" (
    echo ❌ 目标目录不存在：%GIT_WORK_DIR%
    pause
    exit /b 1
)

cd /d "%GIT_WORK_DIR%"
if !errorlevel! neq 0 (
    echo ❌ 切换到目录失败：%GIT_WORK_DIR%
    pause
    exit /b 1
)
echo ✅ 已切换到目录：%GIT_WORK_DIR%
echo.

:: 4. 检查是否为Git仓库
echo [4/6] 检查Git仓库合法性...
git rev-parse --is-inside-work-tree >nul 2>&1
if !errorlevel! neq 0 (
    echo ❌ 当前目录不是Git仓库，请检查路径！
    pause
    exit /b 1
)
echo ✅ 确认是合法的Git仓库
echo.

:: 5. 拉取远程最新代码（避免冲突）
echo [5/6] 拉取远程[%GIT_BRANCH%]分支最新代码...
git pull %GIT_REMOTE% %GIT_BRANCH%
if !errorlevel! equ 0 (
    echo ✅ 拉取远程代码成功
) else (
    echo ⚠️  拉取代码出现警告（可能无远程分支或已最新），继续执行...
)
echo.

:: 6. 提交代码（生成时间戳作为提交信息）
echo [6/6] 执行Git提交和推送...
:: 生成格式化时间戳（格式：YYYY-MM-DD_HH-MM-SS）
for /f "tokens=1-2 delims=/ " %%a in ("%date%") do (
    set "month=%%a"
    set "day=%%b"
)
for /f "tokens=3 delims=/ " %%c in ("%date%") do set "year=%%c"
for /f "tokens=1-2 delims=:. " %%d in ("%time%") do (
    set "hour=%%d"
    set "minute=%%e"
)
for /f "tokens=3 delims=:. " %%f in ("%time%") do set "second=%%f"
:: 补零处理（确保单数字时间为两位）
if !hour! lss 10 set "hour=0!hour!"
if !month! lss 10 set "month=0!month!"
if !day! lss 10 set "day=0!day!"
set "COMMIT_MSG=Auto commit at !year!-!month!-!day!_!hour!-!minute!-!second!"

:: 添加所有修改的文件
git add .
if !errorlevel! equ 0 (
    echo ✅ 已添加所有修改的文件到暂存区
) else (
    echo ⚠️  添加文件时出现警告（可能无修改），继续执行...
)

:: 提交代码
git commit -m "!COMMIT_MSG!"
if !errorlevel! equ 0 (
    echo ✅ 提交成功，提交信息：!COMMIT_MSG!
    :: 推送代码到远程
    git push %GIT_REMOTE% %GIT_BRANCH%
    if !errorlevel! equ 0 (
        echo ✅ 推送代码到远程[%GIT_REMOTE%/%GIT_BRANCH%]成功
    ) else (
        echo ❌ 推送代码失败，请检查网络或分支权限！
        pause
        exit /b 1
    )
) else (
    echo ⚠️  无代码修改，无需提交
)
echo.

echo ==========================================================
echo 🎉 脚本执行完成！
echo 执行时间：%date% %time%
echo ==========================================================

pause
endlocal
```

### 脚本关键部分解释
1. **配置区域**
   - 你需要修改`GIT_USER_NAME`、`GIT_USER_EMAIL`、`GIT_WORK_DIR`这三个核心配置，其他配置可根据仓库实际情况调整（如分支名）。
   - `chcp 65001`：设置UTF-8编码，避免中文提交信息或目录名乱码。

2. **Git身份配置**
   - 先执行`--unset`清空原有配置，再用`--global`设置新的全局用户名/邮箱，确保提交身份准确。
   - 增加`errorlevel`检查，失败时终止脚本并提示。

3. **目录与仓库检查**
   - 先验证目标目录是否存在，再切换目录，最后检查是否为Git仓库（通过`git rev-parse --is-inside-work-tree`），避免操作非Git目录。

4. **时间戳生成**
   - 解析系统`%date%`和`%time%`变量，格式化为`YYYY-MM-DD_HH-MM-SS`格式，确保提交信息唯一且易识别。
   - 对单数字的时间（如9点→09点）做补零处理，保证格式统一。

5. **Git操作流程**
   - `git pull`：先拉取远程最新代码，避免本地与远程冲突。
   - `git add .`：添加所有修改/新增的文件到暂存区。
   - `git commit`：用时间戳作为提交信息。
   - `git push`：推送到远程仓库。

6. **错误处理与反馈**
   - 每个步骤都有明确的成功/失败提示（✅/❌/⚠️），便于排查问题。
   - 非致命错误（如无原有配置、无代码修改）给出警告并继续执行，致命错误（如目录不存在）终止脚本。

### 使用说明
1. **修改配置**：打开脚本，找到「配置区域」，替换为你的实际信息：
   - `GIT_USER_NAME`：你的Git用户名（如github用户名）。
   - `GIT_USER_EMAIL`：你的Git绑定邮箱。
   - `GIT_WORK_DIR`：Git仓库的本地目录（如`D:\projects\my_poem_project`）。

2. **运行脚本**：
   - 将脚本保存为`git_auto_commit.bat`。
   - 双击运行，或在CMD/PowerShell中执行：`git_auto_commit.bat`。

3. **注意事项**：
   - 确保Windows 11已安装Git并配置到系统环境变量（可在CMD中执行`git --version`验证）。
   - 首次推送可能需要手动输入Git账号密码，或配置SSH密钥免密登录。
   - 脚本执行后会暂停，可查看所有操作日志，按任意键退出。

### 总结
1. 脚本实现了「清空Git身份→设置新身份→切换目录→检查仓库→拉取代码→提交推送」的完整流程，适配Windows 11。
2. 提交信息采用格式化时间戳，确保唯一性和可读性。
3. 包含完善的错误处理和可视化反馈，新手也能轻松排查问题。
4. 配置区域与执行逻辑分离，只需修改少量参数即可适配不同项目。

## 参考资料和链接

- [GitHub 官方 SSH 文档](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [Win11 配置多个 GitHub 账号](https://www.chengxulvtu.net/how-to-work-with-multiple-github-accounts-on-win11/)
- [Git 配置教程](https://git-scm.com/book/zh/v2/)









