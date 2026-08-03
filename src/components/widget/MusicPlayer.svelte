<script lang="ts">
	import Icon from "@iconify/svelte";
	import { onMount } from "svelte";

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
	let volumePercent = 10;
	let currentTime = 0;
	let duration = 0;
	let isSeeking = false;
	let showVolume = false;
	let playerRoot: HTMLDivElement | null = null;
	let audio: HTMLAudioElement | null = null;

	onMount(() => {
		if (typeof localStorage !== "undefined") {
			const savedVolume = localStorage.getItem("music-volume");
			const savedMode = localStorage.getItem("music-mode");
			const savedIndex = localStorage.getItem("music-index");
			if (savedVolume !== null) {
				const n = Number(savedVolume);
				if (Number.isFinite(n)) volumePercent = n;
			}
			if (savedMode !== null && MODES.includes(savedMode as PlayMode))
				mode = savedMode as PlayMode;
			if (savedIndex !== null && playlist.length) {
				const n = Number(savedIndex);
				if (Number.isFinite(n))
					currentIndex = Math.min(n, playlist.length - 1);
			}
		}
		const handleDocClick = (e: MouseEvent) => {
			if (playerRoot && !playerRoot.contains(e.target as Node))
				showVolume = false;
		};
		document.addEventListener("click", handleDocClick);
		return () => document.removeEventListener("click", handleDocClick);
	});

	function toggleVolume() {
		showVolume = !showVolume;
	}

	function getAudio(): HTMLAudioElement {
		if (!audio) {
			audio = new Audio();
			audio.volume = volumePercent / 100;
			audio.addEventListener("ended", handleEnded);
			audio.addEventListener("timeupdate", handleTimeUpdate);
			audio.addEventListener("loadedmetadata", handleLoadedMetadata);
			audio.addEventListener("durationchange", handleLoadedMetadata);
		}
		return audio;
	}

	function handleTimeUpdate() {
		if (audio && !isSeeking) {
			currentTime = audio.currentTime;
			duration = audio.duration || duration;
		}
	}

	function handleLoadedMetadata() {
		if (audio) duration = audio.duration || 0;
	}

	function playTrack(index: number) {
		if (!playlist.length) return;
		currentIndex = index;
		if (typeof localStorage !== "undefined")
			localStorage.setItem("music-index", String(index));
		const a = getAudio();
		a.src = playlist[index].url;
		a.play().catch(() => {});
		isPlaying = true;
	}

	function togglePlay() {
		if (!playlist.length) return;
		const a = getAudio();
		if (isPlaying) {
			a.pause();
			isPlaying = false;
		} else {
			if (!a.src) a.src = playlist[currentIndex].url;
			a.play().catch(() => {});
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
				audio.play().catch(() => {});
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

	function onSeekInput(e: Event) {
		isSeeking = true;
		currentTime = Number((e.target as HTMLInputElement).value);
	}

	function onSeekChange(e: Event) {
		const t = Number((e.target as HTMLInputElement).value);
		currentTime = t;
		if (audio) audio.currentTime = t;
		isSeeking = false;
	}

	function formatTime(sec: number): string {
		if (!Number.isFinite(sec) || sec < 0) sec = 0;
		const m = Math.floor(sec / 60);
		const s = Math.floor(sec % 60);
		return `${m}:${String(s).padStart(2, "0")}`;
	}
</script>

{#if playlist.length}
	<div bind:this={playerRoot}>
		<!-- 传输行:模式/上一首/播放暂停/下一首 + 音量按钮(点击弹出竖状slider) -->
		<div class="flex items-center justify-center gap-1.5 mb-1.5">
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
			<div class="relative">
				<button
					class="btn-regular rounded-lg h-9 w-9 active:scale-90"
					title="音量"
					aria-label="音量"
					aria-expanded={showVolume}
					on:click={toggleVolume}
				>
					<Icon icon={volumePercent === 0 ? "mdi:volume-mute" : "mdi:volume-high"} class="text-[1.25rem]"></Icon>
				</button>
				{#if showVolume}
					<div class="absolute bottom-[calc(100%+0.5rem)] left-1/2 -translate-x-1/2 z-20 flex flex-col items-center gap-0.5 rounded-lg bg-white dark:bg-neutral-800 px-1.5 py-1.5 shadow-lg border border-black/5 dark:border-white/10">
						<Icon
							icon={volumePercent === 0 ? "mdi:volume-mute" : "mdi:volume-high"}
							class="text-[0.95rem] text-black/50 dark:text-white/50"
						></Icon>
						<input
							type="range"
							min="0"
							max="100"
							value={volumePercent}
							on:input={onVolumeInput}
							class="volume-slider accent-[var(--primary)]"
						/>
						<span class="text-[0.7rem] leading-none text-black/60 dark:text-white/60 tabular-nums">{volumePercent}</span>
					</div>
				{/if}
			</div>
		</div>

		<div class="text-sm text-center text-neutral-500 dark:text-neutral-400 truncate px-1 mb-1.5">
			{playlist[currentIndex].title}
		</div>

		<!-- 进度条行:当前时间 / seek slider / 总时长(保持不变) -->
		<div class="flex items-center gap-2 px-1">
			<span class="text-xs text-neutral-500 dark:text-neutral-400 tabular-nums shrink-0">
				{formatTime(currentTime)}
			</span>
			<input
				type="range"
				min="0"
				max={Math.max(duration, 0)}
				step="0.1"
				value={currentTime}
				on:input={onSeekInput}
				on:change={onSeekChange}
				class="w-full accent-[var(--primary)]"
			/>
			<span class="text-xs text-neutral-500 dark:text-neutral-400 tabular-nums shrink-0">
				{formatTime(duration)}
			</span>
		</div>
	</div>
{:else}
	<div class="text-center text-sm text-neutral-500 dark:text-neutral-400">暂无音乐</div>
{/if}

<style>
	.volume-slider {
		writing-mode: vertical-lr;
		direction: rtl;
		height: 2.5rem;
		width: 1.5rem;
	}
</style>
