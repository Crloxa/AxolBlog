# 写博客指南（AX Blog / Asterris）

> 本文汇总了本模板 7 篇示范博文展示的所有功能。示范博文已删除，写作时以此为准。

## 1. 文章放哪里

所有文章放在 `src/content/posts/` 目录下，支持子目录组织（建议按分类建文件夹）：

```
src/content/posts/
├── post-1.md
└── post-2/
    ├── cover.png
    └── index.md
```

快捷方式：运行 `npm run new-post -- 文章名`，脚本会在 `src/content/posts/` 生成一个带 front-matter 骨架的 `.md` 文件（不建子目录）。

## 2. Front-matter 字段

```yaml
---
title: 文章标题            # 必填
published: 2026-08-03      # 必填，日期
updated: 2026-08-05        # 可选，更新日期
description: 简短描述       # 可选，显示在首页/列表
image: ./cover.jpg         # 可选，封面图，写法见下
tags: [标签1, 标签2]        # 可选
category: 分类名            # 可选
draft: false               # 可选，true 则不在主页/列表显示
pinned: false              # 可选，true 则置顶
series: 系列名              # 可选，归入某个专栏（series）
---
```

**`image` 封面图的三种写法：**
1. 以 `http://` / `https://` 开头 → 使用网络图片
2. 以 `/` 开头 → 指向 `public/` 目录
3. 无前缀相对路径 → 相对于该 markdown 文件所在目录

## 3. 正文插图

- 放在 `public/`：`![说明](/图片.jpg)`，原图加载。
- 放在 `src/assets/images/`：用相对路径引用，走 astro:assets 优化（自动压缩、响应式）。例如从 `src/content/posts/分类/` 里写 `![说明](../../assets/images/图片.png)`。
- 要控制尺寸可用 HTML：`<img src="/图片.jpg" width="300" alt="说明">`。

## 4. 基础 Markdown

以下写法直接复制进正文即可生效。

### 标题
`#`~`######` 六级标题，`#` 越多字号越小（`##` 及以下的标题会进右侧目录）：

```
# 一级标题
## 二级标题
### 三级标题
```

### 强调
```
**粗体**
*斜体*
~~删除线~~
`行内代码`
```

### 链接与图片
```
[文字](https://example.com)
![替代文字](https://example.com/img.png)
```
图片的三种本地写法见上面「正文插图」。

### 列表
```
无序列表：
- 第一项
- 第二项

有序列表：
1. 第一项
2. 第二项

嵌套列表（子项缩进两格即可）：
- 外层一项
  - 内层一项
    - 更内层一项
```

### 引用
```
> 这是一段引用文字。
```

### 表格
```
| 列 1 | 列 2 |
| ---- | ---- |
| A | B |
```

### 脚注
```
这是一段正文[^1]。

[^1]: 脚注内容显示在文章末尾。
```

### 分隔线
```
---
```

### 数学公式（KaTeX）
```
行内公式：$a^2 + b^2 = c^2$
块级公式（独占一行）：$$E = mc^2$$
```

## 5. 扩展组件（本模板特有）

### 提示框（Admonition）

```
:::note
普通提示，用户需要注意的信息。
:::

:::tip
可选信息，帮助用户更顺利。
:::

:::important
用户成功所需的关键信息。
:::

:::warning
存在潜在风险，需要立即关注。
:::

:::caution
某行动可能带来的负面后果。
:::
```

- 自定义标题：`:::note[自定义标题]`
- GitHub 语法也支持：`> [!TIP]`、`> [!NOTE]` 等

### 剧透（Spoiler）

```
这是内容 :spoiler[被隐藏的 **文字**]！
```

### GitHub 仓库卡片

```
::github{repo="owner/repo"}
```

页面加载时自动从 GitHub API 拉取描述、star、fork、license、头像。

### 链接卡片

```
::link-card{url="https://example.com" title="标题" description="描述" icon="https://.../icon.png"}
```

`icon` 可选。url 需以 `https://` 开头。**注意：多个属性之间用空格分隔，不能用逗号**（本模板的指令解析器只认空格）。

## 6. 嵌入视频

直接粘贴 YouTube / Bilibili 的 iframe 嵌入代码即可：

```html
<iframe width="100%" height="468" src="https://www.youtube.com/embed/视频ID" title="..." allowfullscreen></iframe>
```

## 7. 代码块（Expressive Code）

- 语法高亮：```` ```js ```` 等语言标签
- 编辑器/终端框：```` ```js title="my-file.js" ````（编辑器）、```` ```bash title="..." ````（终端）
- 高亮指定行：```` ```js {1,4,7-8} ````
- 行标记类型：`del={2} ins={3-4}`（删除/插入样式）、```` ```diff ````（diff 语法）
- 行内标记：```` ```js "给定文本" ````
- 行号：```` ```js showLineNumbers ````、`startLineNumber=5`
- 折叠代码段：```` ```js collapse={1-5} ````
- 自动换行：```` ```js wrap ````
- ANSI 彩色终端输出：```` ```ansi ````

## 8. 草稿与发布

- `draft: true` 的文章不会显示在主页/列表，可当草稿用；发布时改为 `false`。

## 9. 常用命令

在项目根目录运行：

| 命令 | 作用 |
| ---- | ---- |
| `npm run dev` | 启动本地开发服务器，默认 http://localhost:4321/，改文件实时刷新 |
| `npm run new-post -- 文章名` | 在 `src/content/posts/` 生成一篇带 front-matter 骨架的新文章 |
| `npm run build` | 构建站点 + 生成站内搜索索引（pagefind），产物在 `dist/` |
| `npm run preview` | 本地预览构建产物（先 `build` 再用） |
| `npm run check` | Astro 类型与结构检查 |

包管理器用 `pnpm` 也等价（`pnpm dev`、`pnpm new-post -- 文章名`）。
