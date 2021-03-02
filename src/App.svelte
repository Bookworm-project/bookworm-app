<script>

	import Bookworm from 'bookworm-vega';
	import TopBar from './TopBar.svelte';
	import List, {Item} from '@smui/list/bare.js'
	import Button from '@smui/button/bare.js';
	import { Diamonds } from 'svelte-loading-spinners';
	import Dialog, {Content, Title as DialogTitle, Actions} from '@smui/dialog/bare.js';
	import Snackbar, {Label} from '@smui/snackbar/bare.js';
	export let query;
	export let name;

	// Await a promise for basic bookworm data.
	let initial_load = false;

	
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
		query.host = window.location.protocol + "//" + window.location.host,
		query.host = query.host.replace("movies.benschmidt.org", "benschmidt.org")
		if (window.location.port == "1500") {
			query.host = query.host.replace("1500", "10012")
		}
		query = query
	}

	export let bookworm = new Bookworm("#bookworm", query)

	// wait for schema to be ready.
	let misaligned = true;

	function first_draw_with_db() {

		return bookworm.schema.then((schema) => {
			initial_load = true
			console.log("Got schema",{schema})
			bookworm.plotAPI(query).then(
				() => {
					misaligned = false;
				}
			)


		})
	}
	first_draw_with_db()

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

	}

	let active_message = {text: "", type:"info"};
	let alertbar;

	function handle_bookworm(message) {

		if (message.text == "Request Redraw")  {
			execute()
			return
		}
		else if (message.detail) {
			if (message.detail.text == "Request Redraw") {
				execute()
				message.stopPropagation()
				return;
			}
			// It might be a DOM event from the core module.
			if (message.detail.type == "search") {
				console.warn("Running search")
				runBookwormSearch(message)
			}

			else {
				active_message = message.detail
				alertbar.open()
				message.stopPropagation()
			}
		}
		else {alert("Can't handle", message)}
		return
	}
	let show_visual = false;

	function toggle_visual() {
		show_visual = !show_visual
	}

	function runBookwormSearch(event) {
		search_dialog.open()
		event.stopPropagation()
		search_results = fetch(event.detail.href_json)
			.then(response => response.json())
			.then(json => {
				if (json.status ==  "error") {
					throw error.message
				}
				console.log(json)
				return json.data.searchstring
			})
	}
	let search_results = Promise.resolve([])
	let search_dialog
</script>

<TopBar on:bookworm={handle_bookworm} {bookworm} bind:query></TopBar>

<Dialog style="min-width:66vw max-width 90vw" bind:this={search_dialog} aria-labelledby="search-results">
	<DialogTitle id="search-results">Search Results</DialogTitle>
	<Content component={List} id="list-content" style="min-width:66vw">
		<div class=search-results>
		{#await search_results}
			<div style="min-width:200px">
				<Diamonds></Diamonds>
			</div>
		{:then results}
			{#if results.length == 0}
				<p style="color: red">No results for search term in point: try changing smoothing</p>
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

		</div>
		<div class=query-part>
		</div>
	</div> <!--query-->
	<div>
		<Button variant="raised" style="width: 60%;" on:click={execute} color="primary" disabled={!misaligned}>

			{#if misaligned}
				Redraw Chart
				<Diamonds color="#FFFFFF" />
			{:else}
				Click on the chart below to see individual texts.
			{/if}
		</Button>
	</div>
	<div id="bookworm" on:bookworm={handle_bookworm}/>
</main>

<Snackbar
	primary={active_message.type=="info"}
	warning={active_message.type=="warning"}
	error={active_message.error=="error"}
	secondary={active_message.debug=="secondary"}
	bind:this={alertbar}>
	<Label>
		{active_message.text}
	</Label>
</Snackbar>

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

</style>
