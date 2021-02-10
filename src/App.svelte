<script>

	import Bookworm from 'bookworm-vega';
	import TopBar from './TopBar.svelte';
	import List, {Item} from '@smui/list/bare.js'
	import Aesthetics from './Aesthetics.svelte';
	import Corpora from './Corpora.svelte';
	import Button from '@smui/button/bare.js';
	import Paper, {Title} from '@smui/paper/bare.js';
	import { Diamonds } from 'svelte-loading-spinners';
	import Dialog, {Content, Title as DialogTitle, Actions} from '@smui/dialog/bare.js';

	export let query;
	export let name;

	// Await a promise for basic bookworm data.
	let initial_load = false;
	let metrics;

	
	if (!window.location.hash) {
		query = {
			"plottype": "linechart",
			"smoothingSpan": 0,
			"database": "subtitles",
			"words_collation": "Case_Sensitive",
			"aesthetic": {
				"x": "date_year",
				"y": "WordsPerMillion",
				"color": "Search"
			},
			"search_limits": [{
				"word": ["terrible"]
			},
			{
				"word": ["great"]
			}]
			}
	} else {
		query = JSON.parse(decodeURI(window.location.hash.slice(1)))
		if (!query.search_limits.length) {
			// For here, always bundle in a search.
			query.search_limits = [query.search_limits]
		}
	}
	if (!query.host) {
		// Just for Ben, for now.
		query.host = window.location.port == 1500 ? "http://localhost:10012/" : window.location.protocol + "//" + window.location.host,
		query = query
	}

	export let bookworm = new Bookworm("#bookworm", query)

	// wait for schema to be ready.
	let misaligned = true;

	function first_draw_with_db() {

		return bookworm.schema.then((schema) => {
			initial_load = true
			console.log("Got schema",{schema})
			metrics = bookworm.metrics;
			bookworm.plotAPI(query).then(
				() => {
					misaligned = false;
				}
			)


		})
	}
	first_draw_with_db()

	let schema = bookworm.schema

	$: {
		// Changes to query mean we're misaligned
		query;
		misaligned = true;
	}

	/*	$: {
		// If the database is changed, everything needs to be regenerated.
		query.database;
		console.log("database changed")
		query = query;
		first_draw_with_db()
	}*/

	let plot_query = query;


	function execute() {
		plot_query = query;
		misaligned = false;
		bookworm.plotAPI(plot_query)
			.then(() => {
				metrics = bookworm.metrics;
			})

	}

	function handle_bookworm(message) {
		if (message.text = "Request Redraw")  {execute()}
		else {alert("Can't handle", message)}
	}
	let show_visual = false;

	function toggle_visual() {
		show_visual = !show_visual
	}
	function runBookwormSearch(event) {
		console.log(event)
		event.stopPropagation()
		search_dialog.open()
		search_results = fetch(event.detail.href_json)
			.then(response => response.json())
			.then(json => {
				if (json.status ==  "error") {
					throw error.message
				}
				console.log(json)
				return json.data.searchstring
				console.log(json)
				return ["foo", "<em>bar</em>"]
			})
	}
	let search_results = Promise.resolve([])
	let search_dialog
</script>

<TopBar bind:query={query}></TopBar>

<Dialog style="min-width:50vw max-width 66vw" bind:this={search_dialog} aria-labelledby="search-results">
	<DialogTitle id="search-results">Search Results</DialogTitle>
	<Content component={List} id="list-content">
		<div class=search-results>
		{#await search_results}
			<Diamonds></Diamonds>
		{:then results}
			{#if results.length == 0}
					AAH
				<p style="color: red"No results for point: try changing smoothing></p>
			{:else}
				{#each results as result}
				<Item>
					{@html result}
				</Item>
				{/each}
			{/if}
	{:catch error}
		<p style="color: red"Error fetching results></p>
	{/await}
	</div>
	</Content>
	<Actions>
		<Button on:click={search_dialog.close()}>
			Close
		</Button>
	</Actions>
</Dialog>

<main>
	<div class=query>
		<div class=query-part>
			<Paper elevation=10>
				<div class="corpora">
					<Title> Create a corpus</Title>
					{#if initial_load}
						<Corpora on:bookworm={handle_bookworm} bind:query={query} {schema} {bookworm} />
					{/if}
				</div>
			</Paper>
		</div>
		<div class=query-part>
			<Paper elevation=10>
				<div class="aesthetics">
					<Title>Visual Representation<Button on:click={toggle_visual}>Edit</Button></Title>
					<div class="aesthetics-edit" style="display:{show_visual ? 'inline': 'none'}">
						{#await bookworm.schema}
							<Diamonds> </Diamonds>
						{:then schema}
							<Aesthetics bind:plottype={query.plottype} bind:aesthetic={query.aesthetic} {metrics} {schema} />
						{/await}
						</div>
				</div>
			</Paper>
		</div>
	</div> <!--query-->
	<div>
		<Button variant="raised" style="width: 60%;" on:click={execute} color="primary" disabled={!misaligned}>
			Draw Chart
			{#if misaligned}
			  <Diamonds color="#FFFFFF" />
			{/if}
		</Button>
	</div>
	<div id="bookworm" on:bookwormSearch={runBookwormSearch}/>
</main>

<style>

	main {
		text-align: left;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}


	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}


	div.search-results {
		min-width: 100vh;
		min-height:  50vh;
		padding-left: 20px;
		padding-right: 20px;
		padding-top: 10px;
		padding-bottom: 50px;
	}

	div.query-part {
		margin-left: 20px;
		margin-top: 20px;
	}

	div.query {
		display: flex;
		flex-wrap: wrap;
	}

	div.corpora {
		max-width:"60%";
		min-width:"33%";
	}

	div.aesthetics {
		max-width:"60%";
		min-width:"33%";
	}
</style>
