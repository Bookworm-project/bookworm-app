<script>
  import Button, {Group, GroupItem, Label, Icon} from '@smui/button';
  import Menu from "@smui/menu";
  import List, {Item, Separator, Text, PrimaryText, SecondaryText, Graphic} from '@smui/list';

  export let required;
  export let value;
  export let title;
  export let possibilities
  let menu;
</script>

<div>
  <Button class="shaped-outlined" variant="outlined"
  on:click={() => menu.setOpen(true)}>{title}: {value}</Button>
  <Menu bind:this={menu}>
  <List>
    {#if true}
    {#if !required}
      <Item on:click={() => value=undefined}><Icon class='material-icons'>delete</Icon>Delete</Item>
      <Separator></Separator>
    {/if}

    {#each possibilities as opt}
      <Item on:click={() => value=opt.value}><Text>{opt.label}</Text></Item>
    {/each}
    {/if}

  </List>
</Menu>
</div>
