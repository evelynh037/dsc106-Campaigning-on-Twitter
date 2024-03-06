<script>
  import { scaleBand, scaleLinear } from 'd3-scale';
  export let index, filtered_trump, filtered_hillary, selected_word;
  let width = 500;
  let height = 220;
  const padding = { top: 1, right: 40, bottom: 10, left: 15 };

  // Combine the two and assign with category for bars
  $: combinedData = [
    ...filtered_hillary.map(d => ({ ...d[0], category: 'Hillary' })),
    ...filtered_trump.map(d => ({ ...d[0], category: 'Trump' }))
  ];

  // Get proportion
  $: selected_word.forEach((element) => {
    let filtered = combinedData.filter((d) => d.Word == element.toLowerCase())
    let total = filtered.reduce((acc, item) => acc + item.Frequency, 0)
    filtered.forEach((element) => {
      element.proportion = (element.Frequency / total) * 100
    });
  });
  $: console.log(combinedData)

  // Create scales
  $: yScale = scaleBand()
    .domain(selected_word)
    .range([padding.top, height - padding.bottom])
    .padding(0.1);

  $: xScale = scaleLinear()
    .domain([0, 100])
    .range([padding.left, width - padding.right]);

  // Generate x-axis ticks
  function generateXTicks(maxFrequency, step) {
    const ticks = [];
    for (let i = 0; i <= maxFrequency; i += step) {
      ticks.push(i);
    }
    return ticks;
  }
  $: xTicks = generateXTicks(100, 100 / 5);
</script>

{#if index === 3} 
  <h2>Proportion of Selected Words In Each Nominee's Tweets </h2>
  <div class="chart" bind:clientWidth={width} bind:clientHeight={height}>
    <svg>
      <g class="axis x-axis" transform="translate(0, {padding.top})">
        {#each xTicks as tick}
          <g class="tick tick-{tick}" transform="translate({xScale(tick)}, 0)">
            <line y2="{height - padding.top - padding.bottom}" />
            <text x="{tick === 1 ? -1 : 1}" dy="1.2em" text-anchor="{tick === 0 ? 'end' : 'start'}">{tick}%</text>
          </g>
        {/each}
      </g>
      <g class="axis y-axis" transform="translate({padding.left}, 0)">
        {#each selected_word as word}
          <g class="tick" transform="translate(0, {yScale(word) + yScale.bandwidth() / 2})">
            <text transform="rotate(0)" x="2" y="30" dy="0.4em" text-anchor="end">{word}</text>
          </g>
        {/each}
      </g>
      <g class="bars">
        {#each combinedData as {Word, proportion, category}, i}
          <rect
            y={yScale(Word) + (category === 'Hillary' ? yScale.bandwidth() / 4 : yScale.bandwidth() / 4)}
            x={(category === 'Hillary' ? xScale(0) : xScale(100) - xScale(proportion)+padding.left)}
            height={yScale.bandwidth() / 2.5}
            width={xScale(proportion) - xScale(0)}
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
		stroke: #7e7d7d; 
    stroke-dasharray: 3;
	}

	.tick text {
		fill: #000000;
		text-anchor: start;
    font-size: 1.4em;
	}

	.tick.tick-0 line {
		stroke-dasharray: 0;
	}

	.bars rect {
		opacity: 0.9;
	}
</style>