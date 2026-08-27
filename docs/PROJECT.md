# 笔记站工程说明

## 这个仓库是什么

一个用 Hugo 搭建的中文笔记站，主题为 PaperMod．源码放在 GitHub 公开仓库 `rcc-cubAC/notes`，每次推送到 `main` 由 GitHub Actions 构建并部署到 GitHub Pages，线上地址是 `https://rcc-cubac.github.io/notes/`．

内容以中文笔记为主．当前的写作重点是数字几何处理方向的学习笔记，服务于 AI 图形学与 AIGC 3D 生成的研究工作．

本地工作副本在 `/Users/mathmagic1139/Desktop/github_workspace/notes`．

## 隐藏式发布

仓库是公开的，站点也能直接打开，但刻意不希望被搜索引擎收录．为此做了三件事，改动配置时都不要破坏：

- `hugo.yaml` 里 `disableKinds` 关掉了 RSS 与 sitemap，`enableRobotsTXT` 保持 `false`．
- 站点级 `cascade` 向所有页面下发 `robotsNoIndex: true`，PaperMod 据此在每个页面输出 noindex 标记．PaperMod 只读取页面级的这个参数，写在站点 `params` 里不生效，必须用 `cascade` 下发．
- 根路径 `https://rcc-cubac.github.io/` 保持 404，只有带 `/notes/` 前缀的地址可用．

新增栏目或页面时不要覆盖 `robotsNoIndex`．

## 目录结构

```
content/            正文
  geo/              数字几何处理笔记，当前主线
  posts/            一般笔记
  papers/           论文阅读记录
  archives.md       归档页
  search.md         站内搜索页
archetypes/         新建内容的模板，按栏目名匹配
layouts/partials/
  extend_head.html  KaTeX 的样式与脚本，由页面参数 math 控制
themes/PaperMod/    主题，git submodule
docs/               本文档与写作规范
.github/workflows/  部署工作流
```

## 本地预览与构建

首次克隆必须先初始化主题子模块，否则构建会失败：

```bash
git submodule update --init --recursive
```

日常预览与构建：

```bash
hugo server -D          # 本地预览，-D 表示包含草稿
hugo --gc --minify      # 完整构建，产物在 public/
```

## 部署

推送到 `main` 会触发 `.github/workflows/hugo.yml`，工作流用 Hugo 0.165.0 extended 构建，再由 `deploy-pages` 发布．升级本地 Hugo 时要同步改工作流里的 `HUGO_VERSION`，避免本地能构建而线上失败．

## 已知问题

**未来时间戳**．front matter 里的 `date` 如果晚于构建时刻，Hugo 默认不输出这个页面，线上就是看不到．写作时不要手写将来的日期，也要注意时区导致的几小时偏差．

**数学公式**．goldmark 会把 `\!` 的反斜杠吃掉，也会把成对下划线当成斜体标记，公式因此渲染出错．`hugo.yaml` 已经启用 goldmark 的 passthrough 扩展，把公式原样交给 KaTeX．改动 `markup` 配置时不要删掉这一段．

**构建产物**．`public/` 与 `resources/_gen/` 已经写进 `.gitignore`，不要提交．
