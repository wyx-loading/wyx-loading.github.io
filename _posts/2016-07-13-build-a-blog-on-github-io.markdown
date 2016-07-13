---
layout:	post
title: 	Build a blog on github.io
date:	2016-07-13 13:45:00 +0800
categories:	['blog', 'jekyll']
---

## 在[GitHub Pages][Github Pages]上建立自己的blog

简单几步即可申请到一个个人页面。（Github给每个用户提供的服务，one allow per user）
我们主要是看，[**Blogging with Jekyll**][Blogging with Jekyll]

## Blogging with Jekyll

>Like GitHub Pages, Jekyll is self-aware, so if you add folders and files following specific naming conventions, when you commit to GitHub, Jekyll will magically build your website.

对于目录命名符合特定要求的（Jekyll）的Github Pages项目，Github会在你提交之后自动构建静态页面。

[Jekyll(en)][Jekyll en]: https://jekyllrb.com/
[Jekyll中文][Jekyll cn]: http://jekyllcn.com/

你可以选择手工的构建Jekyll的目录框架，参考
[Creating and Hosting a Personal Site on GitHub][github pages guides]

这里我选择看Jekyll的官方文档，然后构建一个简单的"博客"。

### 本地搭建一个简单的Jekyll环境

Jekyll基于Ruby，几个简单的步骤可以让你快速搭建本地环境。(windows)

- 安装Ruby [downloads][ruby downloads]
- 安装Dev Kit [downloads][ruby downloads]
先cd到Dev Kit解压目录，执行
`ruby dk.rb init`
成功后会生成config.yml文件，编辑该文件，在---块内添加
`- D:\Ruby23-x64(你的ruby安装目录)`
再执行
`ruby dk.rb install`
（若报无ruby这个命令，可以在你安装的时候勾选添加ruby到PATH，或者自己手动添加）
- 安装Jekyll
`gem install jekyll`

完成安装后，你可以使用`jekyll new my-site`创建一个项目，my-site可使用你喜欢的名字，并开始你的blogging之路。

### 学会使用Jekyll
看Jekyll的文档：

- [目录结构][jekyll structure]，了解Jekyll的目录结构，命名方式等。
- [配置][configuration]，了解Jekyll的配置，包括一些全局配置(某些和命令行参数起相同作用)，markdown配置等。文档中还结合一些语法，如if来说明配置的作用。
- [撰写博客][posts]，学习写第一篇博客
- [发布到Github Pages][deploy github pages]，部署到Github Pages上

### Jekyll进阶

Jekyll还有很多功能，本文没有介绍到，这个会边改进本博客边记录下来。
比如模板、集合、插件、主题等。

[Github Pages]: https://pages.github.com/
[Jekyll en]: https://jekyllrb.com/
[Jekyll cn]: http://jekyllcn.com/
[github pages guides]: http://jmcglone.com/guides/github-pages/
[ruby downloads]: http://rubyinstaller.org/downloads/
[jekyll structure]: http://jekyllcn.com/docs/structure/
[configuration]: http://jekyllcn.com/docs/configuration/
[posts]: http://jekyllcn.com/docs/posts/
[deploy github pages]: http://jekyllcn.com/docs/github-pages/