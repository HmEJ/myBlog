install dependencies

```
npm i
```

install hexo-cli

```
npm i hexo-cli
```

clean hexo cache

```
hexo cl
```

generate hexo static files

```
hexo g
```

start server

```
hexo s
```

deploy hexo blog

```
hexo d
```
# front-matter

不详细之处参考：[volantis官方文档](https://volantis.js.org/v6/page-settings/#front-matter)

| 字段         | 含义                     | 值类型    | 默认值                |
| ------------ | ------------------------ | --------- | --------------------- |
| layout       | 布局模版                 | String    | -                     |
| title        | 页面标题                 | String    | -                     |
| seo_title    | 网页标题                 | String    | page.title            |
| short_title  | 页面标题（在group列表中显示） | String    | page.title            |
| date         | 创建时间                 | Date      | 文件创建时间          |
| updated      | 更新日期                 | Date      | 文件修改时间          |
| link         | 外部文章网址             | String    | -                     |
| music        | 内部音乐控件             | [Object]  | -                     |
| robots       | robots                   | String    | -                     |
| keywords     | 页面关键词               | String    | -                     |
| description  | 页面描述、摘要           | String    | -                     |
| cover        | 是否显示封面             | Bool      | true                  |
| top_meta     | 是否显示文章或页面顶部的meta信息 | Bool      | true                  |
| bottom_meta  | 是否显示文章或页面底部的meta信息 | Bool      | true                  |
| sidebar      | 页面侧边栏               | Bool, Array | theme.layout.*.sidebar |
| body         | 页面主体元素             | Array     | theme.layout.on_page.body |
| thumbnail    | 缩略图                   | String    | false                 |
| icons        | 图标                     | Array     | []                    |
| pin          | 是否置顶                 | Bool      | false                 |
| headimg      | 文章头图                 | url       | -                     |
| readmore     | 阅读更多按钮             | Bool      | -                     |

`layout:post` 特有配置：

| 字段	| 含义	| 值类型	| 默认值 |
|-------|------|-----------|--------|
| author	| 文章作者	| [Object]	| config.author |
| categories	| 分类	| String, Array	| - |
| tags	| 标签	| String, Array	| - |
| toc	| 是否生成目录	| Bool	| true |

## 顶置：

```front-matter
---
pin: true
---
```

## 分类：

```front-matter
---
categories:
    - a
    - [b,c]
---
```

## 摘要:

在文章中插入 `<!-- more -->`，前面的部分即为摘要。

```
---
title: xxx
date: 2020-02-21
---

这是摘要

<!-- more -->

这是正文
```

## 作者：

```
---
author: aHang
---
```

## 引入外部文章：

利用 link，搭配自定义的文章作者信息，你可以在文章列表中显示外部文章或者网址，例如：

```
---
layout:     post
date:       2017-07-05
title:      [转]如何搭建基于Hexo的独立博客
categories: [Dev, Hexo]
tags:
  - Hexo
author: xaoxuu
link: https://xaoxuu.com/blog/2017-07-05-hexo-blog/
---

![](https://img.vim-cn.com/d9/a9af7dc49fc0af8ca3e6dd2450a2f7095a87db.png)

```

## 不显示readmore

```
---
readmore: false
---
```

## 不归档

```
---
archive: false
---
```