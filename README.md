# 博客说明文档

![build_and_push](https://github.com/zyzh2002/hexo-blog/actions/workflows/build_and_push.yml/badge.svg)
[![Netlify Status](https://api.netlify.com/api/v1/badges/dd099dfc-3fb5-41af-931e-ab2d417e8699/deploy-status)](https://app.netlify.com/sites/zyzh0-pesonal-blog/deploys)

这是个人博客<https://blog.zyzh20021020.cn>的说明文档。

## 构建环境与所用框架

项目使用`Node`版本为`v22.x`。无特殊需求请默认使用高于该版本的`Node`，并参考框架所需要求。

博客基于[hexo](https://hexo.io/)与其[icarus](https://github.com/ppoffice/hexo-theme-icarus)主题构建。修改相关配置文件之前建议完整阅读前文所提项目文档。

## 目录结构

项目目录结构类似于典型的hexo项目，但是资源的组织形式上有改动。

```text
.
├── _config.yml
├── _config.icarus.yml
├── _config.landscape.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   ├── _posts
|   └── images
|       └── covers
└── themes
```

`source/images`里存放了自定义资源文件；`source/images/covers`里存放文章的封面图片。

## 构建方式

确保你安装了`node`并配置好了`npm`，如果没有的话请参考[官方教程](https://nodejs.org/zh-cn/download)安装`fnm`和`npm`。如果你在中国大陆还要确保对`npm`进行换源。

```sh
npm config set registry https://registry.npmmirror.com #淘宝源
```

之后恢复项目环境

```sh
npm install
```

在此之后就可以运行`npx hexo-cli`相关命令了。

已经使用Github Actions对博客构建进行了自动化部署。将修改好的项目push到远程仓库，`public`文件夹下的内容会自动上传至腾讯云COS。
