<script>
  //import Multiselect from './Multiselect.svelte'

  import Chip, {Set, LeadingIcon as Icon, Text} from '@smui/chips';
  import Button from '@smui/button';
  import { slide } from 'svelte/transition';
  import { bookworm } from './stores.js'
  export let array;
  export let field;

  let all_labels = [];
  let expanded = false;

  function expand() {
    expanded = !expanded;
    //menu.setOpen(true)
    if (all_labels.length == 0) {
        $bookworm.category_labels(field)
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