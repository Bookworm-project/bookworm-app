<script>
  export let query;
  export let schema;
  export let bookworm;
  import Corpus from "./Corpus.svelte";
  import IconButton from '@smui/icon-button/bare.js';

	function clone_limit(i) {
		const new_copy = JSON.parse(JSON.stringify(query.search_limits[i]))
		query.search_limits.splice(i + 1, 0, new_copy)
		query = query
	}

	function remove_limit(i) {
		query.search_limits.splice(i, 1)
		query = query
	}

  $: {
    query.search_limits
    query = query;
  }


</script>

{#each query.search_limits as limits, i}
<div class="corpus">
  {#if query.search_limits.length > 1}
    <IconButton class="material-icons" on:click={() => remove_limit(i)}>delete</IconButton>
  {/if}
  <IconButton class="material-icons" on:click={() => clone_limit(i)}>add</IconButton>
  <Corpus on:bookworm bind:limits={limits} {schema} {bookworm}></Corpus>
</div>
{/each}


<style>
  div.corpus {
    display: flex;
  }
</style>