<script>

	import Bookworm from 'bookworm-vega';
	import search_limit_labels from 'bookworm-vega/src/search_limit_labels';
	import Multiselect from "./Multiselect.svelte";
	import Corpus from "./Corpus.svelte";
	import { JSONEditor } from 'svelte-jsoneditor';
	import TopBar from './TopBar.svelte';
	import Button from '@smui/button';
	export let bookworm = new Bookworm("#bookworm")
	export let query;
    import IconButton from '@smui/icon-button';

	export let schema = [];



	if (!window.location.hash) {
		query = {
		"plottype": "linechart",
		"smoothingSpan": 0,
		"database": "subtitles",
		"words_collation": "Case_Sensitive",
		"aesthetic": {
			"x": "year",
			"y": "WordsPerMillion"
		},
		"search_limits": [{
			"word": ["terrible"]
		},
		{
			"word": ["great"]
		}]
		}
	} else {
		query = JSON.parse(decodeURI(window.location.hash))
	}
	if (!query.host) {
		// Just for Ben, for now.
		query.host = window.location.port == 1500 ? "http://localhost:10012/" : window.location.protocol + "//" + window.location.host,
		query = query
	}

	function clone_limit(i) {
		console.log(i)
		const new_copy = JSON.parse(JSON.stringify(query.search_limits[i]))
		query.search_limits.splice(i + 1, 0, new_copy)
		query = query
	}

	function remove_limit(i) {
		query.search_limits.splice(i, 1)
		query = query
	}
	
	$: plot = bookworm.plotAPI(plot_query).then(() => {
		schema = bookworm.schemas[bookworm.query.host + '-' + bookworm.query.database]
	}) 

	let misaligned = false;
	$: {
		// Changes to query mean we're misaligned
		query;
		misaligned = true;
	}
	function execute() {
		plot_query = query;
		misaligned = false;
	}

	let plot_query = query;

	misaligned = false;
</script>

<TopBar bind:query={query}></TopBar>

<main>

	<div class=query>
		<div class="corpora">
	{#each query.search_limits as limits, i}
		<div class="corpus">

			{#if query.search_limits.length > 1}
				<IconButton class="material-icons" on:click={() => remove_limit(i)}>delete</IconButton>
			{/if}

			<IconButton class="material-icons" on:click={() => clone_limit(i)}>add</IconButton>

			<Corpus bind:limits={limits} schema={schema} {bookworm}></Corpus>

		</div>
	{/each}
	</div>

	<Button on:click={execute} color="primary" disabled={!misaligned}>Draw Chart</Button>
	</div>
	<div id="bookworm">
		
	</div>

</main>


<style>

	main {
		text-align: left;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}
	div.corpus {
    	display: flex;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
	div.query {
		display: flex;
	}
	div.corpora {
		max-width:"60%";
	}
</style>
