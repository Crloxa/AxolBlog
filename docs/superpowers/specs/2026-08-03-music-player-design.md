# 侧栏音乐播放器设计

日期:2026-08-03
项目:Fuwaru Astro 博客模板(Astro 5 + Svelte 5)

## 目标

1. 把个人卡片(Profile widget)中 GitHub/Telegram 图标按钮的位置改造成一个音乐播放器,风格与前端一致。
2. GitHub/B站/邮箱 链接移动到侧栏最下方(标签 widget 之下)的新建小区域,图标形式与现有 GitHub 按钮一致。
3. 音乐从 `src/assets/music/` 目录读取,尽量兼容多种音频格式。

## 布局改动

### `src/components/widget/Profile.astro`

- 删除链接图标行(当前第 25-36 行,即 `config.links` 的渲染逻辑)。
- 在 bio 下方新增 `<MusicPlayer />` 组件。

### `src/components/widget/SideBar.astro`

- 在 `<Tag>` widget 之后新增一个小 widget:用 `card-base` 卡片 + 标题栏(与"标签" widget 一致),内部渲染 GitHub / B站 / 邮箱 三个图标按钮。
- 图标按钮复用现有样式:`btn-regular rounded-lg h-10 w-10 active:scale-90`,图标尺寸 `text-[1.5rem]`。
- 链接数据继续从 `profileConfig.links` 读取(由 config 驱动),只是渲染位置移到侧栏底部。

### `src/config.ts`

- `profileConfig.links` 改为三个条目,移除 Telegram:
  - GitHub:`fa6-brands:github` → `https://github.com/yCENzh`
  - B站:`fa6-brands:bilibili` → `https://space.bilibili.com/361736008`
  - 邮箱:`fa6-regular:envelope` → `mailto:3845362991@qq.com`

## 播放器组件 `src/components/widget/MusicPlayer.svelte`

### 曲库扫描

- 使用 `import.meta.glob('../../assets/music/*.{mp3,ogg,wav,m4a,flac,aac}', { eager: true })`,构建时自动扫描。
- 支持格式:mp3 / ogg / wav / m4a / flac / aac。
- 曲目标题 = 文件名去掉扩展名(`decodeURIComponent` 处理特殊字符)。
- 用户往 `src/assets/music/` 丢文件、重新构建后自动收录,无需改代码。

### 状态与行为

- 状态:`currentIndex`、`isPlaying`、`mode`、`volume`。
- 播放模式(循环切换:顺序 → 循环 → 随机 → 单曲循环 → 顺序):
  - 顺序:播完列表停止。
  - 循环:列表首尾相接。
  - 随机:随机选曲,避免与当前曲重复。
  - 单曲循环:单曲重复。
- 上一首:当前播放超过 3 秒则回到开头,否则切到上一首。
- 下一首:切到下一首(随机模式下随机选曲)。
- 播放/暂停、音量滑条。
- 音量与播放模式持久化到 localStorage,刷新后恢复。

### 空状态

- `src/assets/music/` 无文件时显示"暂无音乐"占位,组件不报错。

### 技术要点

- 用 HTML5 `<audio>` 元素 + Svelte 状态管理。
- 浏览器自动播放策略:初始为暂停态,需用户点击播放。
- swup 页面切换:侧栏不在 swup 容器(`main`/`#toc`/`#series`)内,页面跳转时播放不中断。

## 图标与风格

- 播放器按钮:`btn-regular rounded-lg` + `@iconify/svelte`(与 Search 组件一致)。
  - 图标:`mdi:play`、`mdi:pause`、`mdi:skip-previous`、`mdi:skip-next`、`mdi:shuffle`、`mdi:repeat`、`mdi:repeat-once`、`mdi:playlist-play`、`mdi:volume-high`。
- 侧栏链接:`astro-icon` 的 `Icon`,图标 `fa6-brands:github` / `fa6-brands:bilibili` / `fa6-regular:envelope`。

## 测试

- 放一个测试 mp3 到 `src/assets/music/`(已就绪:`The Green Kingdom - Untitled.mp3`)。
- `pnpm check`(类型检查)。
- `pnpm dev` 验证:播放器出现、能播放/暂停/切歌/调音量/切换模式;侧栏底部三个链接可点。
