<script>

	export let host;
	export let database;
	export let plottype;

	import Bookworm from 'bookworm-vega';
	import { bookworm, query } from './stores.js';
	import TopBar from './TopBar.svelte';
	import List, {Item} from '@smui/list'
	import Button from '@smui/button';
	import { Diamonds } from 'svelte-loading-spinners';
	import Dialog, {Content, Title as DialogTitle, Actions} from '@smui/dialog';
	import Snackbar, {Label} from '@smui/snackbar';
	import Vega from './vega/Chart.svelte';
	import 'material-icons/iconfont/material-icons.css';
	import { onMount } from 'svelte';

	console.log({host, database})
	$query = {host, database}
	$bookworm = new Bookworm({host, database})

	// is a promise.
	const default_query = $bookworm.default_query();

	default_query.then((q) => {
		console.log(q)
		$query = q;
		redraw()
	})

	onMount( () => {
		window.bookworm = $bookworm;
	})
	console.log("Created")
	let spec = undefined;

	const redraw = async function() {
		await $bookworm.schema;
		spec = await $bookworm.vega_lite_spec($query)
	}

	let active_message = {text: "", type : "info"};
	let alertbar;

	function handle_bookworm(message) {
		console.log("handling event", message)
		if (! $bookworm) return
		if (message.text === "Request Redraw")  {
			console.error("REQUEST!")
			redraw()
			return
		}
		else if (message.detail) {
			if (message.detail.text == "Request Redraw") {
				console.log("handling event", $query)
				redraw()
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

	let search_results = Promise.resolve([])
	let search_dialog
</script>


<TopBar on:bookworm={handle_bookworm}></TopBar>
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
				<p style="color: red"> No results for search term in point: try changing smoothing </p>
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

	<div class=query>
		<div class=query-part>
		</div>
		<div class=query-part>
		</div>
	</div>
	<div>
		<Button variant="raised" style="width: 60%;" on:click={redraw} color="primary" disabled={false}>
			{#if $bookworm === undefined}
				Redraw Chart
				<Diamonds color="#FFFFFF" />
			{:else}
				Click on the chart below to see individual texts.
			{/if}
		</Button>
	</div>
	<div id="bookworm" on:bookworm={handle_bookworm}>
		<Vega {spec} />
	</div>

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
		min-height: 50vh;
		padding-left: 20px;
		padding-right: 20px;
		padding-top: 10px;
		padding-bottom: 50px;
	}

</style>
