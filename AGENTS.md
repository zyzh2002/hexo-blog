# AGENTS.md

## 项目概述

Hexo 8.1.1 静态博客，使用 icarus 主题，部署至 Cloudflare Pages。无测试、lint 或 typecheck 命令。

## 关键命令

```bash
npm install          # 安装依赖
npm run clean        # 清理缓存和 public/
npm run build        # hexo generate，生成 public/
npm run server       # 本地预览，默认 http://localhost:4000
```

构建前务必先 `npm run clean`，否则可能残留旧文件。CI 中的构建顺序是 `hexo clean && hexo g`。

## 发布新文章工作流

1. 在 `source/_posts/` 下创建中文命名的 `.md` 文件（如 `我的新文章.md`）
2. 编写 front matter 和正文，添加 `<!-- more -->` 截断摘要
3. 图片放入与文章同名的文件夹（如 `source/_posts/我的新文章/图片.png`），文章中用 `![alt](图片.png)` 引用（不加路径前缀）
4. `npm run clean && npm run server` 本地预览
5. 首次构建后检查 `.md`——`hexo-abbrlink` 会自动写入 `abbrlink` 字段，需把这个修改 commit 回仓库
6. Push 到 main/master，CI 自动构建部署

## 文章规范

- Front matter：`title`、`date`（`YYYY-MM-DD HH:mm:ss`）、`tags`（数组）、`categories`（**单值字符串**，非数组）
- `abbrlink` 由插件自动生成（crc32 + hex），首次构建时写入 front matter，**不要手动修改**
- 封面图放 `source/images/`，在 front matter 中用 `cover: images/xxx.png` 引用（相对于 `source/`）
- Post Asset Folder 已启用，图片放文章同名文件夹，用 `![alt](文件名)` 引用（`postAsset: true` 自动解析路径）
- 使用 `<!-- more -->` 标记摘要截断，否则首页显示全文
- 文章中出现 `{%`、`{{`、`%}` 等模板语法字符时，用 `{% raw %}...{% endraw %}` 包裹

## 配置文件结构

- `_config.yml` — Hexo 主配置
- `_config.icarus.yml` — icarus 主题配置（实际生效的主题配置）
- `_config.landscape.yml` — landscape 主题配置（未使用，当前主题为 icarus）
- `scaffolds/` — 新文章模板

## CI/CD

GitHub Actions 在 push 到 main 且涉及 `*.json`、`**.yml`、`**/source/**`、`themes/**`、`.node-version`、`package*.json` 路径时触发，执行 `hexo clean && hexo g` 后将 `public/` 部署至 Cloudflare Pages。修改非源码文件（如 README）不会触发构建。

## 注意事项

- Node 版本要求 v24（`.node-version` 指定 24）
- `public/` 目录在 `.gitignore` 中，不要提交构建产物
- `hexo-neat` 插件会在构建时压缩 HTML/CSS/JS，排除了 `.min.*` 和 live2d 相关文件
- `skip_render` 排除了百度验证文件 `baidu_verify_codeva-7MvbRcSn38.html`
- permalink 格式为 `posts/:abbrlink/`，确保文章 URL 稳定
- icarus 主题已从 npm 切换到本地管理（`themes/icarus/`），需要在 `themes/icarus/` 中运行 `npm install` 安装主题依赖
- `package.json` 中额外保留了 3 个主题依赖（`babel-plugin-inferno`、`@babel/preset-env`、`bulma-stylus`），因为 Babel/JSX 和 Stylus 的模块解析需要它们出现在根 `node_modules/` 中。升级 icarus 版本时需同步更新这 3 个包的版本号
