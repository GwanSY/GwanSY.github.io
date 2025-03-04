---
layout: post
title: "GitHub Pages 学习1——搭建及配置"
date: 2024-10-16 11:29:08 +0800
categories: GitHubPage
---

# GitHub Pages 配置

GitHub Pages 是一个方便开发者和用户发布静态网页的服务，本文将介绍如何配置 GitHub Pages，以及如何使用 Jekyll 或其他静态网站生成工具进行网站发布。

## 1. 什么是 GitHub Pages?

GitHub Pages 是 GitHub 提供的免费静态网页托管服务，允许用户直接通过 GitHub 仓库发布个人、项目或组织网站。它通过托管静态文件（如 HTML、CSS、JavaScript）来展示内容，并支持一些高级功能，如 Jekyll 动态网站生成工具。

## 2. 开始配置

### 2.1 创建 GitHub 仓库

1. 登陆 GitHub 后，点击 **New Repository** 创建一个新的仓库。
2. 仓库命名规则：
   - 个人主页的仓库名称必须是 `your-username.github.io`，例如 `GwanSY.github.io`。
   - 项目页面可以是任意名字，但要与项目相关。

### 2.2 激活 GitHub Pages

1. 进入 GitHub 仓库的设置页面 (**Settings**)。
2. 滑动至 <kbd>Pages</kbd> 部分。
3. 在 <kbd>Source</kbd> 下拉菜单中，选择 `main` 分支，并保存（Save）。
4. 本人使用的模板 [**Chirpy Starter**](https://github.com/cotes2020/chirpy-starter/)：
   点击按钮 'Use this template' > 'Create a new repository'，新仓库名为 `USERNAME.github.io`，其中 `USERNAME` 是你的 GitHub username。

## 3. 使用 Jekyll 定制 GitHub Pages 网站

Jekyll 是 GitHub Pages 支持的静态网站生成工具。它允许你使用模板生成 HTML 内容，极大地提高了页面创建的效率。更多信息可以参考：[**Jekyll中文文档**](https://jekyll.bootcss.com/)、[**Jekyll英文文档**](https://jekyllrb.com/)、[**Jekyll主题列表**](https://jekyllthemes.org/)。

### 3.1 安装 Jekyll

在本地使用 Jekyll 进行开发需要安装 Git+Ruby+RubyGems。

详细安装教程可以参考以下步骤：

1. 下载 [**Ruby**](https://rubyinstaller.org/downloads/)，下载 [**RubyGems**](https://rubygems.org/pages/download)。
2. Ruby 和 RubyGems 安装教程：[**安装教程**](https://blog.csdn.net/qq_32454347/article/details/87968706)。
3. 在命令行中进入仓库目录。
4. 切换镜像源：
```bash
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
gem sources -l
```
5. 安装 jekyll：
```bash
gem install jekyll
```
6. 安装 jekyll-paginate：
```bash
gem install jekyll-paginate
jekyll -v <!-- 验证 -->
```
7. 安装 Bundler：
```bash
gem install bundler
 ```
8. 本地启动服务：
```bash
jekyll server
```
9. 查看网站：
   打开浏览器并访问 `127.0.0.1:4000` 或 `localhost:4000`。

### 3.2 自定义 GitHub Pages
更换主题。GitHub Pages 提供了一些默认的 Jekyll 主题，你可以在 GitHub 仓库的 '**_config.yml**' 文件中设置主题。例如：'theme: jekyll-theme-cayman'，你也可以从第三方网站[**Jekyll主题列表**](https://jekyllthemes.org/)下载主题，并将其添加到项目中。

### 3.3 部署到GitHub Pages

```bash
1. git add "具体文件"
2. git commit -m "first commit"
3. git push origin main
```
检查你远端仓库已经跟你本地是否同步了，然后在浏览器里输入 '**username.github.io**' ，就可以访问你的博客了。

## 4 编写文章
所有的文章都是 _posts 目录下面，文章格式为 markdown 格式。

直接从 _posts/ 目录下创建新的markdown文件 ，修改名字为 2016-10-16-article1.markdown ，注意：文章名的格式前面必须为 2016-10-16- ，日期可以修改，但必须为 年-月-日- 格式，后面的 article1 是整个文章的连接 URL，如果文章名为中文，那么文章的连接URL就会变成这样的，中文字符会经过转义：https://baixin.io/2015/08/HEXO%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/， 所以建议文章名最好是英文的或者阿拉伯数字。 双击 2016-10-16-article1.markdown 打开。

```bash
---
title:云端Docker搭建ABY库以及本地CLion远程开发
date: 2023-04-03 16:30:00 +0800
categories:[Cyber Engineering]
tags:[Cyber Engineering]
pin: false
---
```

   - title: 显示的文章名， 如：title: 我的第一篇文章<br>
   - date: 显示的文章发布日期，如：date: 2016-10-16<br>
   - categories: tag标签的分类，如：categories: 随笔<br>

如果你对 markdown 语法不熟悉的话，可以看看[**作业部落的教程**](https://www.zybuluo.com/)

# 参考资料
> [Github Page Configuration](https://country-if.github.io/posts/github-page-configuration/)
>
> [Jekyll搭建个人博客](https://www.jianshu.com/p/245aabdace05)
>
> [个人博客网站搭建](https://zhuanlan.zhihu.com/p/87225594)
>
> [搭建个人博客(Jekyll+Github)](https://blog.csdn.net/m0_46578941/article/details/126489793)


