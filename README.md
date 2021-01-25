A single-page app for creating Bookworm visualizations.

This uses the Bookworm-Vega repo to generate charts and handle server negotations,
and svelte (and svelte-material-ui) to generate queries.

It's currently under development; right now, it only supports line charts. But the components
for generating corpora and queries can pretty easily extend to heatmaps, dot charts, maps,
bar charts, and anything else in the bookworm-vega repo.
If anyone actually wants to use it, note that you'll have to change the location of the
bookworm-vega repo in package.json.

For an example, see the [subtitle bookworm](https://movies.benschmidt.org).

