# 侧栏音乐播放器 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在个人卡片中新增一个与前端风格一致的音乐播放器(顺序/循环/随机/单曲循环、上一首/播放暂停/下一首、音量调节),并把 GitHub/B站/邮箱 链接移到侧栏最下方的新建小区域。

**Architecture:** 新建 Svelte 组件 `MusicPlayer.svelte`,用 `import.meta.glob` 构建时扫描 `src/assets/music/` 的音频文件生成曲库,用 HTML5 `new Audio()` + Svelte 状态管理控制播放。链接仍由 `profileConfig.links` 驱动,渲染位置从 Profile 移到 SideBar 底部。

**Tech Stack:** Astro 5、Svelte 5(沿用项目里 Search.svelte 的 legacy 语法风格)、@iconify/svelte、astro-icon、Tailwind。

## Global Constraints

- 音乐目录:`src/assets/music/`,支持格式 `mp3/ogg/wav/m4a/flac/aac`(glob 通配符 `{mp3,ogg,wav,m4a,flac,aac}`)
- 播放器功能:播放模式 顺序→循环→随机→单曲循环 循环切换;上一首;播放/暂停;下一首;音量滑条
- 播放模式行为:顺序=播完停止;循环=首尾相接;随机=随机且不重复当前曲;单曲循环=单曲重复
- 上一首:播放超 3 秒则回到开头,否则切上一首
- 链接配置:`GitHub → https://github.com/yCENzh`(`fa6-brands:github`)、`B站 → https://space.bilibili.com/361736008`(`fa6-brands:bilibili`)、`邮箱 → mailto:3845362991@qq.com`(`fa6-regular:envelope`)
- 音量、播放模式、当前曲目持久化到 localStorage;刷新恢复
- SSR 安全:localStorage 访问必须用 `typeof localStorage !== "undefined"` 守卫(Astro 会 SSR 渲染 Svelte 组件)
- 样式:复用 `card-base`、`btn-regular rounded-lg`、CSS 变量(`--primary`、`dark:text-neutral-*`)
- 空状态:`src/assets/music/` 无文件时显示"暂无音乐",不报错
- 浏览器自动播放策略:初始为暂停态,需用户点击播放
- swup 页面切换:侧栏不在 swup 容器内,播放不中断(无需处理)
- 项目无测试框架,验证方式为 `pnpm check`(类型检查)+ `pnpm dev`(手动浏览器验证)

---

### Task 1: 更新 config.ts 链接配置

**Files:**
- Modify: `src/config.ts:67-75`(profileConfig.links 数组)

**Interfaces:**
- Consumes: 无
- Produces: `profileConfig.links` = GitHub + B站 + 邮箱 三个条目(Task 2 的 SideBar 链接区域读取)

- [ ] **Step 1: 修改 config.ts**

把 `profileConfig.links` 中 GitHub + Telegram 两个条目替换为 GitHub + B站 + 邮箱。当前内容(第 67-75 行):

```ts
		{
			name: "GitHub",
			icon: "fa6-brands:github",
			url: "https://github.com/yCENzh",
		},
		{
			name: "Telegram",
			icon: "fa6-brands:telegram",
			url: "https://t.me/yCENzh",
		},/*
```

替换为:

```ts
		{
			name: "GitHub",
			icon: "fa6-brands:github",
			url: "https://github.com/yCENzh",
		},
		{
			name: "Bilibili",
			icon: "fa6-brands:bilibili",
			url: "https://space.bilibili.com/361736008",
		},
		{
			name: "Email",
			icon: "fa6-regular:envelope",
			url: "mailto:3845362991@qq.com",
		},/*
```

- [ ] **Step 2: 类型检查**

Run: `pnpm check`
Expected: PASS(无类型错误;`ProfileConfig.links` 的字段不变)

- [ ] **Step 3: 提交**

```bash
git add src/config.ts
git commit -m "config: 更新个人链接为 GitHub/B站/邮箱"
```

---

### Task 2: 侧栏底部新增链接区域

**Files:**
- Modify: `src/components/widget/SideBar.astro:27-29`(在 `<Tag>` 之后新增)

**Interfaces:**
- Consumes: `profileConfig.links`(Task 1 产出);`WidgetLayout` 组件;astro-icon 的 `Icon`
- Produces: 侧栏最下方渲染三个图标链接按钮

