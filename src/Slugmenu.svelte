<script>

  import Menu from '@smui/menu';
  import {Anchor} from '@smui/menu-surface';
  import List, {Item, Text} from '@smui/list';
  import Button from '@smui/button';

  export let title
  export let options
  export let value
  let anchor;
  let menu;
  let clicked;
  let anchor2Classes = []

  let labs = {}
  for (let {label, value} of options) {
    labs[value] = label;
  }
</script>


  <div style="min-width: 100px;">
    <Button variant="outlined" on:click={() => menu.setOpen(true)}>{title}: {labs[value]}</Button>
    <Menu bind:this={menu} anchorCorner="BOTTOM_LEFT">
      <List>
        {#each options as opt}
        <Item on:SMUI:action={() => value = opt.value}><Text>{opt.label}</Text></Item>
        {/each}
      </List>
    </Menu>
  </div>
