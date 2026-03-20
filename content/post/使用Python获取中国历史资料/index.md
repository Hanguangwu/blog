---
title: 使用Python获取中国历史资料
description: 本文介绍如何使用Python获取中国历史资料。
date: 2026-03-20T22:34:25-08:00
draft: false
categories:
- 编程
tags:
- Python
- 爬虫
- 历史
---

# 使用Python获取中国历史资料


## 基本流程

资料来源： https://www.ilishi.com.cn

先顺序获取URL地址，然后根据生成的JSON文件逐年获取历史资料，最后存为 Markdown 文件。

## Python代码实现


### 顺序获取 URL 地址`scrape_history_zhou.py`

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
使用Playwright自动化工具抓取历史年表URL
从公元前841年开始，通过"下一篇"链接逐页获取
保存为JSON文件，按时间排序
"""

import json
import os
import re
import sys
from datetime import datetime

try:
    from playwright.sync_api import sync_playwright
except ImportError:
    print("请安装playwright: pip install playwright")
    print("并运行: playwright install chromium")
    sys.exit(1)

# ============== 配置区域 ==============
START_URL = 'https://www.ilishi.com.cn/2024/101776.html'  # 起始页面（公元前841年）
OUTPUT_FILE = 'zhou_ilishi_history_urls.json'

HEADLESS = True  # 无头模式
TIMEOUT = 30000  # 页面加载超时（毫秒）


# ======================================


def extract_year_from_title(title):
    """从页面标题提取年份"""
    # 匹配 "公元前X年" 或 "公元X年"
    match = re.search(r'(公元前\d+年|公元\d+年)', title)
    if match:
        year_str = match.group(1)
        # 转换为数字
        if '公元前' in year_str:
            year = -int(re.search(r'\d+', year_str).group())
        else:
            year = int(re.search(r'\d+', year_str).group())
        return year, year_str
    return None, None


def scrape_history_urls(start_url, output_file):
    """抓取历史年表URL"""
    results = []
    visited = set()  # 避免重复访问
    page_num = 0
    max_pages = 500  # 最大页数限制，防止死循环

    print(f"开始抓取，起始URL: {start_url}")
    print("=" * 50)

    with sync_playwright() as p:
        # 启动浏览器
        browser = p.chromium.launch(headless=HEADLESS)
        context = browser.new_context()
        page = context.new_page()

        current_url = start_url

        while current_url and page_num < max_pages:
            page_num += 1

            # 检查是否已访问
            if current_url in visited:
                print(f"[{page_num}] 跳过已访问: {current_url}")
                break

            visited.add(current_url)

            try:
                # 访问页面
                print(f"[{page_num}] 访问: {current_url}")
                page.goto(current_url, timeout=TIMEOUT)

                # 获取页面标题
                title = page.title()
                print(f"  标题: {title}")

                # 提取年份
                year, year_str = extract_year_from_title(title)

                if year is None:
                    print(f"  [警告] 无法从标题提取年份: {title}")
                    # 尝试从页面内容提取
                    content = page.content()
                    year_match = re.search(r'(公元前\d+年|公元\d+年)', content)
                    if year_match:
                        year_str = year_match.group(1)
                        if '公元前' in year_str:
                            year = -int(re.search(r'\d+', year_str).group())
                        else:
                            year = int(re.search(r'\d+', year_str).group())
                            print(f"  从内容提取年份: {year_str}")
                    else:
                        print(f"  [停止] 无法提取年份，终止抓取")
                        break

                # 保存结果
                results.append({
                    'year': year,
                    'year_str': year_str,
                    'title': title.replace(' - 爱历史网', ''),
                    'url': current_url
                })

                print(f"  年份: {year_str}, URL: {current_url}")

                # 查找"下一篇"链接
                next_link = None

                # 方法1: 通过CSS选择器查找
                try:
                    # 查找包含"下一篇"的元素
                    next_element = page.locator('text=下一篇').first
                    if next_element.count() > 0:
                        # 获取父级链接
                        next_link = next_element.locator('xpath=../a').get_attribute('href')
                except:
                    pass

                # 方法2: 通过HTML结构查找
                if not next_link:
                    try:
                        # 查找 class="fr chao f16" 的 p 标签中的链接
                        next_link = page.evaluate('''
                                                  () => {
                                                      const elements = document.querySelectorAll('.fr.chao.f16');
                                                      for (const el of elements) {
                                                          const link = el.querySelector('a');
                                                          if (link && link.href) {
                                                              return link.href;
                                                          }
                                                      }
                                                      return null;
                                                  }
                                                  ''')
                    except:
                        pass

                # 方法3: 通过正则匹配HTML
                if not next_link:
                    html = page.content()
                    match = re.search(r'下一篇：<a href="(/2024/\d+\.html)">([^<]+)</a>', html)
                    if match:
                        next_link = 'https://www.ilishi.com.cn' + match.group(1)
                        print(f"  通过正则找到下一篇: {next_link}")

                if next_link:
                    # 确保是完整URL
                    if next_link.startswith('/'):
                        next_link = 'https://www.ilishi.com.cn' + next_link

                    # 检查是否是有效的下一页
                    # if '10260' in next_link or '/2024/1026' in next_link:
                    if 'fl' in next_link or 'jn' in next_link:
                        print(f"  [停止] 下一篇链接异常: {next_link}")
                        break
                    else:
                        current_url = next_link
                        print(f"  -> 下一篇: {current_url}\n")
                else:
                    print(f"  [停止] 未找到下一页链接")
                    break

            except Exception as e:
                print(f"  [错误] 处理页面失败: {e}")
                break

        browser.close()

    # 按年份排序
    print("\n" + "=" * 50)
    print("按年份排序...")
    sorted_results = sorted(results, key=lambda x: x['year'])

    # 保存为JSON
    print(f"保存到: {output_file}")
    with open(output_file, 'w', encoding='utf-8') as f:
        json.dump(sorted_results, f, ensure_ascii=False, indent=2)

    print(f"\n抓取完成! 共获取 {len(sorted_results)} 条记录")

    # 显示部分结果
    print("\n前5条:")
    for item in sorted_results[:5]:
        print(f"  {item['year_str']}: {item['url']}")

    print("\n后5条:")
    for item in sorted_results[-5:]:
        print(f"  {item['year_str']}: {item['url']}")

    return sorted_results


def main():
    """主函数"""
    print("=" * 50)
    print("历史年表URL抓取工具")
    print("=" * 50)
    print(f"起始URL: {START_URL}")
    print(f"输出文件: {OUTPUT_FILE}")
    print(f"无头模式: {HEADLESS}")
    print("=" * 50 + "\n")

    # 抓取
    results = scrape_history_urls(START_URL, OUTPUT_FILE)

    print("\n完成!")


if __name__ == '__main__':
    main()

```

### 根据URL获取当年内容`scrape_history.py`

```python
import json
import requests
from bs4 import BeautifulSoup
import time
import re


def get_year_content(url, year):
    try:
        headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
        }
        response = requests.get(url, headers=headers, timeout=10)
        response.encoding = 'utf-8'

        if response.status_code != 200:
            return None

        soup = BeautifulSoup(response.text, 'html.parser')
        content_div = soup.find('div', {'id': 'contentText'})

        if not content_div:
            return None

        paragraphs = content_div.find_all('p')
        content_paragraphs = []

        for p in paragraphs:
            if p.find('img'):
                continue

            text = p.get_text(strip=True)

            if not text:
                continue

            if re.match(r'^公元\d+年，', text) or re.match(r'^西元\d+年，', text):
                continue

            content_paragraphs.append(text)

        if not content_paragraphs:
            return None

        content_text = '\n'.join(content_paragraphs)

        return {
            'year': f'西元{year}',
            'content': content_text
        }

    except requests.RequestException:
        return None
    except Exception:
        return None


def scrape_history(urls, years):
    results = []
    failed_urls = []
    for i in range(len(urls)):

        temp_url = urls[i]
        year = years[i]

        print(f'Processing {temp_url} (西元{year})...')

        data = get_year_content(temp_url, year)

        if data:
            results.append(data)
            print(f'  Success: {data["year"]}')
        else:
            failed_urls.append(url)
            print(f'  Failed or empty')

        time.sleep(0.5)

    return results, failed_urls


def save_to_markdown(results, filename):
    with open(filename, 'w', encoding='utf-8') as f:
        f.write('# 历史年表\n\n')
        f.write('| 年份 | 内容 |\n')
        f.write('|------|------|\n')

        for item in results:
            escaped_content = item['content'].replace('|', '\\|').replace('\n', '<br/>')
            f.write(f'| {item["year"]}年 | {escaped_content} |\n')

    print(f'\nData saved to {filename}')


def process_urls_from_json(input_json):
    """从JSON读取URL列表，批量获取内容"""
    # 读取URL列表
    print(f"读取URL列表: {input_json}")
    with open(input_json, 'r', encoding='utf-8') as f:
        url_data = json.load(f)

    print(f"共有 {len(url_data)} 个URL需要处理")
    urls = []
    years = []
    for item in url_data:
        urls.append(item['url'])
        years.append(item['year'])
    return urls, years

if __name__ == '__main__':

    input_json = 'zhou_ilishi_history_urls.json'

    filename = 'history_data_zhou.md'

    urls, years = process_urls_from_json(input_json)

    results, failed_urls = scrape_history(urls, years)

    print(f'\nSuccessfully scraped {len(results)} years')
    print(f'Failed to scrape {len(failed_urls)} URLs')

    if failed_urls:
        print('\nFailed URLs:')
        for url in failed_urls[:10]:
            print(f'  {url}')
        if len(failed_urls) > 10:
            print(f'  ... and {len(failed_urls) - 10} more')

    save_to_markdown(results, filename)

```
## 结果展示

URL地址：

```json
[  
  {  
    "year": -841,  
    "year_str": "公元前841年",  
    "title": "公元前841年历史纪年",  
    "url": "https://www.ilishi.com.cn/2024/101776.html"  
  },  
  {  
    "year": -831,  
    "year_str": "公元前831年",  
    "title": "公元前831年历史纪年",  
    "url": "https://www.ilishi.com.cn/2024/101777.html"  
  },  
  {  
    "year": -830,  
    "year_str": "公元前830年",  
    "title": "公元前830年历史纪年",  
    "url": "https://www.ilishi.com.cn/2024/101778.html"  
  }
]
```

历史资料内容：

```
# 历史年表

| 年份 | 内容 |
|------|------|
| 西元-841年 | 公元前841年，岁次庚申，猴年。是中国有确切纪年的开始。<br/>1、公元前841年简介<br/>公元前841年，是中国历史的端口年份， 此时在位的周厉王刚愎自用，行事暴虐封杀言论，对敢言其不是的国人实行高压手段，《国语·周语上》有载：“国人莫敢言，道路以目。”最终酿成“国人暴动”，导致厉王逃出王宫“出奔于彘”，由召公与周公二相行政，史称“共和行政”。<br/>2、公元前841年“共和时政”时代<br/>公元前841年历史年表 公元前841年历史大事 公元前841年大事记国人暴动<br/>历史上大名鼎鼎的“共和元年”。“共和”这一新概念，当时是指不要国王了，有多位贤臣共同来执政，求得一种和谐与和平。这个“共和”制度肯定比周厉王一人独裁好，是没有疑问的。而且从这一年开始，中国每一位帝王的在位、驾崩时间，全部有了明确记载至到宣统退位(共计2752年)，这也是世界史上没有的。至于召公会不会实行他那十二条了解下情的办法，没有史书记载，不好瞎说。又过了十四年(公元前828年)，厉王死于彘。这时太子(姬静) 也在召公家里读书学习长大成人了。周、召两公就和大臣商量撤下了“共和”这个招牌，又复回了禹王的老传统，拥立太子为周宣王。宣王继位一开始十多年还算好。二十年后，他老爸周厉王的毛病又在他身上复发了。不再听大臣的忠谏，结果打了一次败仗，七年后去世。又换上了周厉王之孙(周幽王) 继了位，这是西周最后的一位亡国之君。公元前的西周、东周姬家王朝长达790年，从公元前大约1046年(见《夏商周断代工程》)的周武王姬发，到公元前256年的周赧王姬延)，它是一个在世界上独一无二的，比较牢固而又漫长可怕的奴隶、封建、以君为本的社会。<br/>公元前841年，如果仅仅以年份而论，这是中国历史上最重要的一年，因为从这一年开始，中国历史才有了确切的年份记载。从此之后，中国历史就进入了完全信史时代。而在这一年之前，在前面写了好几千年，明确写出年份的也有一千多年，但那些年代，都是大致年份，并不是有文字确切记录的年份。只有到了公元前841年，我们才敢于充满信心地将哪一年的事归于哪一年。感谢伟大的司马迁先生，他以他的付出，为我们这些后世历史工作者及爱好者，梳理出了一条准确的历史通道。伟人千古。<br/>当然，公元前841年之所以了不起，还不仅仅因为这一年是准确记年之始，这一年所发生的“国人暴动”，让中国进入了一个“共和”时代。国人暴动的起因很简单，那就是典型的官逼民反。上一节我们谈到过，周厉王以其残酷的统治，辅以特务手段，成功地使国内的老百姓再也没人敢于抨击他的暴政，不仅不敢抨击，评论也不敢了，又不止不敢评论，说话都不敢了。让人连话都不敢说的政府，不管他有多么强横的手段，也注定不长久。<br/>说他在编纂史书时查阅了大量的文献资料，关于纪年的书籍很多，但是他们在共和元年（公元前841年）之后的记载是一致的，也就是说，公元前841年以后的确切纪年是可以确定的。<br/>3、公元前841年历史记载<br/>那么前841年以前的纪年有没有记载呢？有，而且司马迁说，自黄帝时代以来都有确切的纪年。可是不同的资料说法也有所不同，真实的历史纪年没法确定，于是本着实事求是的原则，司马迁没有把它们记录下来。随着历史的演进，那些有着公元前841年以前的纪年的珍贵史料渐渐湮没失传了，流传下来最早的年表就只有史记的十二诸侯年表了，因此我们取公元前841年作为中国有确切纪年的历史的开始。<br/>《史记·十二诸侯年表》始于前841年。是年为：宋釐公十八年，周厉王三十七年、共和元年，齐武公十年，陈幽公十四年，蔡武侯二十三年，曹夷伯二十四年，燕惠侯二十四年，鲁真公十五年，楚熊勇七年，卫釐侯十四年，晋靖侯十八年，秦嬴仲四年。<br/>共和行政指西周时期国人暴动后，周厉王出奔，由召穆公、周定公共同主持政事的政权，一说为共伯和执政。从公元前841年到公元前828年，共持续十四年。共和元年，即公元前841年，是我国现存史料中有确切纪年的开始。共和十四年，即公元前828年，周厉王死于彘，太子静即位，是为周宣王，共和结束。<br/>公元前841年·中国周厉王暴虐，国人暴动，厉王逃亡。召公、周公行政，号曰共和。中国历史准确年代自此开始（共和当年改元）共和 前841年——前828年 在位14年。公元前841年1月12日，周厉王三十七年、鲁真公十五年夏正头年十二月朔壬子。周历共和元年正月初一日。由《西周金文历谱》按《膳夫山鼎》推定为：“厉王三十七年正月壬子朔。” |
| 西元-831年 | 在公元前831年，撒缦以色遭到乌拉尔图人的反抗。公元前831年，周共和十一年 鲁真公二十五年 陈僖公元年 宋国君僖公卒，子惠公覸（jian）即位。陈幽公十二年，幽公二十三年卒就可以推定为公元前<br/>公元前831年 - 周宣王<br/>周宣王七年（公元前831年），随申伯平定申、甫、许三个小国，此时仲氏也随父亲孙子仲到甫国，并留下《汝坟》和《汉广》等诗。宣王八年到十年，尹吉甫奉命东征鲁国，担任监建营房一职。即至三年后回家时，父母却为他娶了一名姜姓女子，致使仲氏被休回娘家。<br/>在公元前831年，撒缦以色遭到乌拉尔图人的反抗。子举，子鲍祀之子，公元前858~前831年，宋厘公，葬于河南永城市芒山镇僖山 春秋时期: 子 见:子举之子，公元前830~前800年，宋惠公，葬于河南商丘旧城区地下待考 子□□:子见之子，公元前800~前800年，宋哀公，葬于河南商丘旧城区地下待考 子□□:宋哀公之子。<br/>公元前831年 - 厉王奔彘<br/>幽公十二年，周厉王奔于彘。二十三年，幽公卒，子釐公孝立。釐公六年，周宣王即位。(《陈杞世家》36.1575、1576)按照这个记载，宣壬“即位”的年代应在厉王奔彘以后17年。如上面己经指定，厉王奔彘年为公元前842年。此年等于陈幽公十二年，幽公二十三年卒就可以推定为公元前831年。子釐公按照古时通例以次年即公元前830年为元年，在位6年“周宣王即位”就是公元前825年。<br/>除了《陈杞世家》以外，其他的古书也反映宣王时代曾使用过两个相隔二年的年历。《史记·秦本纪》里定秦仲的卒年为宣王六年:秦仲立三年，周厉王无道，诸侯或叛之。西戎反王室，灭犬丘大骆之族。周宣王即位，乃以秦仲为大夫，诛西戎。西戎杀秦仲。秦仲立二十三年，死于戎。 |

```




