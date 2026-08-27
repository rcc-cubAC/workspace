---
title: 示例笔记
date: 2026-08-27T09:00:00+08:00
tags:
  - 示例
draft: false
---

这是一篇示例笔记，用来展示常用的写作元素．确认站点正常后可以删除．

## 代码块

```python
def hello():
    print("你好，世界")
```

## 数学公式

行内公式：$e^{i\pi} + 1 = 0$．

独立公式：

$$
\mathcal{L} = \mathbb{E}_{x \sim p_\text{data}} \left[ \log p_\theta(x) \right]
$$

## 列表与引用

- 第一项
- 第二项

> 这是一段引用文字．

## 新建笔记的方法

在仓库目录下执行：

```bash
hugo new content posts/我的新笔记.md
```

写完后把开头的 `draft: true` 改为 `draft: false`，提交并推送即可自动发布．
