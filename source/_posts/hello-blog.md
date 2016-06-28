---
title: Hello, blog!
date: 2016-06-25 14:24
tags: [hexo, 多说]
categories: shit
---
域名买回来快一年了，自从上次捣鼓blog到现在半年有余，早上一时心血来潮，一口气把这东西整好了。感谢开源主题的同学，我很喜欢Material风格，等有空了自己也贡献一些主题。

大致记录一下建站过程，安装过程、生成blog之类的就不赘述，官网文档灰常清楚，说一些遇到的问题。

## Wordpress or Hexo？

最早试用了wordpress，发现不好玩，配置太麻烦，偶然间发现hexo，扫了下文档：nodejs驱动，安装、配置、部署简单，然后去尼玛的wordpress。

## 版本管理
官方推荐的安转主题的方法：
```bash
git clone [主题git仓库] themes/xxx
```
然后问题就来了，用git做版本管理，发现主题目录无法`stage`，当然也就没办法提交了。

### 原因和解决方法
`git clone`下来的目录中有**.git**目录，git认为**xxx**目录为子模块`submodule`，子模块需要使用`git submodule `命令操作。对于git不是很熟悉的同学，比如在下，这简直噩梦。时间紧迫，直接跑一下以下命令，然后就可以继续愉快地使用`git add/commit`了。

```bash
rm -rf themes/xxx/.git
```

## 使用评论插件

我使用的主题集成[多说](http://duoshuo.com/)，按照主题文档指导，要启用的话需要一个***shortname*。妈蛋翻遍多说官网，没找到哪里有**shortname**。Google了一下，看到一些问答，说是先建站，然后有个二级域名，二级域名第一个单词啥的。但死活没人给出建站地址，尼玛官网上居然也找不到。不知道翻了多少问答，终于在一个问答的回复的链接中找到了建站链接，真尼玛坑啊。对，就下面这东西，就是它！！！

<p style="font-size: 36px;text-align:center">多说建站链接：http://duoshuo.com/create-site/</p>
