<script>
	import { createEventDispatcher } from 'svelte';

	import Icon from '@iconify/svelte';

	const dispatch = createEventDispatcher();
	export let cell;
	export let showMines;
	$: cell;
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div
	class="flex aspect-square items-center justify-center border-4 border-b-slate-700 border-l-slate-500 border-r-slate-700 border-t-slate-500 bg-slate-600"
	class:open={cell.open}
	style="--size: {cell.size};"
	on:click={() => {
		if (!cell.flagged && !showMines) dispatch('cell-click', { cell: cell });
	}}
	on:contextmenu={(e) => {
		if (!cell.open) cell.flagged = !cell.flagged;
		if (cell.open && cell.flagged) cell.flagged = false;
		e.preventDefault();
	}}
>
	<!-- ({cell.x},{cell.y}) -->
	{#if cell.showNumber && !cell.flagged}
		<p
			class="number select-none text-center font-bold"
			class:text-red-300={cell.nearbyMines >= 3}
			class:text-orange-300={cell.nearbyMines === 2}
			class:text-green-300={cell.nearbyMines === 1}
		>
			{cell.nearbyMines}
		</p>
	{/if}

	{#if cell.flagged && !showMines}
		<Icon icon="material-symbols:flag" color="white" width="70%" height="70%" />
	{/if}

	{#if showMines && cell.hasMine}
		<Icon icon="material-symbols:heart-broken" color="red" width="70%" height="70%" />
	{/if}
</div>

<style lang="postcss">
	.open {
		border: none;
	}

	.number {
		font-size: 70%;
	}
</style>
