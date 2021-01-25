<script>
    import MenuSurface, {Anchor} from '@smui/menu-surface';
    import TopAppBar, {Row, Section, Title} from '@smui/top-app-bar';
    import IconButton from '@smui/icon-button';
    import Button from '@smui/button';

    import { JSONEditor } from 'svelte-jsoneditor';

    export let query;

    let formSurface;
    let settingsPanel;

</script>

<TopAppBar variant="static" color='primary'>
	<Row>
	  <Section>
		<!-- <IconButton class="material-icons">menu</IconButton> -->
		<Title>{query.database} Bookworm</Title>
	  </Section>

      <Section align="end" toolbar>
        <IconButton class="material-icons" on:click={() => formSurface.setOpen(true)}>code</IconButton>
        <MenuSurface bind:this={formSurface}>
        <JSONEditor bind:json={query} />

        <div style="margin: 1em; display: flex; flex-direction: column; align-items: flex-end;">
            <Button style="margin-top: 1em;" on:click={() => settingsPanel.setOpen(false)}>Submit</Button>
        </div>
        </MenuSurface>
    <IconButton class="material-icons" aria-label="Settings" on:click={() => settingsPanel.setOpen(true)}>settings</IconButton>
    <MenuSurface bind:this={settingsPanel}>
        Case sensitivity: 
        <label>
            <input type=radio bind:group={query.words_collation} value="Case_Sensitive">Case Sensitive
        </label>
    
        <label>
            <input type=radio bind:group={query.words_collation} value="Case_Insensitive">Case Insensitive
        </label>

        <br>
    
        Y axis representing: 
        <select bind:value={query.aesthetic.y}>
            <option value="WordsPerMillion">Uses per Million words</option>
            <option value="WordCount">Total words</option>
        </select>
        <br>
    </MenuSurface>
    <IconButton class="material-icons" aria-label="Download data">file_download</IconButton>
    <IconButton class="material-icons" aria-label="Create a link">link</IconButton>
    <IconButton class="material-icons" aria-label="Share">share</IconButton>
  </Section>
</Row>
</TopAppBar>
