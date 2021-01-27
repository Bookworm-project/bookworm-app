<script>
// This allows the definition of a corpus that can be used as a 
// single element for search_limits or compare_limits.
   export let limits;
   export let schema;
   import CategoryFilter from "./CategoryFilter.svelte"
   import IconButton from '@smui/icon-button';
   import Textfield from '@smui/textfield';
   import Paper, {Title, Subtitle} from '@smui/paper';



   export let expanded = false
   export let bookworm;

   const expand = () => { 
      expanded = !expanded 
   }
  
   const all_limits = {}

   for (let {dbname} of schema) {
    all_limits[dbname] = limits[dbname] || []
   }

   all_limits.word = limits.word || []


   // Drop elements for which there are no defined limits.
   $: limits = Object.entries(all_limits)
    .filter(([k, v]) => v.length > 0)
      .reduce((obj, [k, v]) => {
        obj[k] = v;
        return obj;
      }, {});

</script>

<Textfield
   bind:value={all_limits.word[0]}
   label="term(s)"
   style="min-width: 250px;"
></Textfield>

<div class=filter>
<IconButton class="material-icons" on:click={expand}>filter_alt</IconButton>
   <div class=filter-options class:hidden="{!expanded}">
    <Paper color="info" elevation=10>
        <Title>Limit by metadata</Title>

        <Subtitle>
            Categorical data only, for now.
        </Subtitle>
        {#each schema as field}
            {#if field.type == "character"}
            <CategoryFilter {bookworm} bind:array={all_limits[field.dbname]} field={field.name} />
            {/if}
        {/each}
    </Paper>
  </div>
</div>

<style>

div.filter-options {
   position: absolute;
   max-width: 50vw;
   z-index: 99;
}

div.hidden {
   display: none;
}

</style>