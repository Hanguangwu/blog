---
title: 新文章总是不见？一次 Hugo + GitHub Pages 的完整排查与修复
description: 推送成功、Pages 构建也成功，但刚发布的文章要等下一篇才能看到。本文从「部署落后一篇」的现象出发，定位到 front matter 时区写错导致文章被 Hugo 当作未来内容丢弃，并顺带修掉了 CI 用标准版 Hugo 编不了 SCSS、构建产物被提交进仓库等一连串隐藏问题。
date: 2026-10-24T11:10:00+08:00
draft: false
categories:
- 编程
- 技术科普
tags:
- Hugo
- GitHub Actions
- GitHub Pages
- 博客
---

## 现象

博客是 Hugo + Stack 主题，源码在 `main` 分支，GitHub Actions 构建后把 `public/` 推到 `gh-pages`，再由 GitHub Pages 对外提供。

问题很稳定，也很诡异：

- 新增文章 → 推送 → Actions 里 `pages build and deployment` 显示成功；
- 打开网站，**刚推的文章不在**；
- 等下一次发新文章时，上一篇文章突然出现了。

也就是说，线上内容永远**落后一次提交**。让人以为「站点更新有延迟」，但刷新、强制刷新、无痕窗口都没用。

## 第一步：先确认是构建问题还是缓存问题

怀疑过 CDN 缓存、浏览器缓存、Pages 发布延迟，但都可以排除：`gh-pages` 分支的 `deploy:` 提交时间就在推送后一分钟内，说明部署链路是通的。

于是直接比对「`main` 的最新提交」和「对应那次 deploy 生成的首页」：

```powershell
# 每次 deploy commit 的信息
git log origin/gh-pages --format="%h %ad %s" --date=iso -5

# 取出某次 deploy 生成的首页里最新的一篇文章
git show <deploy-commit>:index.html |
  Select-String -Pattern 'article-title.*?<a [^>]*>([^<]+)</a>' -AllMatches
```

结果非常整齐：

| gh-pages 的 deploy 提交 | 构建时间（UTC） | 首页最新文章 | 说明 |
| --- | --- | --- | --- |
| `aeb8bfae` ← `c14784be` | 10-23 11:30 | 10-12 的文章 | 缺 10-23 那两篇 |
| `2e9dd87d` ← `1c614ea9` | 09-19 09:09 | 美国PhD | 缺 09-19 的内存池 |
| `9c6b4052` ← `7524a8bd` | 10-12 15:51 | 内存池 | 缺 10-12 那两篇 |

每一次 deploy 对应的提交号都是对的，但**生成出来的首页都比那次提交少一篇最新的**。这就不是部署问题了，是 Hugo 在构建时主动把最新那篇排除掉了。

## 第二步：为什么 Hugo 会丢掉最新的文章

看那两篇「消失」文章的前置元数据：

```yaml
---
title: 从黑白方块到信息世界：二维码的原理、历史与 Python 实战
date: 2026-10-23T14:34:25-08:00
draft: false
---
```

提交时间是 `2026-10-23 19:29 +0800`（即 `11:29 UTC`），而 front matter 里的时间是 `2026-10-23T14:34:25-08:00`，换算成 UTC 是 **`22:34 UTC`**。

结论：**这篇文章的发布时间比构建时间晚了 11 个小时**。

原因是时区写错了。文章生成工具把**本地 +08:00 的钟表时间**（14:34）套上了 **`-08:00` 的时区后缀**，于是同一个钟表时间被解释成一个晚了 16 小时的时刻。

而 Hugo 默认 `buildFuture = false`，**发布时间在未来的内容不会被渲染**。所以：

1. 推送到构建只有几分钟，文章时间还在“未来” → 被丢弃；
2. 下一次发文章时（通常隔了几小时甚至几天），上一篇的时间已经变成过去 → 它才被渲染出来。

这正好解释了「永远落后一篇」这个现象。

顺带一提，全站有 **143 篇**文章都是 `-08:00`，属于系统性的模板问题，不是某一篇手滑。

## 第三步：修复时区

思路：**只改时区后缀，不动钟表时间**。这样每篇文章的发布时刻统一提前 16 小时，文章之间的相对顺序完全不变，也不会再出现“未来”时间。

用 PowerShell 批量处理（只替换 front matter 里 `date:` 行的时区，且保持文件为 UTF-8 无 BOM）：

```powershell
$enc = New-Object System.Text.UTF8Encoding($false)
$changed = 0

Get-ChildItem -Path content/post -Recurse -Filter index*.md | ForEach-Object {
    $p    = $_.FullName
    $raw  = [System.IO.File]::ReadAllBytes($p)
    $text = [System.Text.Encoding]::UTF8.GetString($raw)
    if ($raw.Length -ge 3 -and $raw[0] -eq 0xEF) { $text = $text.Substring(1) }  # 去掉 BOM

    # 仅匹配行首的 date: ...-08:00
    $new = [regex]::Replace($text, '(?m)^(date:[ \t]*[^\r\n]*?)-08:00', '$1+08:00')

    if ($new -ne $text) {
        [System.IO.File]::WriteAllText($p, $new, $enc)
        $changed++
    }
}
"changed: $changed"
```

改完一定要核对，确认**只动了 `date` 行**：

```powershell
git diff -U0 -- content
```

## 第四步：让工作流本身更抗造

只改历史数据不够，得让「时区写错也不会再丢文章」。同时把工作流里几个隐患一起处理：