- [ ] **Step 1: 修改 SideBar.astro**

在文件头部 import 增加:

```astro
---
import type { MarkdownHeading } from "astro";
import Categories from "./Categories.astro";
import Profile from "./Profile.astro";
import Tag from "./Tags.astro";
import Series from './Series.astro'
import WidgetLayout from "./WidgetLayout.astro";
import { profileConfig } from "../../config";
import { Icon } from "astro-icon/components";
```

在 `<Tag ...></Tag>` 之后新增链接 widget(仍处于 `#sidebar-sticky` 容器内):

```astro
        <Tag class="onload-animation" style="animation-delay: 200ms"></Tag>
        <WidgetLayout name="链接" id="links" class="onload-animation" style="animation-delay: 250ms">
            <div class="flex flex-wrap gap-2 justify-center mb-1">
                {profileConfig.links.map(item =>
                    <a rel="me" aria-label={item.name} href={item.url} target="_blank" class="btn-regular rounded-lg h-10 w-10 active:scale-90">
                        <Icon name={item.icon} class="text-[1.5rem]"></Icon>
                    </a>
                )}
            </div>
        </WidgetLayout>
```

- [ ] **Step 2: 类型检查**

Run: `pnpm check`
Expected: PASS(`profileConfig.links` 类型匹配 WidgetLayout 使用;Icon name 为字符串)

- [ ] **Step 3: 手动验证**

Run: `pnpm dev`,打开页面,确认侧栏"标签"下方出现"链接"卡片,内有 GitHub / B站 / 邮箱 三个图标按钮。

- [ ] **Step 4: 提交**

```bash
git add src/components/widget/SideBar.astro
git commit -m "feat: 侧栏底部新增 GitHub/B站/邮箱 链接区域"
```

---

### Task 3: 创建 MusicPlayer.svelte 播放器组件

**Files:**
- Create: `src/components/widget/MusicPlayer.svelte`

**Interfaces:**
- Consumes: 无外部依赖;`src/assets/music/` 目录下的音频文件(已存在 `The Green Kingdom - Untitled.mp3`)
- Produces: `MusicPlayer` 组件(Task 4 在 Profile.astro 中渲染)

- [ ] **Step 1: 创建组件文件**

创建 `src/components/widget/MusicPlayer.svelte`,完整代码如下:

