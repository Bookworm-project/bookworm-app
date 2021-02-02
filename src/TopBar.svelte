<script>
  import MenuSurface, { Anchor } from "@smui/menu-surface/bare.js";
  import Snackbar from '@smui/snackbar/bare.js'
  import TopAppBar, { Row, Section, Title } from "@smui/top-app-bar/bare.js";
  import Button, {Group, GroupItem, Label, Icon} from '@smui/button/bare.js';
  import Dialog, {Content, Actions} from '@smui/dialog';
  import { JSONEditor } from "svelte-jsoneditor";

  export let query;
  // export let metrics;
  // export let schema;

  let menu;
  let formSurface;
  let settingsPanel;
  let copy_alert;

  function link() {
    window.location.hash = encodeURI(JSON.stringify(query))
    navigator.clipboard.writeText(window.location.href)
    copy_alert.open()
  }
  let editor;
  
</script>

<div>
    <Dialog bind:this={editor} aria-labelledby="simple-title" aria-describedby="editor-content">
      <!-- Title cannot contain leading whitespace due to mdc-typography-baseline-top() -->
      <Title id="simple-title">Dialog Title</Title>
      <Content id="simple-content">
        Edit the underlying JSON. This shouldn't usually be necessary!
      </Content>
      <Actions>
        <JSONEditor bind:json={query} />        
      </Actions>
    </Dialog>

  </div>

<TopAppBar variant="static" color="primary">
  <Row>
  <Section>
    <Title>{query.database} Bookworm</Title>
  </Section>

  <Section align="end" toolbar>
    <Button on:click={() => {editor.open()}}>
        <Icon class="material-icons">code</Icon>
        <Label>Edit Query</Label>
    </Button>
    <Button on:click={() => settingsPanel.setOpen(true)}>
        <Icon
        class="material-icons"
        >settings</Icon>
        <Label>Settings</Label>
    </Button>
    <MenuSurface bind:this={settingsPanel}>
        Case sensitivity:
        <label>
        <input
        type="radio"
        bind:group={query.words_collation}
        value="Case_Sensitive"
        />Case Sensitive
        </label>

        <label>
        <input
        type="radio"
        bind:group={query.words_collation}
        value="Case_Insensitive"
        />Case Insensitive
        </label>

    </MenuSurface>
    <Button>
        <Icon class="material-icons" aria-label="Download data">file_download</Icon>
        <Label>Download</Label>
    </Button>
    <Button on:click={link}> 
        <Icon class="material-icons" aria-label="Create a link">link</Icon>
        <Label>Link</Label>
    </Button>
  </Section>
  </Row>
</TopAppBar>

<Snackbar primary bind:this={copy_alert}>
    <Label>Link copied</Label>
</Snackbar>