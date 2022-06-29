<script>

    export let counter = 1;


</script>

<h1 style="text-align: center;color: white;">{counter}</h1>

<div id="checkbox">
{#each Array(counter) as i}

    <h2 style="color:green">✓</h2>

{/each}
</div>


<style>

#checkbox {
    display: flex;
}

#checkbox h2 {
    margin:10px;
}


</style>