```svelte
<script lang="ts">
	import Icon from "@iconify/svelte";

	type PlayMode = "order" | "loop" | "random" | "single";

	const trackModules = import.meta.glob(
		"../../assets/music/*.{mp3,ogg,wav,m4a,flac,aac}",
		{ eager: true, import: "default" },
	) as Record<string, string>;

	const playlist = Object.entries(trackModules).map(([path, url], i) => ({
		url,
		title:
			decodeURIComponent(path.split("/").pop()?.replace(/\.[^.]+$/, "") ?? "") ||
			`曲目 ${i + 1}`,
	}));

	const MODES: PlayMode[] = ["order", "loop", "random", "single"];
	const MODE_ICONS: Record<PlayMode, string> = {
		order: "mdi:playlist-play",
		loop: "mdi:repeat",
		random: "mdi:shuffle",
		single: "mdi:repeat-once",
	};
	const MODE_TITLES: Record<PlayMode, string> = {
		order: "顺序播放",
		loop: "列表循环",
		random: "随机播放",
		single: "单曲循环",
	};

	let currentIndex = 0;
	let isPlaying = false;
	let mode: PlayMode = "loop";
	let volumePercent = 80;
	let audio: HTMLAudioElement | null = null;

	// SSR 安全地恢复本地状态
	if (typeof localStorage !== "undefined") {
		const savedVolume = localStorage.getItem("music-volume");
		const savedMode = localStorage.getItem("music-mode");
		const savedIndex = localStorage.getItem("music-index");
		if (savedVolume !== null) volumePercent = Number(savedVolume);
		if (savedMode !== null && MODES.includes(savedMode as PlayMode))
			mode = savedMode as PlayMode;
		if (savedIndex !== null && playlist.length)
			currentIndex = Math.min(Number(savedIndex), playlist.length - 1);
	}

	function getAudio(): HTMLAudioElement {
		if (!audio) {
			audio = new Audio();
			audio.volume = volumePercent / 100;
			audio.addEventListener("ended", handleEnded);
		}
		return audio;
	}

	function playTrack(index: number) {
		if (!playlist.length) return;
		currentIndex = index;
		if (typeof localStorage !== "undefined")
			localStorage.setItem("music-index", String(index));
		const a = getAudio();
		a.src = playlist[index].url;
		a.play();
		isPlaying = true;
	}

	function togglePlay() {
		if (!playlist.length) return;
		const a = getAudio();
		if (isPlaying) {
			a.pause();
			isPlaying = false;
		} else {
			a.play();
			isPlaying = true;
		}
	}

	function next() {
		if (!playlist.length) return;
		if (mode === "random") {
			playRandom();
		} else {
			playTrack((currentIndex + 1) % playlist.length);
		}
	}

	function prev() {
		if (!playlist.length) return;
		const a = audio;
		if (a && a.currentTime > 3) {
			a.currentTime = 0;
			return;
		}
		playTrack((currentIndex - 1 + playlist.length) % playlist.length);
	}

	function playRandom() {
		if (playlist.length <= 1) {
			playTrack(0);
			return;
		}
		let idx = currentIndex;
		while (idx === currentIndex)
			idx = Math.floor(Math.random() * playlist.length);
		playTrack(idx);
	}

	function cycleMode() {
		const idx = MODES.indexOf(mode);
		mode = MODES[(idx + 1) % MODES.length];
		if (typeof localStorage !== "undefined")
			localStorage.setItem("music-mode", mode);
	}

	function handleEnded() {
		if (mode === "single") {
			if (audio) {
				audio.currentTime = 0;
				audio.play();
			}
		} else if (mode === "random") {
			playRandom();
		} else if (mode === "order") {
			if (currentIndex < playlist.length - 1) {
				playTrack(currentIndex + 1);
			} else {
				isPlaying = false;
			}
		} else {
			playTrack((currentIndex + 1) % playlist.length);
		}
	}

	function onVolumeInput(e: Event) {
		volumePercent = Number((e.target as HTMLInputElement).value);
		if (typeof localStorage !== "undefined")
			localStorage.setItem("music-volume", String(volumePercent));
		if (audio) audio.volume = volumePercent / 100;
	}
</script>

{#if playlist.length}
	<div>
		<div class="flex items-center justify-center gap-2 mb-1.5">
			<button
				class="btn-regular rounded-lg h-9 w-9 active:scale-90"
				title={MODE_TITLES[mode]}
				aria-label={`播放模式：${MODE_TITLES[mode]}`}
				on:click={cycleMode}
			>
				<Icon icon={MODE_ICONS[mode]} class="text-[1.25rem]"></Icon>
			</button>
			<button
				class="btn-regular rounded-lg h-9 w-9 active:scale-90"
				aria-label="上一首"
				on:click={prev}
			>
				<Icon icon="mdi:skip-previous" class="text-[1.25rem]"></Icon>
			</button>
			<button
				class="btn-regular rounded-lg h-12 w-12 active:scale-90"
				aria-label={isPlaying ? "暂停" : "播放"}
				on:click={togglePlay}
			>
				<Icon icon={isPlaying ? "mdi:pause" : "mdi:play"} class="text-[1.75rem]"></Icon>
			</button>
			<button
				class="btn-regular rounded-lg h-9 w-9 active:scale-90"
				aria-label="下一首"
				on:click={next}
			>
				<Icon icon="mdi:skip-next" class="text-[1.25rem]"></Icon>
			</button>
		</div>

		<div class="text-sm text-center text-neutral-500 dark:text-neutral-400 truncate px-1 mb-1.5">
			{playlist[currentIndex].title}
		</div>

		<div class="flex items-center gap-2 px-1">
			<Icon
				icon={volumePercent === 0 ? "mdi:volume-mute" : "mdi:volume-high"}
				class="text-[1.25rem] text-black/40 dark:text-white/40 shrink-0"
			></Icon>
			<input
				type="range"
				min="0"
				max="100"
				value={volumePercent}
				on:input={onVolumeInput}
				class="w-full accent-[var(--primary)]"
			/>
		</div>
	</div>
{:else}
	<div class="text-center text-sm text-neutral-500 dark:text-neutral-400">暂无音乐</div>
{/if}
```

