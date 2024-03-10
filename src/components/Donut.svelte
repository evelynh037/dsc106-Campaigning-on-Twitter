<script>
  import * as d3 from "d3";
  export let index;
  import { tweened } from 'svelte/motion';

  let data = [
    { name: "Hillary Clinton", electoralVotes: 227, popularVotePercentage: 48.2 },
    { name: "Donald Trump", electoralVotes: 304, popularVotePercentage: 46.1 },
    { name: "Others", electoralVotes: 7, popularVotePercentage: 5.7 }
  ];

  const width = 200;
  const height = 200;
  const radius = Math.min(width, height) / 2;
  const padding = 20; 

  const colourScale = d3.scaleOrdinal()
    .domain(data.map(d => d.name))
    .range(["#00bfff", "#ff4500", "#aaaaaa"]); 

  const pieElectoral = d3.pie()
    .sort(null)
    .value(d => d.electoralVotes);

  const arcPathElectoral = d3.arc()
    .innerRadius(0)
    .outerRadius(radius);

  const piePopular = d3.pie()
    .sort(null)
    .value(d => d.popularVotePercentage);

  const arcPathPopular = d3.arc()
    .innerRadius(0)
    .outerRadius(radius);

  const arcsElectoral = pieElectoral(data);
  const arcsPopular = piePopular(data);

  let progress = tweened(0, { duration: 1000 });

  // Function to adjust the radius for highlighted arcs
  function getRadius(sliceName, isElectoral) {
    if ((isElectoral && sliceName === "Donald Trump") || (!isElectoral && sliceName === "Hillary Clinton")) {
      return radius * 1.1; // Increase the radius by 10% for highlighted slices
    }
    return radius;
  }
</script>

{#if index === 2} 
<aside>
  <p><strong>2016 U.S. Presidential Election Outcome</strong></p>
  <p6>Left: Electoral Votes | Right: Popular Vote</p6>
</aside>
<div class="chart">
<svg {width} {height} viewBox={`-${width } -${height / 2} ${width * 2.5 + padding * 2} ${height}`}>
  <g class="electoral-votes" transform={`translate(${-width / 3 - padding / 2}, 0)`}>
    {#each arcsElectoral as slice}
      <path d={d3.arc().innerRadius(0).outerRadius(getRadius(slice.data.name, true))(slice)} fill={colourScale(slice.data.name)} stroke="white" />
      <text
        transform={`translate(${arcPathElectoral.centroid(slice)})`}
        text-anchor="middle"
        style="font-size: 12px; font-weight: bold;">
        {slice.data.name}
      </text>
      {#if (slice.endAngle - slice.startAngle) > 0.25}
        <text
          transform={`translate(${arcPathElectoral.centroid(slice)[0]}, ${arcPathElectoral.centroid(slice)[1] + 15})`}
          text-anchor="middle"
          style="font-size: 10px;">
          {slice.data.electoralVotes.toLocaleString("en-US")}
        </text>
      {/if}
    {/each}
  </g>
  <g class="popular-vote" transform={`translate(${width / 2 + padding * 5 }, 0)`}>
    {#each arcsPopular as slice}
      <path d={d3.arc().innerRadius(0).outerRadius(getRadius(slice.data.name, false))(slice)} fill={colourScale(slice.data.name)} stroke="white" />
      <text
        transform={`translate(${arcPathPopular.centroid(slice)})`}
        text-anchor="middle"
        style="font-size: 12px; font-weight: bold;">
        {slice.data.name}
      </text>
      {#if (slice.endAngle - slice.startAngle) > 0.25}
        <text
          transform={`translate(${arcPathPopular.centroid(slice)[0]}, ${arcPathPopular.centroid(slice)[1] + 15})`}
          text-anchor="middle"
          style="font-size: 10px;">
          {slice.data.popularVotePercentage.toLocaleString("en-US") + '%'}
        </text>
      {/if}
    {/each}
  </g>
</svg>
</div>
{/if}

<style>
  p {
  text-align: center;
  font-size: 1.2em;
}
p6 {
  text-align: center;
  font-size: 0.8em;
}
  aside {
    text-align: center;
    font-family:"Times New Roman", Times, serif;
    font-size: 23px; 
  }
  .chart {
    width: 100%;
    max-width: 900px;
    margin: auto;
  }
  svg {
    width: 100%;
    height: 500px;
  }
</style>
  
  