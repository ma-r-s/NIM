<script>
	import { Button } from '$lib/components/ui/button';
	import { Card } from '$lib/components/ui/card';
	import Skull from 'lucide-svelte/icons/skull';

	const initialRocks = 48;

	let rocks = $state(Array(initialRocks).fill('gray')); // Array of rock states: "gray", "blue", "red"
	let message = $state('Choose who starts the game:');
	let gameOver = $state(false);
	let gameStarted = $state(false);

	function computerTurn() {
		if (gameOver) return;

		// Ensure rocks remaining after computer's turn is a multiple of 4
		const remainingRocks = rocks.filter((color) => color === 'gray').length;
		let take = (remainingRocks - 1) % 4;

		// If no optimal move exists, fallback to a random valid number
		if (take === 0) {
			take = Math.floor(Math.random() * Math.min(remainingRocks, 3)) + 1;
		}

		updateRocks('red', take);

		if (rocks.filter((color) => color === 'gray').length === 0) {
			message = `You win!`;
			gameOver = true;
		} else {
			message = `Your turn. Rocks remaining: ${rocks.filter((color) => color === 'gray').length}`;
		}
	}

	function playerTurn(take) {
		if (gameOver || rocks.filter((color) => color === 'gray').length < take) return;

		updateRocks('blue', take);

		if (rocks.filter((color) => color === 'gray').length === 0) {
			message = `You lose!`;
			gameOver = true;
		} else {
			message = `Computer's turn...`;
			setTimeout(computerTurn, 1000); // Small delay for the computer's turn
		}
	}

	function updateRocks(color, count) {
		let updatedCount = 0;
		rocks = rocks.map((rock) => {
			if (rock === 'gray' && updatedCount < count) {
				updatedCount++;
				return color;
			}
			return rock;
		});
	}

	function restartGame() {
		rocks = Array(initialRocks).fill('gray');
		message = 'Choose who starts the game:';
		gameOver = false;
		gameStarted = false;
	}

	function startGame(starter) {
		gameStarted = true;
		if (starter === 'computer') {
			message = "Computer's turn...";
			computerTurn();
		} else {
			message = `Your turn. Rocks remaining: ${rocks.filter((color) => color === 'gray').length}`;
		}
	}
</script>

<div class="flex min-h-screen flex-col items-center bg-gray-100 p-6">
	<Card class="w-full max-w-md bg-white p-6 shadow-lg">
		<h1 class="mb-4 text-center text-2xl font-bold">NIM Game</h1>
		<p class="mb-6 text-center">{message}</p>

		{#if !gameStarted}
			<div class="flex justify-center gap-4">
				<Button onclick={() => startGame('player')} class="bg-blue-500">Player Starts</Button>
				<Button onclick={() => startGame('computer')} class="bg-green-500">Computer Starts</Button>
			</div>
		{:else if !gameOver}
			<div class="flex justify-center gap-2">
				<Button
					onclick={() => playerTurn(1)}
					disabled={message.includes('Computer') ||
						rocks.filter((color) => color === 'gray').length < 1}
					class="bg-blue-500"
				>
					Take 1 Rock
				</Button>
				<Button
					onclick={() => playerTurn(2)}
					disabled={message.includes('Computer') ||
						rocks.filter((color) => color === 'gray').length < 2}
					class="bg-green-500"
				>
					Take 2 Rocks
				</Button>
				<Button
					onclick={() => playerTurn(3)}
					disabled={message.includes('Computer') ||
						rocks.filter((color) => color === 'gray').length < 3}
					class="bg-red-500"
				>
					Take 3 Rocks
				</Button>
			</div>
		{:else}
			<div class="mt-4 flex justify-center">
				<Button onclick={restartGame} class="bg-yellow-500 hover:bg-yellow-600">
					Restart Game
				</Button>
			</div>
		{/if}

		<div class="mt-6 flex flex-wrap justify-center">
			{#each rocks as color, index}
				<div
					key={index}
					class="relative m-1 h-6 w-6 rounded-full"
					style="background-color: {color === 'blue'
						? '#4f86f7'
						: color === 'red'
							? '#e63946'
							: '#adb5bd'};"
				>
					{#if index === rocks.length - 1}
						<Skull class="inset-0 p-1 text-black" />
					{/if}
				</div>
			{/each}
		</div>
	</Card>
</div>
