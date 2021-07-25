<script>
  import TopAppBar, { Row, Section, Title } from "@smui/top-app-bar";
  import Button, {Label, Icon} from '@smui/button';
  import Dialog, {Content} from '@smui/dialog';
  import Tab from '@smui/tab';
  import TabBar from '@smui/tab-bar';
	import { Diamonds } from 'svelte-loading-spinners';
	import { slide, fly } from 'svelte/transition';

//  import { JSONEditor } from "svelte-jsoneditor";
  import Paper, {Title as PaperTitle} from '@smui/paper';
  import Corpora from './Corpora.svelte';
	import Aesthetics from './Aesthetics.svelte';
  import Radio from '@smui/radio';
  import FormField from '@smui/form-field';

  import { query, bookworm } from './stores.js'

  let menu;
  let formSurface;
  let settingsPanel;
  let copy_alert;
  let collations = [
    {"value": "Case_Insensitive",
    "label": "Insensitive"},
    {"value": "Case_Sensitive",
    "label": "Sensitive"}
  ]

  function clone(query) {
    return JSON.parse(JSON.stringify(query))
  }

  function link() {
    if ($query['host'].match(/localhost/)) {
      $bookworm.message({type: "warning", text: "Changing hostname to benschmidt.org"})
      $query['host'] = window.location.protocol + "//benschmidt.org"
    }

    window.location.hash = encodeURI(JSON.stringify($query))
    navigator.clipboard.writeText(window.location.href)
    copy_alert.open()
  }
  let editor;
  

  let  tabs =   [{
      icon: "search",
      label: "Design a query",
      controls: "query"
    },
    {
      icon: "scatter",
      label: "Design chart",
      controls: "chart"
    },
    {
      icon: "code",
      label: "Edit code",
      controls: "editor"
    },
    {
      icon: "settings",
      label: "Settings",
      controls: "settings"
    }
  ]
  let active_tab = tabs[0];
  let dialog_open = false;
  $: {
    if (active_tab.controls === "editor") {
      dialog_open = true;
    } 
  }
</script>

<div>

</div>

<TopAppBar variant="static" color="secondary">
  <Row>
  <Section>
    <Title>{$query.database } Bookworm</Title>
  </Section>
  <Section>
  <TabBar tabs={tabs} let:tab key={tab => tab.controls} bind:active={active_tab}>
    <Tab {tab} stacked={true} indicatorSpanOnlyContent={true}
         tabIndicator$transition="fade">
      <Icon class="material-icons">{tab.icon}</Icon>
      <Label>{tab.label}</Label>
    </Tab>
  </TabBar>
  </Section>

  <Section align="end" toolbar>
    <Button on:click={() => {dialog_open = true}}>
        <Icon class="material-icons">code</Icon>
        Edit Query
    </Button>

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
<div class="query">
  {#if $bookworm && active_tab.controls == "query"}
  {#await $bookworm.schema}
    Waiting for information from server...<Diamonds />
  {:then unused }
  <div class="corpora" transition:slide>
    <Corpora on:bookworm />
  </div>
{/await}
  {/if}

  {#if $bookworm && active_tab.controls == "chart"}
    {#await $bookworm.schema}
      <Diamonds />
    {:then schema}
    <div class="aesthetics" transition:slide>
      {#if $query.plottype}
        <Aesthetics bind:plottype={$query.plottype} bind:aesthetic={$query.aesthetic} metrics={bookworm.metrics} />
        {/if}
    </div>
    {/await}
  {/if}


  {#if active_tab.controls == "settings"}
    Case       
    {#each collations as option}
      <FormField>
        <Radio bind:group={$query.words_collation} value={option.value} />
        <span slot="label">{option.label}</span>
      </FormField>
    {/each}


  {/if}
<Dialog bind:this={editor} open={dialog_open} aria-labelledby="simple-title" aria-describedby="simple-content">

  <Paper elevation=10>
    <Content id="simple-content">
      Edit the API driving the visualization. This shouldn't usually be necessary!
    </Content>
<!--    <JSONEditor bind:json={query} />   -->
  </Paper>
</Dialog>
</div>

<!--
<Snackbar primary bind:this={copy_alert}>
    <Label>Link copied</Label>
</Snackbar>
-->

<style>

	div.query-part {
		margin-left: 20px;
		margin-top: 20px;
	}

	div.query {
		display: flex;
    flex-wrap: wrap;
    min-width:80vw;

	}

	div.corpora {
		max-width:80%;
    min-width:50%;
	}

	div.aesthetics {
		max-width:80%;
    min-width:50%;
	}
</style>