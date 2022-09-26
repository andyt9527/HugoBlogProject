---
title: "使用Hugo和GitHub Pages搭建个人博客"
date: 2022-09-13T13:29:36+08:00
draft: false
---

# 使用Hugo和GitHub Pages搭建个人博客


# Hugo的安装和使用
## Hugo的安装
初步使用Hugo的话，只需要跟随官网的文档过一遍Quick Start就可以了解基本的安装、使用方法了。
这里根据我自己在Ubuntu 20.04部署经历进行简单的说明。

> sudo apt-get install hugo

等待安装完成之后，可以使用`hugo version`命令来验证。

## 建立新站点
接下来从终端进入到你想要放置博客站点原始内容的目录下使用下面的命令来建立站点。

> hugo new site myblog

该命令会在当前目录下新建一个名为myblog的文件夹。你所有的站点文件都会在这个文件夹下面存放。

## 添加主题
与其他的站点工具不同，Hugo没有默认的主题，需要先添加一个主题才能新建文章。Hugo的官网上有很多的主题可选。选定一个喜欢的主题之后，需要将其下载到myblog文件夹中。在主题说明的页面中点击"download"的按钮，会进入到对应的GitHub页面中。有很多种方式可以将主题文件下载，并放置到`myblog/themes/<YOURTHEME>`文件夹中（`<YOURTHEME>`是主题的名称，可以在该主题的GitHub仓库的页面看到）。
在这里为了使用git对站点进行管理，实现在不同的设备上方便的对站点进行维护，我们使用git的submodule功能。

```
cd myblog 
git init 
git submodule add https://github.com/budparr/gohugo-theme-<YOURTHEME>.git themes/<YOURTHEME>
```
接下来，我们需要在配置文件中指名站点所使用的主题。打开`config.toml`直接编辑或者使用`echo 'theme = "<YOURTHEME>"' >> config.toml`命令。
配置文件中还有其他的可配置内容，这些根据实际主题配置修改即可。

## 新建文章
新建文章可以使用如下的命令，或者直接在`content/<CATEGORY>/<FILE>.<FORMAT>`里面手动创建。

> hugo new posts/my-first-post.md

在这里建议使用Hugo的new命令创建，因为根据主题不同，使用new命令创建的文件会包含简单的模版框架。例如：
```
---
title: my-first-post
date: 2020-06-13T19:45:22+08:00
lastmod: 2020-06-13T19:45:22+08:00
author: Author
cover: /post/xxx-cover.jpg
categories: ["技术"]
tags: ["Hugo", "GitHub"]
draft: true
---
```
具体的配置方式和参数的意义，请查看对应的主题说明。

### 文章中添加图片

Hugo的配置文件和文章中引用图片都是从static作为根目录的。也就是说上面例子中的/post/xxx-cover.jpg实际是在static文件夹中。
```
.
└── static
	└── post
		└── xxx-cover.jpg
```

## 开启Hugo本地服务
我们需要将Hugo本地服务跑起来，才能看到上面操作的成果，看到新的站点的样子。
```
hugo server -D

                   | EN
+------------------+----+
  Pages            | 10
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  3
  Processed images |  0
  Aliases          |  1
  Sitemaps         |  1
  Cleaned          |  0

Total in 11 ms
Watching for changes in /Users/workspace/myblog/{content,data,layouts,static,themes}
Watching for config changes in /Users/workspace/myblog/config.toml
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```
打开`http://localhost:1313/`，我们就可以看到刚才新建的站点了。此时标记为草稿的文章也会展示，但是在实际部署站点的时候需要将文章中的 `draft: true`配置改为`false`。在本地服务开启的时候，对站点的改变（修改配置，修改、新增文章等）会直接展示出来。

## 配置文件的修改
打开配置`config.toml`可以看到很多的参数可以配置，这里只描述最基本的内容，不同的主题可能会支持不同的参数配置，具体请看对应主题的说明文档。

`baseURL`是站点的域名。
`title`是站点的名称。
`theme`是站点的主题。

