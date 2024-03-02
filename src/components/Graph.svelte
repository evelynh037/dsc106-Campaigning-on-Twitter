<script>
  import { scaleBand, scaleLinear } from 'd3-scale';
  export let index, filtered_trump,filtered_hillary,prefix,inputText;
  let width = 500;
	let height = 220;
  const padding = {top: 20, right: 15, bottom: 30, left: 50 };

  //combined the two and assign with category for bars
  $: combinedData = [
    ...filtered_hillary.map(d => ({ ...d, category: 'Hillary' })),
    ...filtered_trump.map(d => ({ ...d, category: 'Trump' }))
  ];

  //get the larger frequency for y-axis
  $: maxFrequency = Math.max(...combinedData.map(d => d.Frequency))
  $: totalFrequency = combinedData.reduce((acc, d) => acc + d.Frequency, 0);
  $: relativeFrequency = combinedData.map(d => ({
    ...d,
    percentage: (d.Frequency / totalFrequency) * 100
  }));
  //$: console.log(relativeFrequency)

  

  function generateYTicks(maxFrequency, step) {
    const ticks = [];
    for (let i = 0; i <= maxFrequency; i += step) {
      ticks.push(i);
    }
    return ticks;
  }
  $: yTicks = generateYTicks(maxFrequency,maxFrequency / 4);

  //create scale
	$: xScale = scaleBand()
  .domain(['Hillary', 'Trump'])
  .range([padding.left, width - padding.right])
  .padding(0.1);

	$: yScale = scaleLinear()
  .domain([0, Math.max(...combinedData.map(d => d.Frequency))])
  .range([height - padding.bottom, padding.top]);
</script>

 {#if index === 3 && prefix != '' && inputText != ""} 
 <h2>Frequency of word {prefix.toUpperCase()}</h2>
 <div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
  <svg>
		<g class="axis y-axis">
			{#each yTicks as tick}
				<g class="tick tick-{tick}" transform="translate(0, {yScale(tick)})">
					<line x2="100%" />
					<text y="-3.5">{tick} {tick === maxFrequency ? ' per 3,000 tweets' : ''}</text>
				</g>
			{/each}
		</g>
		<g class="axis x-axis">
      {#each ['Hillary', 'Trump'] as category}
        <g class="tick" transform="translate({xScale(category) + (category == "Hillary"? xScale.bandwidth() /2.:35)}, {height-10})">
          <text>{category}</text>
        </g>
      {/each}
    </g>
    
    

		<g class="bars">
      {#each combinedData as {Frequency, category }, i}
        <rect
          x={xScale(category) + (category == "Hillary"? xScale.bandwidth() /4:6)}
          y={yScale(Frequency)}
          width={xScale.bandwidth() / 2}
          height={yScale(0) - yScale(Frequency)}
          fill={category === 'Hillary' ? '#0800fb' : 'red'}
        />
      {/each}
    </g>
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