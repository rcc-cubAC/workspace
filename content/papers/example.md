---
title: 示例论文阅读
date: 2026-08-27T09:30:00+08:00
tags:
  - 示例
  - 论文
draft: false
---

这是一篇示例论文阅读笔记，展示推荐的记录结构．确认站点正常后可以删除．

## 基本信息

- 标题：Attention Is All You Need
- 来源：NeurIPS 2017（`https://papers.neurips.cc/paper/7181-attention-is-all-you-need.pdf`）

## 核心思想

提出 Transformer 架构，完全依靠注意力机制建模序列依赖，摒弃循环与卷积结构．注意力的计算方式为

$$
\mathrm{Attention}(Q, K, V) = \mathrm{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V
$$

## 我的评价

（记录自己的判断与疑问．）
