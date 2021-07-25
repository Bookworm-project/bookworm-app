<script>
  import { query } from './stores.js'
  import Corpus from "./Corpus.svelte";
  import IconButton from '@smui/icon-button';

	function clone_limit(i) {
		const new_copy = JSON.parse(JSON.stringify(limits[i]))
		limits.splice(i, 0, new_copy)
    $query.search_limits = limits
 	}

	function remove_limit(i) {
		limits.splice(i, 1)
		$query.search_limits = limits
	}

  $ : limits = $query.search_limits;
    
</script>
{#if limits && limits.length}
  {#each limits as limit, i}
    <div class="corpus">
      {#if limits.length > 1}
        <IconButton class="material-icons" 
            on:click={() => remove_limit(i)}>delete</IconButton>
      {/if}
      <IconButton class="material-icons"
          on:click={() => clone_limit(i)}>add</IconButton>
        <Corpus on:bookworm limit_num={i}></Corpus>
      </div>
  {/each}
{/if}
<style>
  div.corpus {
    display: flex;
  }
</style>