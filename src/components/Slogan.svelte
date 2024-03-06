<script>
    import { scaleBand, scaleLinear } from 'd3-scale';
    export let index, slogans;
    let width = 500;
    let height = 220;
    const padding = {top: 20, right: 15, bottom: 30, left: 50 };

    
</script>

{#if index === 1} 
<h2>Frequently Used Campaign Slogans</h2>
<div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
    <svg>
	</svg>
</div>
{/if}

<style>

	h2 {
		text-align: center;
	}

	.chart {
		width: 100%;
		max-width: 500px;
		margin: auto;
	}

	svg {
		width: 100%;
		height: 350px;
	}

	.tick {
		font-family: "Times New Roman", Times, serif;
		font-size: 0.725em;
		font-weight: 200;
	}

	.tick line {
		stroke: #3d3d3b;
		stroke-dasharray: 2;
	}

	.tick text {
		fill: #000000;
		text-anchor: start;
	}

	.tick.tick-0 line {
		stroke-dasharray: 0;
	}

	.bars rect {
		opacity: 0.9;
	}
</style>