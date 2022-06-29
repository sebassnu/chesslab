<script>
	import * as TK from 'telekinesis-js';

	import Count from './components/Count.svelte';

	// const ep = new TK.Entrypoint("ws://localhost:8776");
	const ep = new TK.Entrypoint('wss://telekinesis.cloud').run("@sebassnu/chess_puzzles")
	window.ep = ep;

	let game;

	let el, board;
	let position;
	let ELO;
	let testAnswer;
	let moves = "";
	let evaluation;
	let category = 1;
	let lives = 3;
	let nextmove;
	let lostgame = false;
	let orientation = "w";
	let firstmove;
	let initvisible = true;

	window.lives = lives;
	$: antilives = lives - 3;


	const newPuzzle = async () => {
		[position, ELO, testAnswer, firstmove] = await game.get_puzzle()
		console.log(position, "ELO", ELO, "first move", firstmove);
		moves = "";
		window.tA = testAnswer;
		let checkOrientation = position.split(" ");
		window.checkOrientation = checkOrientation[1];
		window.orientation = orientation;
		console.log(checkOrientation[1],orientation);
		if (checkOrientation[1] == orientation && orientation == "w") {
				console.log("check1");
				orientation = "b";
				board.flip();
			} else if (checkOrientation[1] == orientation && orientation == "b"){
				console.log("check2");
				orientation = "w";
				board.flip();
		};
	};

	const initGame = async () => {
			moves = "";
			board = Chessboard(el, {
				draggable: true,
				onDrop: async (source, target) => {moveSubmitted(source,target)},
				sparePieces: false});
			window.board = board;
	};
	

	const moveSubmitted = async (source,target) => {
		if (source==target) {
			console.log("no move");
		} else {
			console.log("new move",source,target);
			moves = moves + source + target;
			[evaluation, nextmove, lives, category] = await testAnswer(moves);
			if (evaluation == "finished") {
				await newPuzzle();
				board.position(position);
				board.move(firstmove);
				moves = firstmove.slice(0,2) + firstmove.slice(3);
			} else if (evaluation == "correct") {
				board.move(nextmove);
				moves = moves + nextmove.slice(0,2) + nextmove.slice(3);
			} else if (evaluation == "incorrect") {
				moves = firstmove.slice(0,2) + firstmove.slice(3);
				board.position(position);
				board.move(firstmove);
				console.log("incorrect");
			} else if (evaluation == "lost") {
				moves = "";
				board.position("8/8/8/8/8/8/8/8");
				orientation == "w"
				lostgame = true;
				initvisible = true;
			};
		}
	};
</script>

<svelte:head>
	<!-- <script src="/jquery-3.6.0.min.js" on:load={initGame}></script> -->
	<link rel="stylesheet" href="/chessboard-1.0.0.min.css"/>
	<script src="/chessboard-1.0.0.js" on:load={initGame}> </script>
</svelte:head>

<div id="container">

<div id="right-bar">
  {#if initvisible} 
	{#if lostgame}
    <h1 style="margin-top:40px;color:white">Game over!</h1>
    {/if}
    <button style="margin-top:40px" on:click={async () => {
		category = 1;
		lives = 3;
		game = await ep();
		await newPuzzle();
		board.position(position);
		board.move(firstmove);
		moves = firstmove.slice(0,2) + firstmove.slice(3);
		lostgame = false;
		initvisible = false;
	}}>New game</button>
  {:else}	
	{#if orientation=="w"}
	<h2 style="margin-top:10px;color:white">White to play</h2>
	{:else}
	<h2 style="margin-top:10px;color:white">Black to play</h2>
	{/if}

	<h2 style="margin-top:10px;color:white">{ELO}</h2>
	
	<div id="lives">
		{#if lives == 3}
			<h2 style="color:red;">♥</h2>
			<h2 style="color:red;">♥</h2>
			<h2 style="color:red;">♥</h2>
		{:else if lives == 2}
			<h2 style="color:red;">♥</h2>
			<h2 style="color:red;">♥</h2>
			<h2>♡</h2>
		{:else if lives == 1}
			<h2 style="color:red;">♥</h2>
			<h2>♡</h2>
			<h2>♡</h2>
		{/if}
	</div>

	<Count counter={category}/>
  {/if}
</div>

<div id="game" bind:this={el} style="width: 600px; margin: 100px; margin-left:400px;"></div>

</div>

<style>
	#container {
		display: flex;
		background-color: black;
	}

	#right-bar {
		width: 300px;
		max-width: 300px;
		position: fixed;
		height: 100%;
		text-align: center;
		margin: 100px;
		margin-left: 1100px;
	}

	h2 {
		margin: 5px;
	}

	#lives {
		display: flex;
		justify-content: center;
	}

</style>