<script>

	import Bookworm from 'bookworm-vega';
	import TopBar from './TopBar.svelte';
	import Aesthetics from './Aesthetics.svelte';
	import Corpora from './Corpora.svelte';
	import Button from '@smui/button/bare.js';
	import Paper, {Title} from '@smui/paper/bare.js';
	import { Diamonds } from 'svelte-loading-spinners';
	export let query;

	// Await a promise for basic bookworm data.
	let initial_load = false;
	let metrics;

	export let schema = undefined;
	
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

	function first_draw_with_db() {
		return bookworm.schema.then(() => {
		initial_load = true
		schema = bookworm.schemas[bookworm.query.host + '-' + bookworm.query.database]
		metrics = bookworm.metrics;
		bookworm.plotAPI(query)
		})
	}
	let first_draw = first_draw_with_db()

	let misaligned = false;

	$: {
		// Changes to query mean we're misaligned
		// TODO--don't start misaligned.
		query;
		misaligned = true;
	}

	$: {
		// If the database is changed, everything needs to be regenerated.
		query.database;
		query = query;
		first_draw = first_draw_with_db()
	}
	let plot_query = query;


	function execute() {
		plot_query = query;
		misaligned = false;
		bookworm.plotAPI(plot_query)
			.then(() => {
			schema = bookworm.schemas[bookworm.query.host + '-' + bookworm.query.database]
			metrics = bookworm.metrics;
			})
	}

	let show_visual = false;

	function toggle_visual() {
		show_visual = !show_visual
	}

</script>

<TopBar bind:query={query}></TopBar>

<main>
	<div class=query>
		<div class=query-part>
			<Paper elevation=10>
				<div class="corpora">
					<Title> Create a corpus</Title>
					{#if initial_load}
						<Corpora bind:query={query} {schema} {bookworm} />
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
		<Button variant="raised" style="width: 60%;" on:click={execute} color="primary" disabled={!misaligned}>Draw Chart</Button>
	</div>
	<div id="bookworm" />
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
