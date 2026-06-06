# 原理手记 / Field Notes

一个缓慢生长的个人技术博客 —— 用交互式长文（中文）讲清楚复杂系统**到底是怎么运作的**。
追求的是「读完真的懂了机制」，而不只是走马观花。

## 结构

无构建步骤、无依赖、无框架。每篇文章都是一个自包含的 HTML 文件（内联 CSS + 原生 JS）。

```
index.html            # archive 封面 / 目录，入口文件，留在根目录
posts/                # 所有文章 HTML
  llm-lifecycle.html             # 大语言模型的一生
  attention-mechanism.html       # 注意力机制，逐矩阵拆解
  backpropagation.html           # 反向传播：梯度如何流动
  kv-cache.html                  # KV 缓存，逐步拆开
  transformer-architecture.html  # Transformer：一个词向量穿行的塔
  rag.html                       # RAG：让模型当场翻书
.claude/              # field-notes-post 写作 skill 与骨架模板
CLAUDE.md             # 给协作者 / Claude Code 的约定与说明
```

`index.html` 从内联脚本里的 `POSTS` manifest 渲染文章列表，并提供客户端搜索 + 标签过滤。
新增文章时只需在 `posts/` 下建文件，并在 `POSTS` 里登记一条 —— 详见 `CLAUDE.md`。

## 本地预览

```bash
python3 -m http.server 8000   # 然后访问 http://localhost:8000/
```

或直接 `open index.html`。唯一的外部依赖是 Google Fonts（CDN），其余全部内联。
