<script>
  import { hierarchy, interpolateHcl, interpolateZoom, pack, scaleLinear, scaleOrdinal, transition } from 'd3';
  export let index;
import data from './SloganData';
  
  const width = 600; //the outer width of the chart, in pixels
  const height = width; // the outer height of the chart, in pixels
  const margin = 20; //the overall margin between the circle packs to the viewport edge
  const backgroundColor = 'transparent'; // the background color of the chart
  const fontSize = 15; //the font size of the text labels
  const padding = { top: 70, right: 50, bottom: 30, left: 50 };

  const color = scaleLinear()
    .domain([0, 5])
    .range(['hsl(152,80%,80%)', 'hsl(228,30%,40%)'])
    .interpolate(interpolateHcl);

const candidate_color = scaleOrdinal()
  .domain(['Trump', 'Hillary'])
  .range(['#ff4500', '#00bfff']);


  const packFunc = pData => pack()
    .size([width - margin, height - margin])
    .padding(3)
    (hierarchy(pData)
      .sum(d => d.value)
      .sort((a, b) => b.value - a.value));

  const root = packFunc(data);
  let activeFocus = root;
  let view;
  let activeZoomK = width / root.r * 2;
  let activeZoomA = root.x;
  let activeZoomB = root.y;
let selectedCircle;
let descriptionBox;
let tooltip;

  const inactiveZoomTo = (v) => {
    activeZoomK = width / v[2];
    view = v;
    activeZoomA = v[0];
    activeZoomB = v[1];
  };

  inactiveZoomTo([root.x, root.y, root.r * 2 + margin]);

  const zoom = (d, e) => {
    e.stopPropagation();
    const activeFocus0 = activeFocus;

    activeFocus = d;

    transition()
      .duration(750)
      .tween('zoom', () => {
        var i = interpolateZoom(view, [activeFocus.x, activeFocus.y, activeFocus.r * 2 + margin]);
        return function(t) { 
          inactiveZoomTo(i(t)); 
        };
      });
  };

const defaultData = {
  name: "All Slogans Overview",
  description: "Slogans from both candidates are categorized based on themes. Click on each theme to explore how Trump and Clinton differ in their choice of slogan and think about how those difference might affect their voting results."
};

const showDescription = (d) => {
  selectedCircle = d;
  const descriptionBox = document.getElementById('description-box');
  const { data } = selectedCircle;
  console.log(data);
  descriptionBox.innerHTML = `
  <h3>${data.name}</h3>
  <p>Description: ${data.description}</p>
  `;
  descriptionBox.style.left = `700px`;
    descriptionBox.style.top = `20px`;
  descriptionBox.style.display = 'block';

};

const showTooltip = (d) => {
    console.log("has root");
    tooltip.innerHTML = `
    <h3>${d.name}</h3>
    <p>Score: ${d.score}</p>
    <p>Candidate: ${d.candidate}</p>
    <p>Description: ${d.description}</p>
    `;
    tooltip.style.display = 'block';
};

const hideTooltip = () => {
  if (tooltip) {
    tooltip.style.display = 'none';
  }
};
</script>

{#if index === 2} 
<h2>Slogans in Tweets Analysis</h2>
<svg class="chart" width={width} height={height} style="background: {backgroundColor};" on:click={(e) => {zoom(root, e); showDescription(root)}}   >
  <g class="legend" transform="translate({width-80}, {padding.top - 60})">
    <rect x="0" y="0" width="15" height="15" fill="#ff4500" />
    <text x="30" y="12" font-size="0.9em">Trump</text>
    <rect x="0" y="20" width="15" height="15" fill="#00bfff" />
    <text x="30" y="32" font-size="0.9em">Hillary</text>
  </g>
  <g transform="translate({width / 2},{height / 2})">
    {#each root.descendants().slice(1) as rootData}
    <circle class={rootData.parent ? rootData.children ? 'node' : 'node node--leaf' : 'node node--root'}
      fill={rootData.children ? "white" : (rootData.data.candidate === 'Trump' ? '#ff4500' : '#00bfff')}
      stroke='grey'
      on:click={(e) => {
        if (activeFocus !== rootData) {
          zoom(rootData, e); 
          showDescription(rootData);
        }
      }}
      on:mouseover={() => {
        if (activeFocus.children) {
          if (!rootData.children) {
            showTooltip(rootData)
          }
        }}}
        on:mouseout={() => hideTooltip()}
      transform="translate({(rootData.x - activeZoomA) * activeZoomK},{(rootData.y - activeZoomB) * activeZoomK})"
      r={rootData.r * activeZoomK}
    ></circle>
    {/each}


    {#each root.descendants() as rootDes}
      <text font-size='{fontSize}px' class="label" 
        style="fill-opacity: {rootDes.parent === activeFocus ? 1 : 0}; display: {rootDes.parent === activeFocus ? "inline" : "none"};"
        transform="translate({(rootDes.x - activeZoomA) * activeZoomK},{(rootDes.y - activeZoomB) * activeZoomK})"
      >{rootDes.data.name}</text>
    {/each}
  </g>
  <div bind:this={tooltip} class="tooltip"></div>
</svg>
<div class="description-container">
<div class="description-box" bind:this={descriptionBox} id="description-box">
  <!-- Content for the description box goes here -->
  <h3>{defaultData.name}</h3>
  <p>Description: {defaultData.description}</p>
</div>
</div>
{/if}

<style>
h2 {
  text-align: center;
}

.chart {
  width: 100%;
  margin: 20 auto;
}

.description-container {
position: absolute;
left: 700px; /* Adjust the left position as needed */
top: 80px; /* Adjust the top position as needed */
width: 200px; /* Set the width as needed */
height: auto; /* Set the height as needed or use a fixed height */
}

/* Style for the description box inside the container */
.description-box {
background-color: #fff; /* Set the background color as needed */
padding: 10px; /* Adjust padding as needed */
border: 1px solid #ccc; /* Add border styling as needed */
}

.tooltip {
background-color: grey;
padding: 10px;
border: 1px solid #ccc;
display: none;
}

.node {
  cursor: pointer;
}

.node:hover {
  stroke: #000;
  stroke-width: 1.5px;
}

.label {
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  text-anchor: middle;
  text-shadow: 0 1px 0 #fff, 1px 0 0 #fff, -1px 0 0 #fff, 0 -1px 0 #fff;
}

.label,
.node--root,
.node--leaf {
  pointer-events: none;
}
</style>