```yaml
# 连续两次推送时，避免旧构建覆盖新构建
concurrency:
  group: hugo-deploy
  cancel-in-progress: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Git checkout
        uses: actions/checkout@v6
        with:
          submodules: true
          fetch-depth: 1

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: '0.157.0'
          extended: true          # Stack 主题要编译 SCSS，必须 extended

      - name: Build
        run: hugo --theme=hugo-theme-stack --gc --minify --buildFuture --cleanDestinationDir

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

几处改动的理由：

- `--buildFuture`：兜底。即使以后时区又写错，文章也不会被静默丢弃；
- `--cleanDestinationDir`：清掉 `public/` 里上一次构建残留的文件，避免旧页面被一起发布；
- **去掉 `--buildDrafts`**：它会把 `draft: true` 的草稿也公开出去（本站当时就有一篇草稿被发布到了线上）；
- `concurrency`：两次推送挨得很近时，让后一次构建取消前一次，防止旧产物最后落地。

## 第五步：修完时区，CI 反而挂了

改完推送，构建直接失败：

```
ERROR error building site: render: [en v1.0.0 guest] failed to render pages:
render of "/404" failed: ... head/style.html:3:38 ...
TOCSS: failed to transform "/scss/style.scss" (text/x-scss).
Check your Hugo installation; you need the extended version to build SCSS/SASS
with transpiler set to 'libsass'.
```

Stack 主题的样式是这样编译的：

```go-html-template
{{ $sass := resources.Get "scss/style.scss" }}
{{ $style := $sass | toCSS | minify | resources.Fingerprint "sha256" }}
```

`toCSS` 默认走 libsass，而 libsass **只存在于 extended / extended-deploy 版 Hugo**。CI 里 `hugo-version: '0.157.0'` 装的是标准版，所以编不了 SCSS。

但问题来了：**之前几个月为什么一直是好的？**

因为仓库里提交了 `resources/_gen/` —— Hugo 的资源缓存，里面躺着已经编译好的 CSS：

```
resources/_gen/assets/scss/style.scss_77b10c8e....content   # 编译产物
resources/_gen/assets/scss/style.scss_77b10c8e....json      # 元信息
```

CI 每次都命中缓存，直接复用 CSS，**根本没执行过 SCSS 编译**，于是“标准版 Hugo 不够用”这个事实被缓存盖住了。这次清理构建产物时把 `resources/_gen` 从版本库移除，缓存没了，问题立刻现形。

本地能构建成功，是因为本机装的是 `hugo ... +extended`（`hugo version` 里带 `+extended`）。

修法只有一行：

```yaml
with:
  hugo-version: '0.157.0'
  extended: true
```

顺带说明：Hugo 从 **v0.153** 起已把内置 libsass 标记为 deprecated，未来版本会移除。到那时需要改成 Dart Sass —— 在 CI 里安装 Dart Sass，并在项目的 `layouts/partials/head/style.html` 覆盖主题模板，传入 `transpiler: dartsass`。现在用 extended 是最小、正确的修法。

## 顺手清理的几处历史包袱

**1. 构建产物被提交进了仓库**

`public/`（214 个文件）和 `resources/_gen/` 都在版本库里。后果是：本地/CI 的 `public/` 里混着历史主题的残留文件，`hugo` 不会删它们，`actions-gh-pages` 会把整个目录推上去。于是线上长期存在 `about-hugo`、`ar`、`contact`、`posts`、`ts`、`zh-cn` 这类早就没用的目录。

处理：

```powershell
git rm -r --cached public resources/_gen
```

```gitignore
# .gitignore
public/
resources/_gen/
.hugo_build.lock
```

配合 `--cleanDestinationDir`，线上就只剩当前源码真正生成的内容。

**2. favicon 404**

配置里写的是：

```yaml
favicon: /blog/favicon.ico
```

但站点部署在域名根路径（`https://blog.zwzhang.com/`），`/blog/` 下并没有这个文件，控制台就一直报 `GET /blog/favicon.ico 404`。改成：

```yaml
favicon: /favicon.ico   # 对应 static/favicon.ico
```

**3. 控制台那堆 `MaxListenersExceededWarning` 不用管**

```
MaxListenersExceededWarning: Possible EventEmitter memory leak detected.
11 end listeners added. ...
ObjectMultiplex - orphaned data for stream "background-liveness"
ObjectMultiplex - orphaned data for stream "metamask-multichain-provider"
moz-extension://.../scripts/contentscript.js
```

全部来自浏览器里的 **MetaMask 扩展**（`moz-extension://` 协议已经说明了一切），和站点无关。禁用该扩展，警告立刻消失。

## 复盘：几个值得记住的点

1. **“部署成功”不等于“内容正确”**。Pages 的 `pages build and deployment` 成功只说明 `gh-pages` 被发布出去了，至于里面的内容是不是最新的，它不负责。排查时一定要比对**产物里的内容**，而不是只看绿色的对勾。
2. **Hugo 默认不渲染未来时间的内容**（`buildFuture = false`），`--buildDrafts` 管的是草稿，管不了未来时间。
3. **时区后缀必须和钟表时间匹配**。`2026-10-23T14:34:25-08:00` 和 `2026-10-23T14:34:25+08:00` 差了 16 小时，遇到“当天发布当天不生效”的问题，先算这个。
4. **构建缓存会掩盖配置错误**。`resources/_gen` 一旦提交，CI 里就可能永远走不到真正的编译路径，出问题时现象会非常费解。构建产物不该进版本库。
5. **主题用 SCSS 就必须用 extended 版 Hugo**，在 CI 里显式写 `extended: true`，别依赖缓存侥幸通过。
6. 新建文章时直接写成本地时区，例如：

   ```yaml
   date: 2026-10-24T11:10:00+08:00
   ```

   比事后批量修 143 篇要省事得多。
