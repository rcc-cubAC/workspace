# 写作规范

## 语言与标点

正文一律用简体中文，遵循 `~/.claude/CLAUDE.md` 里的中文写作规范．要点是全角实心圆点句号 `．`、弯引号、不用破折号、不用箭头符号代替文字．

英文术语采用中文技术写作里的通用形式，没有通用形式的保留英文原貌．例如写“词元”而不是 token，写 GPU 而不是“图形处理单元”，写 Transformer 而不是“变形金刚”．

## 文件与命名

文件名用英文小写加连字符，中文标题写在 front matter 的 `title` 里．中文文件名会生成百分号编码的 URL，难以引用和分享．

不配图的笔记用单文件：

```
content/notes/geo/discrete-laplacian.md
```

需要配图的笔记用页面包目录，图片与 `index.md` 放在同一目录，正文里用相对路径引用：

```
content/notes/geo/discrete-laplacian/
  index.md
  cotangent-weights.png
```

## 栏目

- `content/notes/geo/` 数字几何处理笔记，当前主线，线上地址 `/workspace/notes/geo/`
- `content/notes/` 其他笔记
- `content/papers/` 论文阅读记录

新建几何笔记必须显式指定模板，因为 Hugo 按路径首层目录名匹配模板，写 `notes/geo/` 时它找的是 `archetypes/notes.md`：

```bash
hugo new content --kind geo notes/geo/discrete-laplacian.md
```

其他笔记用默认模板即可：

```bash
hugo new content notes/<slug>.md
```

## front matter

必填 `title`、`date`、`draft`、`tags`．`date` 由模板自动生成，不要手写成将来的日期，否则线上不会输出这个页面．

数学渲染已经在站点级 `params.math` 打开，单篇笔记不需要再声明．

## 标题与段落

标题不加序号，也不用冒号式结构．一级标题由 `title` 生成，正文从二级标题写起．

粗体限制在单个词汇，不要加粗整句，否则粗体的提示作用会被稀释．

## 数学公式

行内公式用 `$...$`，独立公式用单独成行的 `$$`．站点已启用 passthrough，公式内容原样交给 KaTeX，可以放心写下划线与 `\!` 这类记号．

KaTeX 不支持的宏要避开，例如 `\label` 与 `\eqref` 的公式编号引用．需要编号时在正文里用文字说明．

几何处理笔记的记号约定，除非某篇笔记另作声明：

| 对象 | 记号 |
| --- | --- |
| 三角网格 | $M = (V, E, F)$ |
| 顶点位置 | $\mathbf{v}_i \in \mathbb{R}^3$ |
| 向量与矩阵 | 粗体，如 $\mathbf{n}$、$\mathbf{L}$ |
| 离散拉普拉斯算子 | $\mathbf{L}$，质量矩阵 $\mathbf{M}$ |
| 顶点 $i$ 的一环邻域 | $\mathcal{N}(i)$ |
| 半边 | $h$，其对边 $\mathrm{twin}(h)$ |

## 代码块

代码块必须标注语言，方便高亮．伪代码标 `text`．

## 引用来源

引用文献时优先引用期刊或会议的正式发表版本，其次才用预印本．正文里用 Markdown 链接语法：

```markdown
[Attention Is All You Need](https://papers.neurips.cc/paper/7181-attention-is-all-you-need.pdf)
```

引用本仓库或本机的文件时给出完整路径．

## 草稿流程

模板默认 `draft: true`．写作期间用 `hugo server -D` 预览，完成后把 `draft` 改成 `false`，再提交推送．

提交信息用中文，简述改动内容．
