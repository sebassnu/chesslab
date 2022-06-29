<script>

    export let buttonAction = "async () => {await newPuzzle();board.position(position);}"
    export let lostgame = false;

</script>

<div style= "position: fixed; left:0">
    {#if lostgame}
    <h1>You lost!</h1>
    {/if}
    <button on:click={buttonAction}>New game</button>

</div>

<style>

</style>