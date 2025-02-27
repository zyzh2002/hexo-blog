---
title: Hexo博客的搭建与迁移至腾讯云COS
date: 2025-01-30 16:41:04
categories: 瞎折腾记录
cover: images/cover-Hexo博客的搭建与迁移至腾讯云COS.png
toc: true
tags:
  - 博客
  - CI/CD
  - node.js
---

## Intro

{% raw %}
<style type="text/css">
.heimu { color: #000; background-color: #000; }
.heimu:hover { color: #fff; }
</style>
{% endraw %}

{% raw %}<span class="heimu">前情提要：发现华为送的HECS性能和带宽都是弟中弟</span>{% endraw %}，实在受不了WordPress缓慢的加载速度和难用的富文本编辑器，博客迁移至Hexo已经有一段时间了，但是站点的生成和部署均是基于在新加坡的一台云服务器，要写点稿子还得远程连接上去，多少有点麻烦。因此，在这里将Hexo部署转移到Github上，并使用Github Actions做自动化构建，构建完成后直接发布至腾讯云COS。

<!-- more -->

## 构建基本的Hexo项目

本文假设你已经完成了`node`的安装。如果还未安装`node`，请参考[官网](https://nodejs.org/zh-cn/download)使用`fnm`进行安装。尽量使用较新的LTS版本。{% raw %}<span class="heimu">不是哥们你要是这一步都装不明白还是别往下看了</span>{% endraw %}

### 配置Hexo环境

可以使用`npm`安装Hexo。

```sh
npm install -g hexo-cli
```

{% raw %}<article class="message is-warning"><div class="message-body">{% endraw %}
**注意**：如果你处于中国大陆地区，请为`npm`正确配置镜像源，以免无法下载。
{% raw %}</div></article>{% endraw %}

安装完成后便可以使用Hexo的命令行。执行以下命令来在指定文件夹中初始化你的Hexo项目：

```sh
$ hexo init <YOUR_REPO_FOLDER>
...
INFO  Cloning hexo-starter https://github.com/hexojs/hexo-starter.git
INFO  Install dependencies
INFO  Start blogging with Hexo!
```

之后你的项目结构将类似这样：

```text
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

构建完成Hexo项目之后，我们需要卸载全局的`hexo-cli`的`node`包，转而将其安装至现在的项目目录中。这样可以使项目更加独立，并且方便之后在Github Actions上的部署。

```sh
$ npm uninstall -g hexo-cli
$ cd <YOUR_REPO_FOLDER>
$ pwd
<YOUR_REPO_FOLDER> #请确认你的工作目录，注意一定是在项目目录中执行接下来的安装命令
$ npm install hexo-cli
$ npx hexo version
...
INFO  === Checking package dependencies ===
INFO  === Checking theme configurations ===
INFO  === Registering Hexo extensions ===
hexo: 7.3.0
hexo-cli: 4.3.2
...
zlib: 1.3.0.1-motley-82a5fec
```

到这里你的项目就相对独立了，可以按照你个人的喜好对Hexo进行配置。只是注意，在之后执行Hexo的命令行时，请确保在项目目录中使用`npx`来调用。

## 上传项目至Github并配置自动化部署

### git仓库相关设置

请先检查`.gitignore`文件并添加你需要的忽略项。一般情况下项目自动生成的文件已经能满足要求。

```text
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
_multiconfig.yml
```

之后将你项目的git仓库上传至Github并检查内容是否正确。

### 配置Github Actions

如果你不知道Github Actions是什么，请阅读[官方文档](https://docs.github.com/en/actions/about-github-actions/understanding-github-actions#overview)，“GitHub Actions is a continuous integration and continuous delivery (CI/CD) platform that allows you to automate your build, test, and deployment pipeline”。

可以使用Github Acions来实现博客的自动生成和部署。仓库的`.github/workflow`文件夹内保存了描述工作流的`yaml`文件，这里我们新建一个`build_and_push.yml`。

```yaml
# .github/workflow/build_and_push.yml

name: zyzh0's Blog CI/CD

on:
  push:
    branches: [main, master] # 在发生push时触发workflow
    paths:
      - '*.json'
      - '**.yml'
      - '**/source/**'

jobs:
  blog:
    timeout-minutes: 30 # 运行超时时长
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - name: Cache node_modules # 缓存node_modules以加快运行速度
        uses: actions/cache@v4
        env:
          cache-name: cache-node-modules
        with:
          path: ~/.npm
          key: ${{ runner.os }}-build-${{ env.cache-name }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-build-${{ env.cache-name }}-
            ${{ runner.os }}-build-
            ${{ runner.os }}-
      - name: Install Node.js dependencies # 安装依赖
        run: |
          npm install
          echo "init node successful"
      - name: Build Blog # 构建网站
        run: |
          npx hexo clean
          npx hexo g
          echo "build blog successful"
      - name: Deploy Blog to Tencent COS # push到腾讯云的对象存储COS
        uses: TencentCloud/cos-action@v1
        with:
            secret_id: ${{ secrets.TENCENT_CLOUD_SECRET_ID }} # 腾讯云的Secret_ID和Secret_KEY
            secret_key: ${{ secrets.TENCENT_CLOUD_SECRET_KEY }}
            cos_bucket: ${{ secrets.COS_BUCKET }}
            cos_region: ${{ secrets.COS_REGION }}
            local_path: public # 构建输出文件夹
            remote_path: . # 直接推送到COS根目录
            clean: true
```

请按照注释理解该`yaml`文件描述的工作流。该工作流将仓库签出，然后设置`node`环境并安装依赖，最后构建网站到`public`文件夹中，然后使用腾讯云提供的对COS的操作将其push到我个人的COS存储桶中。

### 配置访问密钥

这是获取上一步配置文件中腾讯云`Secret_ID`和`Secret_KEY`的必要条件。首先登陆你的腾讯云账号，在右上角的账户头像处点击“访问管理”：

![进入访问管理](访问管理.png)

选择左侧的“用户列表”，之后新建一个访问COS的专用子账户：

![新建用户](新建用户.png)

之后选择自定义创建，需要勾选“编程访问”，以及赋予“QcloudCOSFullAccess”权限。

![设置编程访问](编程访问.png)

![赋予权限](权限.png)

建立完成用户之后便可以新建相应的`Secret_ID`和`Secret_KEY`，一定要妥善保存。

![新建访问密钥](新建密钥.png)

在获取了`Secret_ID`和`Secret_KEY`之后，我们将其以“Repository secrets”的形式将其添加到Github repo以保证其不是被直接保存在仓库的公开内容中。打开你的仓库，按照“Settings”、“Security”、“Secrets and variables”、“Actions”的顺序打开并按顺序添加名为`TENCENT_CLOUD_SECRET_ID`和`TENCENT_CLOUD_SECRET_KEY`的两个secret，分别为刚才创建的`Secret_ID`和`Secret_KEY`的值。

![创建secret](新建secret.png)

## COS相关配置

### 设置静态网站

在腾讯云的[对象存储控制面板](https://console.cloud.tencent.com/cos/bucket)中添加你的存储桶之后，根据[官方教程](https://cloud.tencent.com/document/product/436/32670?from=copy)将存储桶设置为静态网站，在调试完成之后建议打开“强制HTTPS”选项。如果你有自己的域名，可以[添加自己的域名至存储桶访问](https://cloud.tencent.com/document/product/436/36638?from=copy)。因为在配置存储桶设置是打开了强制HTTPS，所以你还需申请上传你自己的TLS证书。

![打开强制HTTPS](https.png)

## 完整工作流

配置完成后便可以实现Hexo的完全serverless部署。在clone自己的仓库至本地后，只需要在本地调试好内容，之后便可以直接将更改push到Github上，定义好的workflow会自动构建并发布到腾讯云COS中，实现自动化部署。更改并发布文章实际上只需要一台配置有`node`环境的电脑即可。

![Github Actions执行成功](workflow.png)

## 参考的文章

* [轮子再造 | 使用 GitHub Actions 自动部署 Hexo 博客 - 上篇](https://oreo.life/blog/2021-09-01-deploy-hexo-with-github-actions-1/#%E5%B0%86-workflow-%E9%83%A8%E7%BD%B2%E5%88%B0-github-actions)
* [拥抱腾讯云服务：Github Actions+COS，快速搭建你的Wiki文档](https://cloud.tencent.com/developer/article/1545303)

<a class="tag is-dark is-medium" href="https://ppoffice.github.io/hexo-theme-icarus/gallery/preview.png" target="_blank">
<span class="icon"><i class="fas fa-camera"></i></span>&nbsp;&nbsp;
Icarus Preview
</a>
