<script>
	import * as d3 from "d3";

	export let colors = [];

	$: pie = d3.pie().value(d => 1)(colors);

	$: arcs = pie.map(pie =>
		d3.arc()({
			...pie,
			innerRadius: 0,
			outerRadius: 12
		})
	);
</script>

{#each arcs as arc, i}
	<path d={arc} fill={colors[i]} />
{/each}