---
title: "Getting Started"
layout: page
date: 2099-06-02 00:00
---

[TOC]

# Simiki #

[![Latest Version](http://img.shields.io/pypi/v/simiki.svg)](https://pypi.python.org/pypi/simiki)
[![The MIT License](http://img.shields.io/badge/license-MIT-yellow.svg)](https://github.com/tankywoo/simiki/blob/master/LICENSE)
[![Build Status](https://travis-ci.org/tankywoo/simiki.svg)](https://travis-ci.org/tankywoo/simiki)
[![Coverage Status](https://img.shields.io/coveralls/tankywoo/simiki.svg)](https://coveralls.io/r/tankywoo/simiki)

Simiki is a simple wiki framework, written in [Python](https://www.python.org/).

* Easy to use. Creating a wiki only needs a few steps
* Use [Markdown](http://daringfireball.net/projects/markdown/). Just open your editor and write
* Store source files by category
* Static HTML output
* A CLI tool to manage the wiki

Simiki is short for `Simple Wiki` :)

## 快速使用指南 ##

### Install ###

	pip install simiki

### Update ###

	pip install -U simiki

### Init Site ###

	mkdir mywiki && cd mywiki
	simiki init

### Create a new wiki ###

	simiki n -t 'test' -c linux

### Generate ###

	simiki g

### Preview ###

	simiki p -w

For more information, `simiki -h` or have a look at [Simiki.org](http://simiki.org)

## 数学公式和插件 ##
simiki 本身并不支持数学公式，你需要依赖一个主题模板，采用前端数学渲染库 MathJax 来进行渲染书写在 markdown 中的 数学公式。注意升级你的 simiki 保持和 markdown 版本兼容，否则可能会报错 `pkg_resources.DistributionNotFound: The 'Markdown==2.6.8' distribution was not found and is required by simiki`. `sudo pip install simiki --upgrade` 对 simiki 进行升级。

配置主题模板的时候，可以在 base.html 文件中加入关于数学公式的配置，特别是 inline mode，可能你写的 Latex 版本的 inline mode 的数学公式 MathJax 并不认识。可以参考 [MathJax 文档](http://docs.mathjax.org/en/latest/options/preprocessors/tex2jax.html)
```js
<script type="text/x-mathjax-config">
        MathJax.Hub.Config({
          tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
        });
        </script>
        <script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.2/MathJax.js?config=TeX-MML-AM_CHTML'></script>
        <!--script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script!-->
        <script src="https://code.jquery.com/jquery-2.2.4.min.js"
            integrity="sha256-BbhdlvQf/xTY9gja0Dq3HiwQF8LaCRTXxZKRutelT44="
            crossorigin="anonymous"></script>

```
注意MathJax和jupyter中数学公式的区别，MathJax没有jupyter中的数学公式兼容性好，比如`$$ db^{[l]} = \frac{\partial \mathcal{L} }{\partial b^{[l]}} = \frac{1}{m} \sum_{i = 1}^{m} dZ^{[l] (i)}\tag{9}$$` 需要注意 `[l]` 和 `(i)` 之间的空格！`\begin{align}` 不能加星号。

## 插入图片 ##
由于 simiki 不支持插入图片，因此只有通过 html 原生语言来插入图片了。我的图片管理都在同一个文件夹下. `/static/images`.


* [simiki.org](http://simiki.org)
* <https://github.com/tankywoo/simiki>
* Email: <me@tankywoo.com>
* [Simiki Users](https://github.com/tankywoo/simiki/wiki/Simiki-Users)

## 使用评论插件 ##
多说和Discus都太老了，现在有一款基于github issue 的评论系统[gitment](https://github.com/imsun/gitment)，给作者点赞.
- [Gitment：使用 GitHub Issues 搭建评论系统](https://imsun.net/posts/gitment-introduction/)

## github page的相关概念 ##
Github 为每一个账户都设置了一个默认的 user pages, 如果你要使用，创建的 repository name 必须符合 <username>.github.io,并且 user pages 只能使用这个项目的主分支master来作为发布源。`User pages must be built from the master branch.`
因此一个账户有且仅有一个 user pages. organization page 和 user pages 类似。那如何在一个账户创建多个 pages呢? 就是另一个 project pages 了。其实每一个项目都可以创建 project pages 来发布。对于project page，你可以选择 master 作为 githubpages 发布源，也可以选择 gh-pages 分支作为发布源，因为pages一般是用来说明项目的，因此项目一般不要使用主分支发布，直接使用 gh-pages 分支更好，这样就和你的 master 代码分开了。如果你就是想要在主分支发布，那么在主分支的root目录下必须要有一个 /docs 目录，发布源代码需要在 /docs 目录下。 自定义域名只要 在发布源的代码目录下面加入CNAME文件即可，里面只需要填写你的自定义域名！如果你的域名还未被DNS收录，那么github会发出警告消息。

我的wiki就是使用 project page，然后 master 是所有的源文件代码目录和markdown笔记文件，但是这个并不是发布源，发布源使用 gh-pages 分支，每次只发布 output 目录下的东西，也就是说我的 gh-pages 分支，只有simiki 生成的output目录下的内容。主分支是整个内容。每次最终要发布的内容推送到 gh-pages 分支，然后备份是推送到 master 分支，这样我的md文件就不会丢失了！在其他电脑上我也可以下载下来编辑和修改，然后重新发布！
## 配置参考 ##
 - [Configuring a publishing source for GitHub Pages](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/)
 - [单个GitHub帐号下添加多个GitHub Pages的相关问题](http://chitanda.me/2015/11/03/multiple-git-pages-in-one-github-account/)
 - [User, Organization, and Project Pages](https://help.github.com/articles/user-organization-and-project-pages/)
 - [Configuring a publishing source for GitHub Pages](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/)
 - [使用simiki搭建个人wiki图片功能](https://www.dadclab.com/archives/6510.jiecao)
