<script>

  export let field;
  export let val;
  export let bookworm;
  import Select, {Option} from '@smui/select/bare.js';

  if (val == undefined) {
    val = []
  }

  let selected = val.length ? val[0] : undefined;

  $: {
    console.warn({selected})
    val = [selected]
    if (selected !== undefined && selected != val[0]) {
      bookworm.message("Request Redraw")
    }

  }
  </script>

{#await bookworm.category_labels(field)}
  ...
  {:then categories}
  <Select label={field} bind:value={selected} style="min-width:350px">
      {#each categories as entr (entr[field])}
          <Option value={entr[field]}>{entr[field]}</Option>
      {/each}
  </Select>
{/await}

