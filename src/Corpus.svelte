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

    export let focused = false;

    const expand = () => { 
        console.log(expanded)
        expanded = !expanded 
    }
  
    let all_limits = {...limits}
    
    $: limits = Object.entries(all_limits)
      .filter(([k, v]) => v != [])
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

<span class=transition>in</span>

<div class=filter>
<IconButton class="material-icons" on:click={expand}>filter_alt</IconButton>

    <div class=filter-options class:hidden="{!expanded}">
    <Paper color="info" elevation=10>
        <Title>Limit by metadata</Title>

        <Subtitle>
            Type comma-separated words into the relevant fields.
            TODO; make this fetch the actual possible values.
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

span.transition {
  height: 100px;
  line-height: 100px;
  white-space: nowrap;

}

div.filter-options {
    position: absolute;
    max-width: 50vw;
    z-index: 99;
}

div.hidden {
    display: none;
}

</style>