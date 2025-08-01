<script lang="ts">
	import { onDestroy, onMount, tick } from 'svelte';
	import { Pane, PaneResizer } from 'paneforge';

	import Drawer from '../common/Drawer.svelte';
	import EllipsisVertical from '../icons/EllipsisVertical.svelte';

	export let show = false;
	export let pane = null;

	export let containerId = 'note-container';

	let mediaQuery;
	let largeScreen = false;

	let minSize = 0;

	const handleMediaQuery = async (e) => {
		if (e.matches) {
			largeScreen = true;
		} else {
			largeScreen = false;
			pane = null;
		}
	};

	onMount(() => {
		// listen to resize 1000px
		mediaQuery = window.matchMedia('(min-width: 1000px)');

		mediaQuery.addEventListener('change', handleMediaQuery);
		handleMediaQuery(mediaQuery);

		// Select the container element you want to observe
		const container = document.getElementById(containerId);

		// initialize the minSize based on the container width
		minSize = Math.floor((400 / container.clientWidth) * 100);

		// Create a new ResizeObserver instance
		const resizeObserver = new ResizeObserver((entries) => {
			for (let entry of entries) {
				const width = entry.contentRect.width;
				// calculate the percentage of 200px
				const percentage = (400 / width) * 100;
				// set the minSize to the percentage, must be an integer
				minSize = Math.floor(percentage);

				if (show) {
					if (pane && pane.isExpanded() && pane.getSize() < minSize) {
						pane.resize(minSize);
					}
				}
			}
		});

		// Start observing the container's size changes
		resizeObserver.observe(container);
	});

	onDestroy(() => {
		mediaQuery.removeEventListener('change', handleMediaQuery);
	});
</script>

{#if !largeScreen}
	{#if show}
		<Drawer
			{show}
			onClose={() => {
				show = false;
			}}
		>
			<div class=" px-3.5 py-2.5 h-screen max-h-dvh flex flex-col">
				<slot />
			</div>
		</Drawer>
	{/if}
{:else if show}
	<PaneResizer
		class="relative flex w-2 items-center justify-center bg-background group bg-white dark:shadow-lg dark:bg-gray-850 border-l border-gray-50 dark:border-gray-850"
	>
		<div class="z-10 flex h-7 w-5 items-center justify-center rounded-xs">
			<EllipsisVertical className="size-4 invisible group-hover:visible" />
		</div>
	</PaneResizer>

	<Pane
		bind:pane
		defaultSize={Math.max(20, minSize)}
		{minSize}
		onCollapse={() => {
			show = false;
		}}
		collapsible={true}
		class=" z-10 "
	>
		{#if show}
			<div class="flex max-h-full min-h-full">
				<div
					class="w-full pl-1.5 pr-2.5 pt-2 bg-white dark:shadow-lg dark:bg-gray-850 z-40 pointer-events-auto overflow-y-auto scrollbar-hidden flex flex-col"
				>
					<slot />
				</div>
			</div>
		{/if}
	</Pane>
{/if}