- [ ] **Step 2: 类型检查**

Run: `pnpm check`
Expected: PASS(组件未被使用也能通过检查;确认无 SSR localStorage 报错、glob 类型断言无误)

- [ ] **Step 3: 提交**

```bash
git add src/components/widget/MusicPlayer.svelte
git commit -m "feat: 新增侧栏音乐播放器组件"
```

---

### Task 4: 集成播放器到个人卡片

**Files:**
- Modify: `src/components/widget/Profile.astro:24-36`

**Interfaces:**
- Consumes: `MusicPlayer` 组件(Task 3 产出)
- Produces: 个人卡片 avatar/name/bio 之下显示播放器

- [ ] **Step 1: 修改 Profile.astro**

删除链接图标行(当前第 25-36 行):

```astro
        <div class="flex flex-wrap gap-2 justify-center mb-1">
            {config.links.length > 1 && config.links.map(item =>
                    <a rel="me" aria-label={item.name} href={item.url} target="_blank" class="btn-regular rounded-lg h-10 w-10 active:scale-90">
                        <Icon name={item.icon} class="text-[1.5rem]"></Icon>
                    </a>
            )}
            {config.links.length == 1 && <a rel="me" aria-label={config.links[0].name} href={config.links[0].url} target="_blank"
                                            class="btn-regular rounded-lg h-10 gap-2 px-3 font-bold active:scale-95">
                <Icon name={config.links[0].icon} class="text-[1.5rem]"></Icon>
                {config.links[0].name}
            </a>}
        </div>
```

在文件头部 import 增加 `MusicPlayer`:

```astro
---
import { Icon } from "astro-icon/components";
import { profileConfig } from "../../config";
import { url } from "../../utils/url-utils";
import ImageWrapper from "../misc/ImageWrapper.astro";
import MusicPlayer from "./MusicPlayer.svelte";
```

在 bio 的 `</div>` 之后、外层 `</div>` 之前新增:

```astro
        <div class="px-2 mt-1.5">
            <MusicPlayer></MusicPlayer>
        </div>
```

同时,删除 `config.links` 后 `config` 变量若不再被引用则一并删除该行(`const config = profileConfig;` 及 `import { profileConfig }` 只在引用时保留;若 `profileConfig` 不再使用则删除对应 import,避免未使用警告)。

- [ ] **Step 2: 类型检查**

Run: `pnpm check`
Expected: PASS(确认无未使用 import 报错;MusicPlayer 组件正常引用)

- [ ] **Step 3: 手动验证**

Run: `pnpm dev`,打开首页:
- 个人卡片(头像/名字/bio)下方出现播放器:四个模式/切歌按钮 + 歌名 + 音量滑条
- 点击播放按钮,音乐播放(`The Green Kingdom - Untitled.mp3`),按钮变暂停图标
- 上一首/下一首可用;点击模式按钮在 顺序→循环→随机→单曲循环 间切换
- 音量滑条拖动生效
- 刷新页面后音量与模式保持(localStorage 恢复)

- [ ] **Step 4: 提交**

```bash
git add src/components/widget/Profile.astro
git commit -m "feat: 个人卡片集成音乐播放器"
```

---

### Task 5: 整体验证与收尾

**Files:**
- 无新增/修改(仅验证)

**Interfaces:**
- Consumes: Task 1-4 全部产物

- [ ] **Step 1: 类型检查**

Run: `pnpm check`
Expected: PASS(全项目无类型错误)

- [ ] **Step 2: 构建验证**

Run: `pnpm build`
Expected: 构建成功;`dist/` 中包含音频文件(哈希名)与播放器产物

- [ ] **Step 3: 浏览器完整验证**

Run: `pnpm dev`,逐项确认:
- 侧栏底部"链接"卡片显示 GitHub / B站 / 邮箱 三个图标,点击可跳转
- 个人卡片播放器功能齐全(播放/暂停/上下首/模式切换/音量)
- 空状态不可复现(有测试 mp3),但删除 music 目录文件后应显示"暂无音乐"(可选验证)
- 页面跳转(如点进文章)后播放不中断

- [ ] **Step 4: 汇报**

向用户汇报完成情况,说明测试 mp3 已可播放、如何新增歌曲(丢进 `src/assets/music/` 重新构建)、B站/邮箱链接已生效。
