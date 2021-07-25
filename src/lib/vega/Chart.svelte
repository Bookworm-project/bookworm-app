<script>
  export let spec
  import { onMount } from 'svelte';
  import { bookworm } from '$lib/stores.js'
  import embed from 'vega-embed';
  let mounted = false;
  onMount(() => {    
    mounted = true;
  })

  async function run_embed() {
    const embedded = await embed("#vis", spec)
    // Handle clicks.
    embedded.view.addEventListener('click', (event, item) => {
      if (!item.datum) {return}
      $bookworm.search_results(bookworm.query, item.datum)
    })
  }

  $ : if (mounted && spec) {
    console.log("Embedding", {spec})
    run_embed()
  }

</script>

<div id="vis">

</div>

<style>

</style>