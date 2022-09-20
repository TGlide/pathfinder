<script lang="ts">
	import { browser } from '$app/environment';
	import { getPathOffset, getPathScale, getPointAtLength, getSvgScale } from '$utils/svg';
	import { onMount } from 'svelte';

	let progress = 50;
	let smoothedProgress = 50;
	let frame: number | null = null;
	let autoPlay = false;

	const animate = () => {
		if (!browser) return;

		const wrapper = document.querySelector('.wrapper');
		const path = document.querySelector('svg path') as SVGPathElement;
		const pathFinder = document.querySelector('#path-finder') as HTMLDivElement;
		if (!wrapper || !path || !pathFinder) return;

		if (autoPlay) {
			progress += 0.5;
		}

		if (smoothedProgress !== progress) {
			smoothedProgress = smoothedProgress + (progress - smoothedProgress) * 0.1;
		}

		const renderedProgress = smoothedProgress % 100;

		const scale = getPathScale(path);
		const offset = getPathOffset(wrapper, path);
		const pathLength = path.getTotalLength();
		const { point, angle } = getPointAtLength(path, (pathLength * renderedProgress) / 100, {
			scale,
			offset: {
				...offset,
				angle: 180
			}
		});
		pathFinder.style.transform = `
    translate(calc(${point.x}px - 50%), calc(${point.y}px - 50%))
    rotate(${angle}deg)
    `;

		frame = window.requestAnimationFrame(animate);
	};

	onMount(() => {
		frame = window.requestAnimationFrame(animate);
		return () => frame && window.cancelAnimationFrame(frame);
	});

	const handleInput = (e: Event) => {
		const target = e.target as HTMLInputElement;
		progress = Number(target.value);
	};

	const handleBtnClick = () => {
		autoPlay = !autoPlay;
		if (!autoPlay) {
			progress = progress % 100;
			smoothedProgress = smoothedProgress % 100;
		}
	};
</script>

<div class="wrapper">
	<svg width="512" height="512" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
		<path
			d="M12 22C17.5228 22 22 17.5228 22 12C22 6.47715 17.5228 2 12 2C6.47715 2 2 6.47715 2 12C2 17.5228 6.47715 22 12 22Z"
			stroke="currentColor"
			stroke-width="0.1"
		/>
	</svg>
	<div id="path-finder">😎</div>
	<input type="range" value={smoothedProgress % 100} on:input={handleInput} disabled={autoPlay} />
	<button on:click={handleBtnClick}>{autoPlay ? 'stop' : 'play'}</button>
</div>

<style>
	.wrapper {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 2.5rem;
		position: relative;
	}

	svg {
		max-width: 100%;
	}

	#path-finder {
		position: absolute;
		left: 0;
		top: 0;
		font-size: 3rem;
	}

	input {
		cursor: pointer;
	}

	button {
		border: 1px solid #fff;
		border-radius: 0.25rem;
		cursor: pointer;

		margin-top: 1rem;
		padding: 0.5rem 1rem;
	}
</style>
