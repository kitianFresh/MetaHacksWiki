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

## Quick Start ##

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

## Others ##

* [simiki.org](http://simiki.org)
* <https://github.com/tankywoo/simiki>
* Email: <me@tankywoo.com>
* [Simiki Users](https://github.com/tankywoo/simiki/wiki/Simiki-Users)

## 配置参考 ##
 - [Configuring a publishing source for GitHub Pages](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/)
 - [单个GitHub帐号下添加多个GitHub Pages的相关问题](http://chitanda.me/2015/11/03/multiple-git-pages-in-one-github-account/)
 - [User, Organization, and Project Pages](https://help.github.com/articles/user-organization-and-project-pages/)
 - [Configuring a publishing source for GitHub Pages](https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/)
 - [使用simiki搭建个人wiki图片功能](https://www.dadclab.com/archives/6510.jiecao)
