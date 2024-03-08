<script>
    import { onMount } from 'svelte';
    import { scaleBand, scaleLinear } from 'd3-scale';
    export let index, categories, selectedCategory, selectedPerson;
    let width = 600;
    let height = 700;

    let xTicks = [];
	let yTicks = [];
	const padding = { top: 20, right: 15, bottom: -10, left: 25 };
    let filteredData = [];
    selectedCategory = 'demographic_issue';

    onMount(() => {
        updateChart();
    });

    $: updateChart();

    function updateChart() {
        filteredData = categories.filter(
        (d) => d.topic === selectedCategory
        );
        xTicks = filteredData.map((d) => d.category);
        yTicks = filteredData.map((d) => d.difference);
    }

    $: xScale = scaleBand()
    .domain(xTicks)
    .range([padding.left, width - padding.right])
    .padding(0.1);

    $: yScale = scaleLinear()
        .domain([0, Math.max.apply(null, yTicks)])
        .range([height - padding.bottom, padding.top]);

    $: innerWidth = width - (padding.left + padding.right);
    $: barWidth = innerWidth / xTicks.length;
</script>

{#if index === 2} 
<h2>Number of Tweets in Different Topics</h2>
<div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
	<svg>
        <!-- Legend -->
        <g transform={`translate({width+40}, {padding.top - 10})`}>
            <rect x="0" y="0" width="15" height="15" fill="#ff4500" />
            <text x="20" y="12.5" font-size="12">Clinton Mentioned More</text>
            <rect x="0" y="25" width="15" height="15" fill="#00bfff" />
            <text x="20" y="37.5" font-size="12">Trump Mentioned More</text>
        </g>

		<!-- y axis -->
		<g class="axis y-axis">
			{#each yTicks as tick}
				<g class="tick tick-{tick}" transform="translate(0, {yScale(tick)})">
					<line x2="100%" />
					<text y="-4">{tick}'Count per 3,000 tweets'</text>
				</g>
			{/each}
		</g>

		<!-- x axis -->
		<g class="axis x-axis">
            {#each filteredData as point, i (i)}
                {console.log('Subtopic:', point.category)}
                <g class="tick" transform={`translate(${xScale(point.category) + xScale.bandwidth() / 2}, ${height - padding.bottom})`}>
                    <text y="4" dy="0.71em" text-anchor="middle">
                        {point.category}
                    </text>
                </g>
            {/each}
        </g>

		<g class="bars">
			{#each filteredData as point, i (i)}
				<rect
					x={xScale(point.category)}
					y={yScale(yTicks[i])}
					width={barWidth - 4}
					height={yScale(0) - yScale(yTicks[i])}
                    fill={point.hc_more === 1 ? '#ff4500' : '#00bfff'}
				/>
			{/each}
		</g>
	</svg>
</div>

<div>
  <label for="categorySelect">Select Category:</label>
  <select id="categorySelect" bind:value={selectedCategory} on:change={updateChart}>
    <option>demographic_issue</option>
    <option>social_issue</option>
    <option>security_policy</option>
    <option>emotional_word</option>
    <option>economic_policy</option>
    <option>campaign_message</option>
    <option>national_identity</option>
  </select>

  <div id="chart"></div>
</div>
{/if}

<style>

	h2 {
		text-align: center;
	}

	.chart {
		width: 100%;
		max-width: 600px;
		margin: 0 auto;
	}

	svg {
		position: relative;
		width: 100%;
		height: 400px;
	}

	.tick {
		font-family: Helvetica, Arial;
		font-size: 0.725em;
		font-weight: 200;
	}

	.tick line {
		stroke: #e2e2e2;
		stroke-dasharray: 2;
	}

	.tick text {
		fill: 'black';
		text-anchor: start;
	}

	.tick.tick-0 line {
		stroke-dasharray: 0;
	}

	.x-axis .tick text {
		text-anchor: middle;
	}

	.bars rect {
		stroke: none;
		opacity: 0.65;
	}
</style>