# Hugo-Stack-Theme

[Card-style theme designed for bloggers](https://stack.jimmycai.com/)

# FrontMatter Example

```md
---
title: 沉浸式开源浏览器翻译插件FluentRead
description: 本文介绍浏览器翻译插件FluentRead。
date: 2026-10-14T12:34:25+08:00
draft: false
categories:
- APP
tags:
- GitHub
- FluentRead
---
```

# Writing

Stack uses Hugo's **page bundles** to organize your content. A page bundle is a directory that contains a content file and any related resources. For example, a page bundle for a blog post might look like this:

```
content
└── post
    └── my-first-post
        ├── index.md
        ├── image1.png
        └── image2.png
```


This is the recommended way to organize your content. You can read more about page bundles in [Hugo's documentation](https://gohugo.io/content-management/page-bundles/).

Inserting external images is supported, but **it is not recommended**.

Features like image gallery and image zooming will not work with external images. Those feature needs to know the image's dimensions, which is not possible with external images.

With above organization, you can insert images in your content like this:

```
--- content/post/my-first-post/index.md ---
![Image 1](image1.png)
![Image 2](image2.png)
```


## Insert image gallery

To insert an image gallery, you need to create a page bundle for the gallery. For example:

```
content
└── gallery
    └── my-first-gallery
        ├── index.md
        ├── image1.png
        ├── image2.png
        └── image3.png
```


Then, you can insert the gallery in your content like this:

```
--- content/gallery/my-first-gallery/index.md ---
![Image 1](image1.png) ![Image 2](image2.png)
![Image 3](image3.png)
```

Which will render in two rows, with two images in the first row and one image in the second row.


# Frontmatter Configs

- [description](https://stack.jimmycai.com/writing/frontmatter#description)
- [image](https://stack.jimmycai.com/writing/frontmatter#image)
- [comments](https://stack.jimmycai.com/writing/frontmatter#comments)
- [license](https://stack.jimmycai.com/writing/frontmatter#license)
- [math](https://stack.jimmycai.com/writing/frontmatter#math)
- [toc](https://stack.jimmycai.com/writing/frontmatter#toc)
- [style](https://stack.jimmycai.com/writing/frontmatter#style)
- [keywords](https://stack.jimmycai.com/writing/frontmatter#keywords)
- [readingTime](https://stack.jimmycai.com/writing/frontmatter#readingtime)

## description

- Type: `string`
- Available in: single pages and list pages

Description of the page.

## image

- Type: `string`
- Available in: single pages and list pages

Featured image of the page.

## comments

- Type: `bool`
- Available in: single pages

Show / hide comment section of the page.

## license

- Type: `string|bool`
- Available in: single pages
- Default: `.Site.Params.Article.License.Default`

License of the page. If it's set to `false`, the license section will be hidden.

## math

- Type: `bool`
- Available in: single pages

Enable / disable KaTeX rendering.

## toc

- Type: `bool`
- Available in: single pages
- Default: `.Site.Params.Article.toc`

Show / hide table of contents of the page.

TOC will be shown only if the page has at least one heading.

## style

- Type: `map[string]string`
- Available in: list pages

Additional CSS styles for taxonomy term badge that appears in article page.

Currently only `background` (background of the badge) and `color` (text color) are supported.

## keywords

- Type: `[]string`

Keywords of the page. Useful for SEO.

## readingTime

- Type: `bool`
- Default: `.Site.Params.Article.ReadingTime`

Show / hide reading time of the page.
# Shortcodes

Stack comes with a set of [shortcodes](https://gohugo.io/content-management/shortcodes/) that you can use in your content.

This page only includes the shortcodes that are specific to Stack. Hugo's built-in shortcodes are documented [here](https://gohugo.io/content-management/shortcodes/#use-hugos-built-in-shortcodes).

## Bilibili video

Embed a [Bilibili](https://www.bilibili.com/) video.

```
{{< bilibili VIDEO_ID PART_NUMBER >}}
```


The `Video_ID` can be found in the URL of the video. For example, the video ID of `https://www.bilibili.com/video/av12345678` is `av12345678`. Both `AV` and `BV` are supported.

The `PART_NUMBER` is optional. It can be used to specify the part of the video to play. For example, the part number of `https://www.bilibili.com/video/av12345678?p=2` is `2`.

## Tencent video

Embed a [Tencent Video](https://v.qq.com/) video.

```
{{< tencent VIDEO_ID >}}
```


The `Video_ID` can be found in the URL of the video. For example, the video ID of `https://v.qq.com/x/cover/hzgtnf6tbvfekfv/g0014r3khdw.html` is `g0014r3khdw`.

## YouTube video

Embed a [YouTube](https://www.youtube.com/) video.

```
{{< youtube VIDEO_ID >}}
```

The `Video_ID` can be found in the URL of the video. For example, the video ID of `https://www.youtube.com/watch?v=VIDEO_ID` is `VIDEO_ID`.

## Generic video file

Embed a video file.

```
{{< video VIDEO_URL >}}

{{< video src="VIDEO_URL" autoplay="true" poster="./video-poster.png" >}}
```


The `VIDEO_URL` can be a URL or a path relative to the `static` directory. For example, `src="/video/my-video.mp4"` will embed the video file `static/video/my-video.mp4` of your site folder.

The `autoplay` attribute is optional. It can be used to specify whether the video should be played automatically. The `poster` attribute is optional. It can be used to specify the poster image of the video.

## GitLab

Embed a [GitLab](https://gitlab.com/) snippets.

```
{{< gitlab SNIPPET_ID >}}
```


The `SNIPPET_ID` can be found in the URL of the snippet. For example, the snippet ID of `https://gitlab.com/-/snippets/1234567` is `1234567`.

## Quote

```
{{< quote author="A famous person" source="The book they wrote" url="https://en.wikipedia.org/wiki/Book">}}
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
{{< /quote >}}
```















