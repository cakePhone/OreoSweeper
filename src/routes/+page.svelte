<script>
	import { onMount } from 'svelte';
	import Cell from './cell.svelte';

	let game = {
		size: 15,
		totalFreeCells: 0,
		difficulty: 2,
		showMines: false,
		field: []
	};

	let firstClick = true;

	function generateField(game) {
		let template = [];

		for (let row = 0; row < game.size; row++) {
			let xrow = [];

			for (let column = 0; column < game.size; column++) {
				xrow.push({
					open: false,
					hasMine: false,
					nearbyMines: 0,
					x: row,
					y: column,
					showNumber: false,
					flagged: false
				});
			}

			template.push(xrow);
		}

		return template;
	}

	function placeMines(clickedX, clickedY) {
		let totalMines = Math.floor(game.size * game.size * game.difficulty * 0.05);

		for (let minesPlaced = 0; minesPlaced < totalMines; minesPlaced++) {
			const placeX = Math.floor(Math.random() * game.size);
			const placeY = Math.floor(Math.random() * game.size);

			const distance = Math.sqrt(Math.pow(clickedX - placeX, 2) + Math.pow(clickedY - placeY, 2));

			const adjCells = adjacentCells(placeX, placeY, false);

			if (!game.field[placeX][placeY].hasMine && distance > Math.sqrt(2)) {
				game.field[placeX][placeY].hasMine = true;

				for (let i = 0; i < adjCells.length; i++) {
					adjCells[i].nearbyMines += 1;
				}
			} else {
				minesPlaced -= 1;
			}
		}
	}

	function adjacentCells(x, y) {
		const adjacentCoordinates = [
			{ x: x - 1, y }, // Left
			{ x: x + 1, y }, // Right
			{ x, y: y - 1 }, // Top
			{ x, y: y + 1 }, // Bottom
			{ x: x - 1, y: y - 1 }, //Top Left
			{ x: x - 1, y: y + 1 }, //Bottom Left
			{ x: x + 1, y: y - 1 }, //Top Right
			{ x: x + 1, y: y + 1 } //Bottom Right
		];

		let cells = [];

		for (let i = 0; i < adjacentCoordinates.length; i++) {
			const adjX = adjacentCoordinates[i].x;
			const adjY = adjacentCoordinates[i].y;

			if (0 <= adjX && adjX < game.field.length && 0 <= adjY && adjY < game.field.length) {
				cells.push(game.field[adjX][adjY]);
			}
		}

		return cells;
	}

	function handleCellClick(cell) {
		if (!cell.open) {
			if (cell.hasMine) {
				game.showMines = true;
				window.alert('You stepped on a heart :c');
				return;
			}

			if (firstClick) {
				placeMines(cell.x, cell.y);
				firstClick = false;
			}

			floodFill(cell.x, cell.y);
			// game.field[cell.x][cell.y].open = true
		}
	}

	function floodFill(x, y) {
		const adjCells = adjacentCells(x, y);

		let freeMines = checkFreeCells();

		if (game.field[x][y].hasMine || game.field[x][y].open || game.field[x][y].nearbyMines > 0) {
			game.field[x][y].open = true;
			game.field[x][y].showNumber = true;
		} else {
			game.field[x][y].open = true;

			for (let i = 0; i < adjCells.length; i++) {
				const adjX = adjCells[i].x;
				const adjY = adjCells[i].y;

				if (!game.field[adjX][adjY].open) {
					floodFill(adjX, adjY);
				}
			}
		}

		if (freeMines <= 1) {
			game.showMines = true;
			setTimeout(() => {
				window.alert('You won');
			}, 10);
		}
	}

	function checkFreeCells() {
		let freeMines = 0;

		for (let row = 0; row < game.size; row++) {
			for (let collumn = 0; collumn < game.size; collumn++) {
				if (!game.field[row][collumn].hasMine && !game.field[row][collumn].open) freeMines += 1;
			}
		}

		return freeMines;
	}

	function reset() {
		window.location.reload();
	}

	onMount(() => {
		game.field = generateField(game);
	});

	$: game.field = firstClick ? generateField(game) : game.field;
</script>

<div
	class="box-border flex h-screen w-full flex-grow flex-col p-5
        lg:flex-row lg:justify-center"
>
	<div
		class="flex w-full flex-shrink-0 flex-col
          lg:h-full lg:w-fit lg:pr-10"
	>
		{#if firstClick}
			<div class="inline-flex h-fit w-full space-x-2 py-2">
				<label for="size" class="w-fit text-xl text-slate-100">Size of field</label>
				<input id="size" type="number" bind:value={game.size} max="25" min="10" />
			</div>
			<div class="inline-flex h-fit w-full space-x-2 py-2">
				<label for="difficulty" class="w-fit text-xl text-slate-100">Difficulty</label>
				<select
					id="difficulty"
					on:change={(e) => {
						game.difficulty = e.target.value;
					}}
				>
					<option value="1">Easy</option>
					<option selected value="2">Normal</option>
					<option value="3">Hard</option>
				</select>
			</div>
		{:else}
			<div class="inline-flex h-fit w-full space-x-2 py-2">
				<label for="size" class="w-fit text-xl text-slate-100">Size of field</label>
				<input id="size" type="number" disabled value={game.size} max="25" min="10" />
			</div>
			<div class="inline-flex h-fit w-full space-x-2 py-2">
				<label for="difficulty" class="w-fit text-xl text-slate-100">Difficulty</label>
				<select
					disabled
					id="difficulty"
					value={game.difficulty}
					on:change={(e) => {
						game.difficulty = e.target.value;
					}}
				>
					<option value="1">Easy</option>
					<option selected value="2">Normal</option>
					<option value="3">Hard</option>
				</select>
			</div>
		{/if}
		<button
			class="h-fit w-fit rounded-full bg-slate-600 px-6 py-3 text-slate-100 transition-all hover:bg-red-600 hover:text-red-200
             lg:w-full"
			on:click={() => reset()}
			>Reset
		</button>
	</div>
	<div
		class="mt-5 box-border grid aspect-square
           lg:mt-0 lg:h-[90%] lg:w-fit"
		id="grid"
		style="--size: {game.size}"
	>
		{#each game.field as row}
			{#each row as cell}
				<Cell
					{cell}
					showMines={game.showMines}
					on:cell-click={(e) => handleCellClick(e.detail.cell)}
				/>
			{/each}
		{/each}
	</div>
</div>

<style>
	#grid {
		grid-template-rows: repeat(var(--size), calc(100% / var(--size)));
		grid-template-columns: repeat(var(--size), 1fr);
		gap: 2px;
		max-width: calc(100% - 28px);
	}
</style>
