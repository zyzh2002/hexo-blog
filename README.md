# 博客说明文档

![build_and_push](https://github.com/zyzh2002/hexo-blog/actions/workflows/build_and_push.yml/badge.svg)

这是个人博客<https://blog.zyzh20021020.cn>的说明文档。

## 构建环境与所用框架

项目使用`Node`版本为`v22.13.1`。无特殊需求请默认使用高于该版本的`Node`，并参考框架所需要求。

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

已经使用Github Actions对博客构建进行了自动化部署。`public`文件夹下的内容会自动上传至腾讯云COS。
