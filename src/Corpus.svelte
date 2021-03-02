<script>
// This allows the definition of a corpus that can be used as a 
// single element for search_limits or compare_limits.
   import CategoryFilter from "./CategoryFilter.svelte"
   import IconButton from '@smui/icon-button/bare.js';
   import Textfield from '@smui/textfield/bare.js';
   import { Title } from "@smui/paper/bare.js";
   import MenuSurface from "@smui/menu-surface/bare.js";
   import SimpleFilter from './SimpleFilter.svelte';
   import { createEventDispatcher } from 'svelte';
   const dispatch = createEventDispatcher();
   function requestRedraw() {
      dispatch('bookworm', {"text": "Request Redraw"});
    }

   export let bookworm;
   export let limits;
   export let schema;
   export let query;

   console.log("CORPUS", {query})
   function label(q) {
    const copy = JSON.parse(JSON.stringify(q))
    copy.word = undefined;
    const lab = bookworm.query_labeller([copy])[0]
    return lab === "" ? "All texts" : lab
  }


   let menu;

   const expand = () => { 
     menu.setOpen(true)
   }
   
   const all_limits = {...limits}

    schema.then( (schema_entries) => {   
     for (let {dbname} of schema_entries) {
        all_limits[dbname] = limits[dbname] || []
     }
     all_limits.word = limits.word || []
   })

   // Drop elements for which there are no defined limits.
   $: limits = Object.entries(all_limits)
    .filter(([k, v]) => v.length > 0)
      .reduce((obj, [k, v]) => {
        obj[k] = v;
        return obj;
      }, {});


    function handleEnter(event) {
		if (event.keyCode==13) {
            requestRedraw()
        }
  }
</script>

<Textfield
   bind:value={all_limits.word[0]}
   on:keyup={handleEnter}
   label="term(s)"
   style="min-width: 150px;"
></Textfield>
<div class=filter>
  {#if !query.ui || !query.ui.simple_filter}
    <div class="slug">
        <span class=mdc-typography--body1>
            {label(limits)}
        </span>
    </div>
  {/if}
{#await schema}
  ...
{:then schema_entries}
{#if query.ui && query.ui.simple_filter}
{#each query.ui.simple_filter as field (field)}
<div class=simple-filter>
  <SimpleFilter {bookworm} {field} bind:val={all_limits[field]}>

  </SimpleFilter>
</div>

{/each}
{:else}
<IconButton class="material-icons" on:click={expand}>filter_alt</IconButton>
<MenuSurface bind:this={menu} color="info" elevation=10>
    <div style="min-width: 33vw; margin: 1em; display: flex; flex-direction: column; align-items: flex-start;">
        <Title>Limit by metadata</Title>
        {#each schema_entries as field}
        {#if field.type == "character"}
            <CategoryFilter {bookworm} bind:array={all_limits[field.dbname]} field={field.name} />
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