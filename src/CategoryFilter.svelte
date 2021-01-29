<script>
  //import Multiselect from './Multiselect.svelte'

  import Chip, {Set, Icon, Text} from '@smui/chips/bare.js';
  import Button from '@smui/button/bare.js';
  import { slide } from 'svelte/transition';

  export let array;
  export let field;
  export let bookworm;
  import List, {Item, Separator} from '@smui/list/bare.js';
  import MenuSurface from '@smui/menu-surface/bare.js';

  let val = "";

  let all_labels = [];
  let expanded = false;

  function expand() {
    expanded = !expanded;
    //menu.setOpen(true)
    if (all_labels.length == 0) {
        bookworm.category_labels(field)
        .then(d => {
            all_labels = d.map(d => d[field])
        })
    }
  }

  let expander;
</script>

<div class=filterset>
<!--
    <Select label={field} onchange={}>
    {#each all_labels as lab}
        <Option value={lab}>lab</Option>
    {/each}
</Select>
-->
    <Text>{field}: </Text>
    <Button bind:this={expander} on:click={expand}><Icon class="material-icons">filter_alt</Icon></Button>
    <div class="chipset" transition:slide>
         <Set bind:chips={all_labels} let:chip filter bind:selected={array}>
            <Chip style={(expanded || array.indexOf(chip) > -1) ? "display:inline" : "display:none"}>
                <Text>{chip}</Text>
            </Chip>
        </Set>
    </div>

</div>

<style>
    div.filterset {display: flex;}
    div.chipset {min-width: 33vw;}
</style>