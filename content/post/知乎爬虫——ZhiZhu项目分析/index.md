---
title: 知乎爬虫——ZhiZhu项目分析
description: 本文介绍知乎爬虫——ZhiZhu项目。
date: 2026-03-05T12:34:25-08:00
draft: false
categories:
- APP
- GitHub
tags:
- 爬虫
- Zhihu
---

# 知乎爬虫——ZhiZhu项目分析


## 简介

[ZhiZhu (知蛛) ——爬取指定知乎用户的所有回答和所有文章，以高保真 Markdown 格式保存到本地。 ](https://github.com/HeroBlast10/ZhiZhu)
  
> *"我们在互联网上留下的每一个字节，都是灵魂的碎片。是时候把它们找回来了。"*  
  
### 为什么是 ZhiZhu？  
  
多年来，你在知乎上敲下的数万、数十万字，不仅仅是简单的问答。它们是你**知识体系的疆界**，是你**价值观的投影**，是你**逻辑思维的演变轨迹**，更是你在这个喧嚣世界中发出的**独特声音**。  
  
然而，散落在互联网角落的数据是脆弱的，也是割裂的。  
  
**ZhiZhu (知蛛)** 是一台时光穿梭机，也是一位数字考古学家。它致力于将你流落在知乎的精神财富，以最纯粹、最通用的 Markdown 格式完整带回本地。无论是精妙的 LaTeX 数学公式，还是承载记忆的图片，都力求高保真还原。  
  
> *Archiving the past to compute the future.*  
> *归档过去，为了计算未来。*  
  
---  
  
### 在 AI 时代，这不仅仅是备份  
  
当你拥有了这份属于自己的全量数据，你就拥有了训练私人 AI 助理的基石。你可以将这些代表你 **"人格（Personality）"** 的文本投喂给 LLM，让 AI 成为你的镜子——  
  
- **自我复盘** — 利用 AI 分析你过去数年的回答，生成你的「认知演变报告」  
- **知识图谱** — 从散乱的回答中提取你的知识结构，重组为系统化的文章  
- **风格克隆** — 让 AI 学习你的文风与逻辑，成为最懂你的写作助手  
- **深度对话** — 与你的「数字孪生」对话，完成一次深度的自我探索与反省  
  
**把数据存成 Markdown，只是第一步；认识你自己，才是 ZhiZhu 的终极目标。**  
  
> *Don't just leave it on the cloud. Own your thoughts.*  
> *别把思想只留在云端，拥有它。*  
  
---  
  
### 核心能力  
  
| 能力 | 说明 |  
|:---|:---|  
| **用户级全量爬取** | 输入用户 URL token，自动收集该用户的全部回答与文章 |  
| **用户想法爬取** | 爬取指定用户的所有知乎想法（Pins） |  
| **问题级爬取** | 爬取指定问题下的全部或前 N 个回答 |  
| **单回答爬取** | 精准爬取某个特定回答，可选附带完整评论区 |  
| **评论区提取** | 通过知乎 API 获取全部根评论与子评论，格式化为 Markdown |  
| **浏览器指纹伪装** | 内置 WebGL、Canvas、AudioContext 等多维度反检测机制 |  
| **智能延迟策略** | 请求间随机等待 10-20 秒（可自定义），以时间换安全 |  
| **断点续传** | 自动记录进度，中断后重新运行即从上次位置继续 |  
| **LaTeX 公式还原** | 完美转换知乎数学公式为标准 `$...$` / `$$...$$` 语法 |  
| **图片本地化** | 自动下载文章图片到本地，重写 Markdown 引用路径 |  
| **内容去噪** | 自动去除广告卡片、视频占位符、知乎直答链接等干扰元素 |  
| **持久化登录** | 登录一次，后续爬取无需重复登录 |  
  
---

## 重要代码分析
### main.py中argparse参数解析

```python
def _add_common_args(parser: argparse.ArgumentParser):
    """为子命令添加公共参数。"""
    parser.add_argument(
        "--output", "-o",
        type=str,
        default=None,
        help="输出目录路径",
    )
```

```python
	subparsers = parser.add_subparsers(dest="command", help="可用命令")

    # ── login 子命令 ──
    login_parser = subparsers.add_parser("login", help="登录知乎（手动登录，保存 Cookie）")
    login_parser.add_argument(
        "--timeout",
        type=int,
        default=300,
        help="等待登录的超时时间（秒），默认 300",
    )
```

### webui.py中的通用流式日志生成器

将 fn() 产生的所有 print() 输出实时 yield 给 Gradio Textbox。

```python
# ── stdout 重定向 ──────────────────────────────────────────────

class _QueueWriter:
    """将 print() 输出收集到 queue.Queue，供生成器逐行 yield 给 Gradio。"""

    def __init__(self, q: queue.Queue) -> None:
        self._q = q
        self._buf = ""

    def write(self, text: str) -> None:
        self._buf += text
        while "\n" in self._buf:
            line, self._buf = self._buf.split("\n", 1)
            self._q.put(line)

    def flush(self) -> None:
        if self._buf:
            self._q.put(self._buf)
            self._buf = ""

    def fileno(self) -> int:
        return sys.__stdout__.fileno()


def _run_in_thread(fn, q: queue.Queue) -> None:
    """在子线程中执行 fn()，完成后向队列推送 None（结束信号）。"""
    old_stdout = sys.stdout
    sys.stdout = _QueueWriter(q)  # type: ignore[assignment]
    try:
        fn()
    except Exception as e:
        sys.stdout.write(f"\n[ERROR] {e}\n")  # type: ignore[union-attr]
    finally:
        sys.stdout.flush()  # type: ignore[union-attr]
        sys.stdout = old_stdout
        q.put(None)  # 结束信号


def _stream_logs(fn) -> Generator[str, None, None]:
    """
    通用流式日志生成器。
    将 fn() 产生的所有 print() 输出实时 yield 给 Gradio Textbox。
    """
    q: queue.Queue = queue.Queue()
    t = threading.Thread(target=_run_in_thread, args=(fn, q), daemon=True)
    t.start()
    log_lines: list[str] = []
    while True:
        item = q.get()
        if item is None:
            break
        log_lines.append(item)
        yield "\n".join(log_lines)
    t.join()
    yield "\n".join(log_lines)
```

### tui.py中的终端交互

提供基于 Textual 的 TUI 界面，覆盖 ZhiZhu 的全部功能。

不同面板对应不同的功能，使用stdout 重定向输出日志。

```python
# ── 任务分发 ───────────────────────────────────────────────────

def _dispatch(key: str, params: dict[str, Any]) -> None:
    """根据功能 key 调用对应的 scraper/merge 函数（在 Worker 线程中执行）。"""

    if key == "login":
        from scraper import login
        asyncio.run(login(**params))

    elif key == "scrape_user":
        from scraper import scrape_user
        asyncio.run(scrape_user(**params))

    elif key == "scrape_pins":
        from scraper import scrape_user_pins
        asyncio.run(scrape_user_pins(**params))

    elif key == "scrape_question":
        from scraper import scrape_question
        asyncio.run(scrape_question(**params))

    elif key == "scrape_answer":
        from scraper import scrape_single_answer
        asyncio.run(scrape_single_answer(**params))

    elif key == "merge":
        from merge_md import merge
        merge(**params)
```

### merge_md.py中的文本预处理

```python
# ── 工具函数 ──────────────────────────────────────────────────

def collect_md_files(root: Path) -> list[Path]:
    """
    从指定根目录中收集所有 .md 文件。
    支持两种结构：
      - <root>/<子文件夹>/index.md
      - <root>/<文件>.md
    """
    files: list[Path] = []
    for path in root.rglob("*.md"):
        files.append(path)
    return files


def extract_date_from_header(text: str) -> str:
    """
    从 Markdown 文件头部的元信息中提取日期字符串（YYYY-MM-DD）。
    格式如：> **日期**: 2025-12-24
    找不到时返回空字符串。
    """
    m = re.search(r'>\s*\*\*日期\*\*:\s*(\d{4}-\d{2}-\d{2})', text[:600])
    return m.group(1) if m else ""


def sort_key_by_date(path: Path) -> tuple[str, str]:
    """按文件头部日期排序，日期相同时按文件路径排序。"""
    try:
        text = path.read_text(encoding="utf-8")
    except Exception:
        text = ""
    date = extract_date_from_header(text)
    return (date, str(path))


def sort_key_by_name(path: Path) -> str:
    """按文件路径字母顺序排序。"""
    return str(path)
```

### stealth.py — 反检测 JavaScript 注入模块  
  
参考 hg3386628/zhihu-scraper 和 yuchenzhu-research/zhihu-scraper 的反爬策略。  

集成 WebGL、Canvas、AudioContext 指纹伪装以及 navigator.webdriver 覆盖。

```python
STEALTH_JS = """
// 覆盖 navigator.webdriver
Object.defineProperty(navigator, 'webdriver', {
    get: () => undefined,
    configurable: true
});

// 添加 Chrome 浏览器特有属性
window.chrome = {
    runtime: {},
    loadTimes: function() {},
    csi: function() {},
    app: {}
};

// 覆盖 Permissions API
const originalQuery = window.navigator.permissions.query;
window.navigator.permissions.query = (parameters) => (
    parameters.name === 'notifications' ?
    Promise.resolve({state: Notification.permission}) :
    originalQuery(parameters)
);

// 伪造 WebGL 渲染器信息
const getParameter = WebGLRenderingContext.prototype.getParameter;
WebGLRenderingContext.prototype.getParameter = function(parameter) {
    if (parameter === 37445) return 'Intel Inc.';
    if (parameter === 37446) return 'Intel Iris OpenGL Engine';
    return getParameter.apply(this, [parameter]);
};

// 随机化 Canvas 指纹
const origToDataURL = HTMLCanvasElement.prototype.toDataURL;
HTMLCanvasElement.prototype.toDataURL = function(type) {
    if (type === 'image/png' && this.width === 16 && this.height === 16) {
        const canvas = document.createElement('canvas');
        canvas.width = this.width;
        canvas.height = this.height;
        const ctx = canvas.getContext('2d');
        ctx.drawImage(this, 0, 0);
        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        const data = imageData.data;
        for (let i = 0; i < data.length; i += 4) {
            data[i] = data[i] + Math.floor(Math.random() * 10) - 5;
            data[i+1] = data[i+1] + Math.floor(Math.random() * 10) - 5;
            data[i+2] = data[i+2] + Math.floor(Math.random() * 10) - 5;
        }
        ctx.putImageData(imageData, 0, 0);
        return origToDataURL.apply(canvas, arguments);
    }
    return origToDataURL.apply(this, arguments);
};

// 修改 AudioContext 指纹
const audioContext = window.AudioContext || window.webkitAudioContext;
if (audioContext) {
    const origGetChannelData = AudioBuffer.prototype.getChannelData;
    AudioBuffer.prototype.getChannelData = function() {
        const channelData = origGetChannelData.apply(this, arguments);
        if (channelData.length > 20) {
            const noise = 0.0001;
            for (let i = 0; i < Math.min(channelData.length, 500); i++) {
                channelData[i] = channelData[i] + (Math.random() * noise * 2 - noise);
            }
        }
        return channelData;
    };
}

// 随机化硬件并发数
Object.defineProperty(navigator, 'hardwareConcurrency', {
    get: () => 8 + Math.floor(Math.random() * 4),
    configurable: true
});

// 随机化设备内存
Object.defineProperty(navigator, 'deviceMemory', {
    get: () => 8,
    configurable: true
});

// 覆盖 plugins 长度（正常浏览器有 plugin）
Object.defineProperty(navigator, 'plugins', {
    get: () => [1, 2, 3, 4, 5],
    configurable: true
});

// 覆盖 languages
Object.defineProperty(navigator, 'languages', {
    get: () => ['zh-CN', 'zh', 'en-US', 'en'],
    configurable: true
});
"""
```

### scraper.py — 知乎内容爬虫核心模块

功能：  
1. 使用 Playwright 持久化上下文登录知乎（手动登录，保存 Cookie）  
2. 爬取指定用户的所有回答和文章链接  
3. 爬取指定用户的所有想法（Pins）  
4. 爬取指定问题下的所有（或前 N 个）回答  
5. 爬取单个回答，可选附带评论区  
6. 逐个访问并提取内容，转为 Markdown 保存  
7. 内置反检测（stealth JS 注入、指纹伪装）  
8. 请求间隔随机延迟，降低被封风险

#### 浏览器上下文管理

```python
# ── 浏览器上下文管理 ─────────────────────────────────────────

async def create_browser_context(pw, headless=False) -> BrowserContext:
    """创建带有反检测的持久化浏览器上下文。"""
    USER_DATA_DIR.mkdir(parents=True, exist_ok=True)

    width = 1920 + random.randint(-100, 100)
    height = 1080 + random.randint(-50, 50)

    launch_args = [
        "--no-first-run",
        "--no-default-browser-check",
        "--disable-blink-features=AutomationControlled",
        "--disable-features=IsolateOrigins,site-per-process",
        "--disable-site-isolation-trials",
        "--disable-infobars",
        f"--window-size={width},{height}",
    ]

    context = await pw.chromium.launch_persistent_context(
        user_data_dir=str(USER_DATA_DIR),
        headless=headless,
        slow_mo=50,
        args=launch_args,
        viewport={"width": width, "height": height},
        user_agent=USER_AGENT,
        locale="zh-CN",
        timezone_id="Asia/Shanghai",
        java_script_enabled=True,
    )

    # 注入反检测脚本
    await context.add_init_script(STEALTH_JS)

    return context

# ── 登录 ─────────────────────────────────────────────────────

async def login(timeout: int = 300):
    """
    打开知乎登录页面，等待用户手动登录。
    登录状态会保存在 browser_data 目录中，后续爬取无需重复登录。

    Args:
        timeout: 等待登录的超时时间（秒），默认 300 秒
    """
    print("=" * 60)
    print("🔐 知乎登录")
    print("=" * 60)
    print(f"将打开浏览器，请在 {timeout} 秒内完成登录。")
    print("登录成功后，程序会自动检测并保存登录状态。\n")

    async with async_playwright() as pw:
        context = await create_browser_context(pw, headless=False)
        try:
            page = context.pages[0] if context.pages else await context.new_page()
            await page.goto("https://www.zhihu.com/signin", wait_until="domcontentloaded")

            print("⏳ 等待登录... 请在浏览器中完成登录操作。")

            # 等待用户登录成功（检测跳转到首页或出现用户头像）
            start_time = time.time()
            while time.time() - start_time < timeout:
                url = page.url
                # 登录成功后一般会跳转到首页
                if "signin" not in url and "signup" not in url:
                    # 额外等待几秒确保 Cookie 完全写入
                    await asyncio.sleep(3)
                    print("✅ 登录成功！登录状态已保存。")
                    print(f"   数据目录: {USER_DATA_DIR.resolve()}")
                    return True
                await asyncio.sleep(2)

            print("❌ 登录超时，请重试。")
            return False

        finally:
            await context.close()
```

#### 爬取回答对应的URL

```python
async def collect_question_answer_links(
    page: Page, question_id: str, max_answers: int | None = None
) -> list[str]:
    """
    在问题页面中滚动，收集回答链接。

    Args:
        page: Playwright 页面对象
        question_id: 知乎问题 ID
        max_answers: 最多收集的回答数量（None 表示全部）

    Returns:
        去重后的回答链接列表
    """
    url = f"https://www.zhihu.com/question/{question_id}"
    print(f"🌍 访问: {url}")
    await page.goto(url, wait_until="domcontentloaded")
    await asyncio.sleep(5)

    # 关闭可能的登录弹窗
    await _dismiss_popup(page)

    collected_links = set()
    no_new_count = 0
    max_no_new = 10
    scroll_count = 0
    prev_scroll_height = 0

    while no_new_count < max_no_new:
        # 使用 CSS 选择器提取回答链接
        link_elements = await page.query_selector_all('a[href*="/answer/"]')
        links = []
        for el in link_elements:
            href = await el.get_attribute("href")
            if href:
                if href.startswith("//"):
                    href = "https:" + href
                elif href.startswith("/"):
                    href = "https://www.zhihu.com" + href
                elif not href.startswith("http"):
                    href = "https://www.zhihu.com/" + href
                if f"/question/{question_id}/answer/" in href:
                    links.append(href.split("?")[0])

        prev_count = len(collected_links)
        collected_links.update(links)
        new_count = len(collected_links) - prev_count

        if new_count == 0:
            no_new_count += 1
        else:
            no_new_count = 0

        scroll_count += 1
        print(f"   📜 第 {scroll_count} 次滚动，已发现 {len(collected_links)} 个回答链接"
              + (f"（新增 {new_count}）" if new_count > 0 else "（无新增）"))

        # 检查是否已达到目标数量
        if max_answers and len(collected_links) >= max_answers:
            print(f"   📋 已达到目标数量 {max_answers}。")
            break

        # 检查页面是否包含明确的"到底"标识
        end_marker = await page.evaluate("""() => {
            const bodyText = document.body.innerText;
            return bodyText.includes('已显示全部') || bodyText.includes('没有更多了');
        }""")

        if end_marker and no_new_count >= 3:
            print("   📋 已到达列表底部（页面提示已显示全部）。")
            break

        # 检查页面高度是否还在增长
        current_scroll_height = await page.evaluate("document.body.scrollHeight")
        height_changed = current_scroll_height != prev_scroll_height
        prev_scroll_height = current_scroll_height

        if not height_changed and no_new_count >= 5:
            print("   📋 页面不再加载新内容，停止滚动。")
            break

        # 滚动 — 使用多种方式触发知乎的懒加载
        scroll_distance = random.randint(800, 1500)
        await page.keyboard.press("End")
        await asyncio.sleep(0.5)
        await page.evaluate(f"document.documentElement.scrollTop += {scroll_distance}")
        await asyncio.sleep(0.3)
        await page.keyboard.press("End")

        await asyncio.sleep(2.0 + random.random() * 2)
        if new_count == 0:
            await asyncio.sleep(2.0)

    result = sorted(collected_links)
    if max_answers:
        result = result[:max_answers]
    return result
```

#### 处理评论

```python
async def collect_question_answer_links(
    page: Page, question_id: str, max_answers: int | None = None
) -> list[str]:
    """
    在问题页面中滚动，收集回答链接。

    Args:
        page: Playwright 页面对象
        question_id: 知乎问题 ID
        max_answers: 最多收集的回答数量（None 表示全部）

    Returns:
        去重后的回答链接列表
    """
    url = f"https://www.zhihu.com/question/{question_id}"
    print(f"🌍 访问: {url}")
    await page.goto(url, wait_until="domcontentloaded")
    await asyncio.sleep(5)

    # 关闭可能的登录弹窗
    await _dismiss_popup(page)

    collected_links = set()
    no_new_count = 0
    max_no_new = 10
    scroll_count = 0
    prev_scroll_height = 0

    while no_new_count < max_no_new:
        # 使用 CSS 选择器提取回答链接
        link_elements = await page.query_selector_all('a[href*="/answer/"]')
        links = []
        for el in link_elements:
            href = await el.get_attribute("href")
            if href:
                if href.startswith("//"):
                    href = "https:" + href
                elif href.startswith("/"):
                    href = "https://www.zhihu.com" + href
                elif not href.startswith("http"):
                    href = "https://www.zhihu.com/" + href
                if f"/question/{question_id}/answer/" in href:
                    links.append(href.split("?")[0])

        prev_count = len(collected_links)
        collected_links.update(links)
        new_count = len(collected_links) - prev_count

        if new_count == 0:
            no_new_count += 1
        else:
            no_new_count = 0

        scroll_count += 1
        print(f"   📜 第 {scroll_count} 次滚动，已发现 {len(collected_links)} 个回答链接"
              + (f"（新增 {new_count}）" if new_count > 0 else "（无新增）"))

        # 检查是否已达到目标数量
        if max_answers and len(collected_links) >= max_answers:
            print(f"   📋 已达到目标数量 {max_answers}。")
            break

        # 检查页面是否包含明确的"到底"标识
        end_marker = await page.evaluate("""() => {
            const bodyText = document.body.innerText;
            return bodyText.includes('已显示全部') || bodyText.includes('没有更多了');
        }""")

        if end_marker and no_new_count >= 3:
            print("   📋 已到达列表底部（页面提示已显示全部）。")
            break

        # 检查页面高度是否还在增长
        current_scroll_height = await page.evaluate("document.body.scrollHeight")
        height_changed = current_scroll_height != prev_scroll_height
        prev_scroll_height = current_scroll_height

        if not height_changed and no_new_count >= 5:
            print("   📋 页面不再加载新内容，停止滚动。")
            break

        # 滚动 — 使用多种方式触发知乎的懒加载
        scroll_distance = random.randint(800, 1500)
        await page.keyboard.press("End")
        await asyncio.sleep(0.5)
        await page.evaluate(f"document.documentElement.scrollTop += {scroll_distance}")
        await asyncio.sleep(0.3)
        await page.keyboard.press("End")

        await asyncio.sleep(2.0 + random.random() * 2)
        if new_count == 0:
            await asyncio.sleep(2.0)

    result = sorted(collected_links)
    if max_answers:
        result = result[:max_answers]
    return result
```

#### 断点续传功能实现

```python
def _scan_done_urls_from_disk(output_dir: Path) -> set[str]:
    """
    扫描输出目录中已存在的 Markdown 文件，从文件头部提取来源 URL。
    兼容两种结构：
      - 普通模式（有图片）：<type_dir>/<子文件夹>/index.md
      - --no-images 模式：<type_dir>/<日期_标题>.md（直接在类型目录中）
    """
    done = set()
    url_pattern = re.compile(r'>\s*\*\*来源\*\*:\s*\[([^\]]+)\]')
    for subdir in ("answers", "articles", "pins"):
        type_dir = output_dir / subdir
        if not type_dir.exists():
            continue
        # 兼容两种结构：递归匹配所有 .md 文件
        for md_file in type_dir.rglob("*.md"):
            try:
                text = md_file.read_text(encoding="utf-8")[:500]
                m = url_pattern.search(text)
                if m:
                    done.add(m.group(1))
            except Exception:
                pass
    return done
```

```python
# ── 保存链接列表（用于断点续传） ──
            links_file = output_dir / "links.json"
            links_data = [{"url": url, "type": t} for url, t in all_urls]
            links_file.write_text(
                json.dumps(links_data, ensure_ascii=False, indent=2), encoding="utf-8"
            )
            print(f"📋 链接列表已保存到: {links_file}\n")

            # ── 检查已爬取的内容（断点续传） ──
            progress_file = output_dir / "progress.json"
            done_urls = set()
            if progress_file.exists():
                try:
                    done_data = json.loads(progress_file.read_text(encoding="utf-8"))
                    done_urls = set(done_data.get("done", []))
                except Exception:
                    pass

            # 扫描磁盘上已存在的文件，补充 progress.json 可能遗漏的记录
            disk_urls = _scan_done_urls_from_disk(output_dir)
            if disk_urls - done_urls:
                print(f"📂 从磁盘扫描发现 {len(disk_urls - done_urls)} 个已下载但未记录的内容")
                done_urls |= disk_urls

            if done_urls:
                # 只统计与当前链接列表匹配的数量
                matched = sum(1 for url, _ in all_urls if url in done_urls)
                print(f"📌 检测到之前的进度，已完成 {matched}/{total} 项，将跳过。\n")

                # 同步更新 progress.json
                progress_file.write_text(
                    json.dumps({"done": list(done_urls)}, ensure_ascii=False),
                    encoding="utf-8",
                )
```

### converter.py——HTML → Markdown 转换模块

处理知乎特有的 LaTeX 公式、代码块、图片链接重写、垃圾内容清洗。

核心策略：在 BeautifulSoup 阶段用占位符保护公式，markdownify 转换后再还原为 `$ / $$` 定界符。

参考 yuchenzhu-research/zhihu-scraper 的 converter.py 实现。

#### 清理噪音（知乎直达）

```python
# ── 预处理 HTML ──────────────────────────────────────────

    def _preprocess(self, html: str) -> str:
        """在交给 markdownify 之前做知乎特定的 HTML 清洗。"""
        soup = BeautifulSoup(html, "html.parser")

        # 1) 移除视频 / 卡片 / 按钮等干扰元素
        for selector in (
            "div.VideoCard, .RichText-video, .VideoCard-player",
            ".LinkCard, .RichText-LinkCard, .Card, .Reward",
            ".ContentItem-actions, .RichContent-actions",
            ".Post-SideActions, .BottomActions",
            ".css-1gomreu, .Voters",
        ):
            for tag in soup.select(selector):
                tag.decompose()

        # 2) 去除知乎直答链接（zhida.zhihu.com），只保留链接文本
        for a_tag in soup.find_all("a", href=True):
            if "zhida.zhihu.com" in a_tag["href"]:
                a_tag.replace_with(a_tag.get_text())

        # 3) 处理数学公式
        #    知乎 2024+ 格式: <span class="ztext-math" data-tex="...">
        for span in soup.find_all("span", class_="ztext-math"):
            tex = span.get("data-tex", "")
            if not tex:
                continue
            is_block = tex.startswith(r"\[") and tex.endswith(r"\]")
            if is_block:
                tex = tex[2:-2].strip()
            placeholder = self._store_math(tex, is_block)
            marker = soup.new_tag("var")
            marker.string = placeholder
            span.replace_with(marker)

        #    兼容旧版: <img class="ztext-math" data-formula="...">
        for img in soup.select("img.ztext-math"):
            formula = img.get("data-formula", "")
            if not formula:
                continue
            is_block = (
                img.parent
                and img.parent.name in ("p", "div", "figure")
                and len(img.parent.get_text(strip=True)) == 0
            )
            placeholder = self._store_math(formula, is_block)
            marker = soup.new_tag("var")
            marker.string = placeholder
            img.replace_with(marker)

        # 3) <code> 里的 <br> 换成换行符
        for code in soup.find_all("code"):
            for br in code.find_all("br"):
                br.replace_with("\n")

        # 4) 代码块语言标注
        for pre in soup.find_all("pre"):
            code = pre.find("code")
            if code:
                lang = ""
                for cls in code.get("class", []):
                    if cls.startswith("language-"):
                        lang = cls[len("language-"):]
                        break
                if lang:
                    code["class"] = [f"language-{lang}"]

        return str(soup)
```

```python
# ── 后处理 Markdown ──────────────────────────────────────  
def _postprocess(self, md: str) -> str:  
    """清理噪音 + 还原公式占位符。"""  
    # 去除知乎直答（zhida.zhihu.com）链接，只保留链接文本  
    md = re.sub(r'\[([^\]]*)\]\(https?://zhida\.zhihu\.com[^)]*\)', r'\1', md)  
    # 压缩连续空行  
    md = re.sub(r"\n{3,}", "\n\n", md)  
    # 清理行尾空白  
    md = "\n".join(line.rstrip() for line in md.splitlines())  
  
    # 还原公式 (并做 KaTeX 兼容性处理)  
    for key, formula in self._math_store.items():  
        fixed_formula = self._fix_katex_array(formula)  
        if key.startswith(self._BLOCK_PH):  
            md = md.replace(key, f"\n\n$$\n{fixed_formula}\n$$\n\n")  
        elif key.startswith(self._INLINE_PH):  
            md = md.replace(key, f"${fixed_formula}$")  
  
    # 再次压缩  
    md = re.sub(r"\n{3,}", "\n\n", md)  
    return md.strip() + "\n"
```













