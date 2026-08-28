# workspace

Hugo 加 PaperMod 搭的中文笔记站，公开仓库，但刻意不被搜索引擎收录．线上地址 `https://rcc-cubac.github.io/workspace/`，当前写作重点是数字几何处理．

动手前先读这两份文档：

@docs/PROJECT.md
@docs/CONVENTIONS.md

常用命令：

```bash
git submodule update --init --recursive                  # 首次克隆后初始化主题
hugo new content --kind geo notes/geo/<slug>.md          # 新建几何处理笔记
hugo server -D                                           # 本地预览，含草稿
```

几条硬性要求：

- 中文写作遵循 `docs/CONVENTIONS.md`，标点用全角实心圆点句号 `．`、弯引号，不用破折号与箭头符号．
- 不要改动 `hugo.yaml` 里与 noindex、passthrough 相关的配置，理由见 `docs/PROJECT.md`．
- 不要改仓库名，它等于线上地址前缀 `/workspace/`．
- front matter 的 `date` 不能晚于构建时刻，否则页面不会输出．
- 提交信息用中文，不要写入 Claude 署名．
