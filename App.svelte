<script>
	import * as d3 from "d3";
	import { onMount } from "svelte";
	import AxisHorizontal from "./AxisHorizontal.svelte";
	import AxisVertical from "./AxisVertical.svelte";
	import Pie from "./Pie.svelte";
	import Tooltip from "./Tooltip.svelte";

	// access data
	let data = [];
	d3.csv(
		"https://raw.githubusercontent.com/jwilber/Bob_Ross_Paintings/master/data/bob_ross_paintings.csv"
	).then(res => {
		data = res.slice(0, 60);
	});

	const xAccessor = d => +d.painting_index;
	const yAccessor = d => +d.num_colors;
	const colorAccessor = d =>
		d.color_hex.split("'").filter(d => d.startsWith("#"));

	// create chart dimensions
	let margin = {
		left: 30,
		top: 0,
		right: 0,
		bottom: 30
	};
	let width = 400;
	let height = 300;
	$: boundsWidth = width - margin.left - margin.right;
	$: boundsHeight = height - margin.top - margin.bottom;

	let xRange = [0, 0];
	$: boundsWidth, (xRange = [0, boundsWidth]);

	// create scales
	$: xScale = d3
		.scaleLinear()
		.domain(d3.extent(data, xAccessor))
		.range(xRange);

	$: yScale = d3
		.scaleLinear()
		.domain([0, d3.max(data, yAccessor)])
		.range([boundsHeight, 0]);

	// set up interactions
	let hoveredPoint = null;
	$: quadtree = d3
		.quadtree()
		.x(d => xScale(xAccessor(d)))
		.y(d => yScale(yAccessor(d)))
		.addAll(data);
</script>


<h2>Color variety in Bob Ross paintings</h2>
<figure>
	<div class="wrapper" bind:clientWidth={width} bind:clientHeight={height}>
		<svg
			width={width}
			height={height}>

			<g transform="translate({margin.left}, {margin.top})">

				{#each data as d,i (d.painting_index)}
					<g transform="translate({xScale(xAccessor(d))}, {yScale(yAccessor(d))})">
						<Pie
							colors={colorAccessor(d)}
						/>
						<circle r="13" fill="none" stroke="black" />
					</g>
				{/each}

				<rect
					width={boundsWidth}
					height={boundsHeight}
					fill="transparent"
					on:mousemove={e => {
						const pos = d3.pointer(e)
						const x = pos[0]
						const y = pos[1]
						const closestPoint = quadtree.find(x, y)
						if (!closestPoint) return
						const hoveredPointPosition = [
							xScale(xAccessor(closestPoint)),
							yScale(yAccessor(closestPoint))
						]
						// don't highlight if too far away
						// a^2 + b^2 = c^2
						const distance = Math.sqrt(
							(x - hoveredPointPosition[0]) ** 2
							+ (y - hoveredPointPosition[1]) ** 2
						)
						if (distance < 50) {
							hoveredPoint = closestPoint
						} else {
							hoveredPoint = null
						}
					}}
					on:mouseleave={() => {
						hoveredPoint = null
					}}
				/>

			</g>
			<g transform="translate({margin.left}, {boundsHeight + margin.top})">
				<AxisHorizontal
					scale={xScale}
					count="5"
				/>
			</g>
			<g transform="translate({margin.left}, 0)">
				<AxisVertical
					count="5"
					scale={yScale}
				/>
			</g>
		</svg>

		<div
			class="label"
			style="transform: translate({margin.left + 6}px, -10px)">
			Number of colors
		</div>

		<div
			class="x-label">
			Painting number
		</div>

		{#if hoveredPoint}
			<Tooltip
				x={margin.left + xScale(xAccessor(hoveredPoint))}
				y={
					margin.top
					+ yScale(yAccessor(hoveredPoint))
					- 15 * (yScale(yAccessor(hoveredPoint)) < 100 ? -1 : 1)
				}
				placement={[
					xScale(xAccessor(hoveredPoint)) > boundsWidth - 50 ? -90 : -50,
					yScale(yAccessor(hoveredPoint)) < 100 ? 0 : -100,
				]}
			>
				<strong>Painting #{xAccessor(hoveredPoint)}</strong>
				<div>
					{yAccessor(hoveredPoint)} colors
				</div>
				<img src={hoveredPoint.img_src} alt="Painting #{xAccessor(hoveredPoint)}" />
			</Tooltip>
		{/if}
	</div>
</figure>

<style>
	.wrapper {
		position: relative;
		margin: 0;
		font-family: sans-serif;
		width: 100%;
		height: 300px;
		min-width: 0;
	}

	svg {
		overflow: visible;
	}

	h2 {
		text-align: center;
		font-size: 1.2em;
		font-family: sans-serif;
	}

	.label {
		position: absolute;
		top: 0;
		left: 0;
		font-size: 0.6em;
	}

	.x-label {
		position: absolute;
		bottom: 0;
		left: 50%;
		font-size: 0.6em;
		transform: translate(-50%, 1.5em);
	}

	img {
		max-width: 120px;
		margin-top: 5px;
	}
</style>