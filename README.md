# Axol's Blog

基于 [Astro](https://astro.build) 构建的个人博客，主题为 [Fuwari](https://github.com/saicaca/fuwari) 的 Astro 移植版 [Asterris](https://github.com/yCENzh/Fuwari-yCENzh) 的**二次~~魔改~~开发**的博客。

没什么好说的，这是一个只要会复制粘贴和对 agent 提要求的人都会做的项目

![Preview](preview.png)

## 特性

- 基于 Astro 5 + Tailwind CSS + Svelte 的静态站点，开箱即用、可静态托管
- 支持 Markdown / MDX 博文，内置 KaTeX 数学公式、脚注、表格、折叠块
- 自定义 link-card / github 指令，可插入歌曲卡片、仓库卡片等
- 专栏（Series）按 category + series 分组归档，博文草稿（draft）在构建时自动隐藏
- 集成 Giscus 评论（GitHub Discussions）
- 内置 pagefind 全文搜索与 RSS 订阅
- swup 无刷新页面切换

## 本地开发

```bash
# 安装依赖（pnpm）
pnpm install

# 启动开发服务器
npm run dev

# 新建一篇博文
npm run new-post -- 文章名

# 生产构建（会生成 pagefind 索引）
npm run build

# 本地预览构建产物
npm run preview
```

更详细的使用指南见 [blog_guidance.md](./blog_guidance.md)。

## 部署地址
```
https://axolblog.axolrc.workers.dev/
```

## 声明

本博客基于开源主题 [Asterris](https://github.com/yCENzh/Fuwari-yCENzh)（[Fuwari](https://github.com/saicaca/fuwari) 的 Astro 移植版）进行二次开发，在原作者基础上做了大量自定义修改。

- 原版主题（Astro 移植版）：[yCENzh/Fuwari-yCENzh](https://github.com/yCENzh/Fuwari-yCENzh)
- 上游主题（原始版）：[saicaca/fuwari](https://github.com/saicaca/fuwari)
- License：MIT（见 [./LICENSE](./LICENSE)）

感谢以上开源作者的工作。
