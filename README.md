# 博客说明文档

这是个人博客<https://blog.zyzh20021020.cn>的说明文档。

## 构建环境与所用框架

项目使用 `Node` 版本为 `24`，以 `.node-version` 为准。

博客基于 [hexo](https://hexo.io/) 与其 [icarus](https://github.com/ppoffice/hexo-theme-icarus) 主题构建。`themes/icarus/` 是本地追踪的主题目录，不再通过根依赖安装主题包。

## 目录结构

项目目录结构类似于典型的 hexo 项目，但是资源的组织形式上有改动。

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
    └── icarus
```

`source/images` 里存放了自定义资源文件；`source/images/covers` 里存放文章的封面图片。

## 构建方式

确保你安装了 `node` 并配置好了 `npm`，如果没有的话请参考 [官方教程](https://nodejs.org/zh-cn/download) 安装 `fnm` 和 `npm`。如果你在中国大陆还要确保对 `npm` 进行换源。

```sh
npm config set registry https://registry.npmmirror.com
```

之后恢复项目环境：

```sh
npm install
cd themes/icarus && npm install
cd ../..
```

构建博客：

```sh
npx hexo clean && npx hexo g
```

在此之后就可以运行 `npx hexo-cli` 相关命令了。

## 自动部署

已经使用 GitHub Actions 对博客构建进行了自动化部署。将修改好的项目 push 到远程仓库后，`public` 文件夹下的内容会自动部署至 Cloudflare Pages。

GitHub 仓库需要配置以下 Secrets：

- `CLOUDFLARE_API_TOKEN`
- `CLOUDFLARE_ACCOUNT_ID`
- `CLOUDFLARE_PROJECT_NAME`
