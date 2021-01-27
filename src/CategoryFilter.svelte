<script>
  //import Multiselect from './Multiselect.svelte'

  import Chip, {Set, Icon, Text} from '@smui/chips/bare.js';
  import Button from '@smui/button/bare.js';
  export let array;
  export let field;
  export let bookworm;

  let val = "";

  //  Use undefined instead of empty list to keep the syntax smaller.
  let all_labels = array;
  let unincluded_labels = [];
  let expanded = false;

  function expand() {
    expanded = !expanded;
    bookworm.category_labels(field)
      .then(d => {
        all_labels = d.map(d => d[field])
    })
  }

  function add(chip) {
      array.push(chip)
      array = array
  }

</script>

<div class=filterset>
<!--
    <Select label={field} onchange={}>
    {#each all_labels as lab}
        <Option value={lab}>lab</Option>
    {/each}
</Select>
-->
<div>
{field}: <Set bind:chips={array} primary let:chip input>
    <Chip>
    <Text>{chip}</Text>
    <Icon class="material-icons">cancel</Icon>
    </Chip>
</Set>
</div>
<div>
<Button on:click={expand}>{expanded ? "Close": "Choose"}</Button>
</div>
<div>
{#if expanded}
<hline></hline>
<Set bind:chips={all_labels} let:chip filter bind:selected={array}>
    <Chip>
    <Text>{chip}</Text>
    <!--Icon class="material-icons" on:click={add(chip)}>add</Icon-->
    </Chip>
</Set>
{/if}
</div>
</div>

<style>
    div.filterset {display: flex;}

</style>