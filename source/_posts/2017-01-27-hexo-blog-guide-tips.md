---
title: "Hexo blog: guide & tips"
date: 2017-01-27 16:10:04
categories:
- Software
tags:
- website
- Hexo
toc: true
---
用Hexo搭建了本博客，加入多说评论、Mathjax数学公式、目录支持，修正了语言显示，添加了图标。在此记录参考的链接、遇到的问题和解决方案。

UPDATE 2021/03/26 本文有很多过时的信息。

<!-- more -->

# 原理
Hexo结合模板，将用Markdown书写的文本转换成html，将html提交到Github。

# 命令
## new
```bash
hexo n [layout] <title>
```

## generate
```bash
hexo g
```

## publish
```bash
hexo p [layout] <filename>
```

## server
```bash
hexo s
```

网址：`http://localhost:4000/`

选项`-s`：只使用静态文件

## deploy
```bash
hexo d
```
选项`-g`：部署之前先生成静态文件

## clean
```bash
hexo clean
```

## list
```bash
hexo l <type>
```

# 安装和初始化
[文档 | Hexo](https://hexo.io/zh-cn/docs/)

`language`设置为`default`或`themes/landscape/languages`中的那些，否则会显示法语。

# 评论
[创建站点 - 多说，社会化评论框](http://duoshuo.com/create-site)
[Hexo使用多说教程](http://dev.duoshuo.com/threads/541d3b2b40b5abcd2e4df0e9)

- 文中提到的`_config.yml`是主目录下的，不是`themes`下的。
- 修改`_config.yml`在重新`hexo s`后才生效。

# 图标
在主目录或对应主题的`source`目录下加入文件，设置主题`_config.yml`的`favicon`。

# Mathjax
[搭建一个支持LaTEX的hexo博客 - Platonic Time](http://blog.csdn.net/emptyset110/article/details/50123231)

$$E=mc^2$$

# 摘要
在文本中使用`<!--more-->`标记摘要结束的地方。

# 目录
[图灵社区：阅读：Hexo站点中添加文章目录以及归档](http://www.ituring.com.cn/article/199624)

这篇文章有两个地方需要修正：
- 用`post`替换`item`。
- `item.toc !== false`改为`!index && post.toc`。进入这个`else`子句时`index`可能为`true`，而我们通常不希望在首页生成目录。

# 页面
使用`hexo n page <filename>`命令，并在主题设置文件的`menu`中添加该页，以在header中显示。

# 其他
- 文章标题中有井号等特殊字符，将整个标题用双引号引起来。