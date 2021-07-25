<script>
// This allows the definition of a corpus that can be used as a 
// single element for search_limits or compare_limits.
   import CategoryFilter from "./CategoryFilter.svelte"
   import IconButton from '@smui/icon-button';
   import Textfield from '@smui/textfield';
   import { Title } from "@smui/paper";
   import MenuSurface from "@smui/menu-surface";
   import SimpleFilter from './SimpleFilter.svelte';
   import { createEventDispatcher } from 'svelte';
   import { query, bookworm } from './stores.js'
   
   const dispatch = createEventDispatcher();
   function requestRedraw() {
      dispatch('bookworm', {"text": "Request Redraw"});
    }

    
   export let limit_num;
  
   let limits = $query.search_limits[limit_num];


   function label(q) {
    const copy = JSON.parse(JSON.stringify(q))
    copy.word = undefined;
    const lab = $bookworm.query_labeller([copy])[0]
    return lab === "" ? "All texts" : lab
  }

   let menu;

   const expand = () => { 
     menu.setOpen(true)
   }
   
   const verbose_limits = {...limits}
   if (!verbose_limits.word) {
     verbose_limits.word = [];
   }
    $bookworm.schema.then( (schema_entries) => {   
     for (let {dbname} of schema_entries) {
        verbose_limits[dbname] = limits[dbname] || []
     }
     verbose_limits.word = limits.word || []
   })

   // Drop elements for which there are no defined limits.
   $: {
     limits = Object.entries(verbose_limits)
    .filter(([k, v]) => v.length > 0)
      .reduce((obj, [k, v]) => {
        console.log({obj, k, v})
        obj[k] = v;
        return obj;
      }, {});
      console.log("Setting limits", limits)
      $query.search_limits[limit_num] = limits;
}
  function handleEnter(event) {
		if (event.keyCode==13) {
      requestRedraw()
    }
  }
</script>
{#if verbose_limits.word.length == 0}
  <IconButton class="material-icons" on:click={() =>{verbose_limits.word = ["a"];  verbose_limits = verbose_limits}}>translate</IconButton>
{/if}
{#each verbose_limits.word as w }
  <Textfield
    bind:value={w}
    on:keyup={handleEnter}
    label="term(s)"
    style="min-width: 150px;"
  ></Textfield>
{/each}
<div class=filter>
  {#if !$query.ui || !$query.ui.simple_filter}
    <div class="slug">
        <span class=mdc-typography--body1>
            {label(limits)}
        </span>
    </div>
  {/if}
  {#await $bookworm.schema}
    ...
  {:then schema_entries}
    {#if $query.ui && $query.ui.simple_filter}
      {#each $query.ui.simple_filter as field (field)}
      <div class=simple-filter>
        <SimpleFilter {field} bind:val={verbose_limits[field]}>

        </SimpleFilter>
      </div>
      {/each}
    {:else}
    <IconButton class="material-icons" on:click={expand}>filter_alt</IconButton>
    <MenuSurface bind:this={menu} color="info" elevation=10>
      <div style="min-width: 33vw; margin: 1em; display: flex; flex-direction: column; align-items: flex-start;">
        <Title>Limit by metadata</Title>
        {#each schema_entries as field}
        {#if field.type === "character"}
          <CategoryFilter bind:array={verbose_limits[field.dbname]} field={field.name} />
        {/if}
        {/each}
      </div>
    </MenuSurface>
    {/if}
  {/await}
</div>

<style>
  div.slug {
    max-width: 275px;
  }
  .simple-filter {
    min-width: 250px;
    padding-left: 10px;
  }
</style>