# 在GitHub Pages部署博客
1. 在 GitHub 创建仓库，仓库名称必须设置为 `<USERNAME>.github.io`，这个仓库仅存放生成的静态内容。
2. 在 GitHub 创建一个项目仓库 `hugo-blog` 并添加为我们本地项目文件夹的远程仓库。这个仓库用来维护站点配置和原始的文章内容。

假设我们在已经通过上文的步骤在 public 文件夹中生成了想发布的静态内容，运行：

`git submodule add -b main https://github.com/<USERNAME>/<USERNAME>.github.io.git public`
在 `public` 目录中创建一个 git 子模块，之后这个目录将以 `https://github.com/<USERNAME>/<USERNAME>.github.io` 作为远程仓库。

> Note: 确保配置文件中的 `baseUrl` 已经设置为了 `<USERNAME>.github.io`。

Hugo 为我们接下来的部署操作提供了一个自动化的 Shell 脚本：
```
#!/bin/sh

# 任一步骤执行失败都会终止整个部署过程
set -e

printf "\033[0;32mDeploying updates to GitHub...\033[0m\n"

# 构建静态内容
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# 切换到 Public 文件夹
cd public

# 添加更改到 git
git add .

# 提交更改
msg="rebuilding site $(date)"
if [ -n "$*" ]; then
msg="$*"
fi
git commit -m "$msg"

# 推送到远程仓库
git push origin main
```

将如上内容保存到`deploy.sh`文件中，并执行`chmod +x deploy.sh`为其添加可执行权限。接着执行部署脚本：`./deploy.sh`
稍等几分钟就可以在 `https://<USERNAME>.github.io` 看到我们的个人博客了。

# 通过 GitHub Actions 自动部署
目前我们的「创作-发布」流程如下：

1. 在项目仓库编辑原始内容并进行版本管理。
2. 执行自动脚本`deploy.sh`生成静态站点并推送到个人主页仓库完成发布。

但是我们可以使用 GitHub Actions，在每次向远程的项目仓库推送原始内容更改时自动执行第 2 步进行发布来提升自动化水平。
GitHub 上有许多这类自动化部署任务的开源 Actions 项目，我选择了这个项目 [GitHub Actions for Hugo](https://github.com/peaceiris/actions-hugo)。具体的操作步骤截图和详细配置项可以查看该项目的README。
下面简单介绍下配置过程：
1. 在项目文件夹中添加目录和文件：`.github/workflows/gh-pages.yml`，
   下面是我个人的`gh-pages.yml`配置:
```
name: GitHub Pages

on:
  push:
    branches:
      - main  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.ref == 'refs/heads/main' }}
        with:
          personal_token: ${{ secrets.HUGOBLOG }}
          external_repository: andyt9527/andyt9527.github.io
          publish_branch: main
          publish_dir: ./public
```
> 这里注意： 
> 1. external_repository 请对应做修改为你自己的Github Pages仓库地址
> 2. personal_token的名字要和博客原仓库Settings-Secrets中添加的token名字一致

2. 添加GitHub Persional access tokens
   > GitHub 账户下 Setting - Developer setting - Personal access tokens 
   > 权限需要开启 repo 与 workflow

3. 在我们博客源仓库 `hugo-blog` 的 Settings - Secrets - Actions 中添加 PERSONAL_TOKEN 环境变量为刚才的 Token， 这里注意token的名字要和`gh-pages.yml`一致

完成上述配置后，推送代码至仓库，即可触发 GitHub Action，自动生成博客页面并推送至 GitHub Pages仓库。

最后我们修改一下`deplay.sh`,这样以后新建博客文章之后只需要再运行`deplay.sh`把原仓库推送到github，然后github actions就会自动更新博客以供访问啦。
```
#!/usr/bin/env bash

# If a command fails then the deploy stops
set -e

# Print out commands before executing them
set -x

# Add changes to source git.
git add .

# Commit changes.
msg="Action in source git $(date)"
if [ -n "$*" ]; then
	msg="$*"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin main
```
