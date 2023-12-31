---
title: Hexo博客搭建
---

# Hexo

使用hexo搭建博客时发现乱七八糟的指令有点多，所以来记录一下下。 

<!--more-->

## 相关软件安装 
添加nodejs源 
```shell
sudo curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
```
安装node.js 
```shell
sudo apt install nodejs
```
安装hexo(-g为全局安装)
```shell
sudo npm install -g hexo
```

## 本地创建博客 
初始化创建 
```shell
hexo init
npm install
```
清除所有记录
```shell
hexo clean
```
生成静态网页
```shell
hexo generate
```
启动服务(-p表示端口 默认4000)
```shell
hexo server -p 80
```

## 主题配置

使用命令安装之后，在博客目录下创建 _config.fluid.yml，将主题的[_config.yml](https://github.com/fluid-dev/hexo-theme-fluid/blob/master/_config.yml)内容复制过去

```shell
npm install --save hexo-theme-fluid
```

后续根据文件注释和[官方文档](https://hexo.fluid-dev.com/docs/)修改配置即可

## 一键部署

在_config.yml补充deploy部分
```yaml
deploy:
  type: git
  repo: #仓库的地址
  branch: master
```

安装插件
```shell
npm install hexo-deployer-git --save
```
上传！
```shell
hexo deploy
```

## 使用github actions自动部署

新建.github/workflows/pages.yml文件

注意这里需要额外安装fluid主题，使用[官网的命令](https://hexo.io/zh-cn/docs/github-pages)会导致主题缺失而部署失败

```yaml
name: Pages

on:
  push:
    branches:
      - master # default branch

jobs:
  pages:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js 20.x
        uses: actions/setup-node@v2
        with:
          node-version: "20"
      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Install fluid
        run: npm install --save hexo-theme-fluid
      - name: Build
        run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

## hexo常用命令整理
```shell
hexo new "postname" #新建文章
hexo new page "pagename"	#新建页面
hexo d #上传 等价于hexo deploy 
hexo g #生成静态网页 等价于hexo generate 
hexo s #本地部署 等价于hexo server 
```

## 撰写博客
摘要和正文使用`<!--more-->`隔开 
标签使用如下形式
```yaml
tags: 
    - hexo
    - fluid
    - blog
```

## 插入图片

打开根目录下_config.yml

将post_asset_folder改为true

此时每次新建文章时将会自动创建同名文件夹用于存放图片

安装插件```npm install hexo-asset-img --save```

> 使用hexo-asset-image插件时转换后的地址错误 无法使用

> 几经周折后发现了hexo-asset-img插件可以正常工作

这个插件实现Typora等Markdown编辑器预览与Hexo发布预览时均能正常显示图片

写博客时只需根据markdown语法```![title](xxx.png)```即可

[参考博客](https://moeci.com/posts/hexo-typora/)

[github地址](https://github.com/yiyungent/hexo-asset-img)

但是`hexo-asset-image`插件和`hexo-abbrlink`插件不兼容，可以使用`hexo-renderer-markdown-it`来渲染

## 生成博客时仓库内包含其他文件(如Readme.md等)

我希望hexo在生成时将readme.md workflows之类的文件加上一起部署到仓库，所以就有了这个需求。

首先在_config.yml文件的```include: ```中加上所需要包含的文件(根目录为source/)，此处支持[通配符](https://github.com/micromatch/micromatch#extended-globbing)

例如：

```yaml
include: 
  - ".github/workflows/*"
```

然后如果不希望文件在生成时被自动转换的话，可以在```skip_render: ```中加上路径，此处支持[通配符](https://github.com/micromatch/micromatch#extended-globbing)

此处添加的文件将会被直接复制到生成后的文件夹(public)中。例如：

```yaml
skip_render: 
  - ".github/workflows/*"
```

然鹅在配完上述内容后，```hexo generate```生成的```public```文件夹中确实有了文件，但是使用```hexo deploy```时我放的```.github```文件夹又消失了。

所以后来又去看了下```hexo-deploy-git```的[配置说明](https://github.com/hexojs/hexo-deployer-git)

如果需要上传隐藏文件的话 需要在```_config.yml```中的```deploy```部分添加```ignore_hidden: false```配置

然后就搞定了

_config.yml文件配置详细内容可以看[官网](https://hexo.io/docs/configuration)

## Hexo插件

### hexo-abbrlink

用于生成文章的简短链接

安装```npm install hexo-abbrlink --save```

根据[仓库说明](https://github.com/rozbo/hexo-abbrlink)操作即可

### hexo-renderer-markdown-it

更快更好的Markdown解析器

安装

```shell
npm un hexo-renderer-marked --save
npm i hexo-renderer-markdown-it --save
```

根据[仓库说明](https://github.com/hexojs/hexo-renderer-markdown-it/)配置即可

### hexo-all-minifier

压缩Hexo生成的文件

安装```npm install hexo-all-minifier --save```

> 这玩意刚开始一直安装不上，后来换了个节点才装上的 大概是网络问题

根据[仓库说明](https://github.com/chenzhutian/hexo-all-minifier)配置即可

## 参考链接
[搭建博客](https://kaiter-plus.gitee.io/2020/03/07/How_To_Freely_Build_Blog/) 

[fluid主题](https://github.com/fluid-dev/hexo-theme-fluid) 

[主题配置](https://blog.csdn.net/weixin_49270402/article/details/117672195) 