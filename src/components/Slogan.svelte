<script>
  import { hierarchy, interpolateHcl, interpolateZoom, pack, scaleLinear, scaleOrdinal, transition } from 'd3';
  export let index;
	import data from './SloganData';
	import * as d3 from 'd3';
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
	let tooltip = {innerHTML:"", visibility: 'hidden', x: 0, y: 0  };
	let xCoordinate = 0;
	let yCoordinate = 0;

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

	const tooltipW = 180;
	const tooltipH = 100;
	const tooltipPaddingTop = 30;
	const tooltipPaddingLeft = 15;
	const tooltipLineHeight = 36;
	let mousePosition = [0, 0];
	function recordMousePosition(event) {
		mousePosition = d3.pointer(event);
	}

	function showTooltip(circle, event) {
		const tooltipBox = document.getElementById('tooltip-box');
		const tooltipText = tooltipBox.querySelector('.tooltip-text');

		tooltipText.innerHTML = `
			<tspan x="0" dy="1.2em">Name: ${circle.data.name}</tspan>
			<tspan x="0" dy="1.2em">Tweet Counts: ${circle.data.value}</tspan>
			<tspan x="0" dy="1.2em">Sentiment Score: ${circle.data.score}</tspan>
			<tspan x="0" dy="1.2em">Candidate: ${circle.data.candidate}</tspan>
			<tspan x="0" dy="1.2em">${circle.data.description}</tspan>
		`;

		tooltipBox.style.left = `${mousePosition[0]}px`;
		tooltipBox.style.top = `${mousePosition[1]}px`;
		tooltipBox.style.width = tooltipW;
		tooltipBox.style.visibility = 'visible';
	}

	function hideTooltip() {
		const tooltipBox = document.getElementById('tooltip-box');
		tooltipBox.style.visibility = 'hidden';
	};
</script>

{#if index === 3 || index === 4} 
	<h2>Slogans in Tweets Analysis</h2>
	<svg class="chart" width={width} height={height} style="background: {backgroundColor};" on:click={(e) => {zoom(root, e); showDescription(root)}} on:pointermove={recordMousePosition}   >
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
				on:mouseover={(event) => {
					if (!rootData.children && activeFocus.depth >=2) {
						showTooltip(rootData, event);
					}
				}}
				on:mouseout={(event) => hideTooltip()}
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
		<g class="tooltip-box" id="tooltip-box" style="visibility: hidden;" transform="translate({mousePosition[0] - tooltipW - 5},{mousePosition[1]})">
			<rect width={tooltipW+130} height={tooltipH} fill="#D3D3D3" stroke="grey"></rect>
			<text class="tooltip-text" x={xCoordinate + 10} y={yCoordinate} font-size="14px"></text>
		</g>
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

	.description-box {
		background-image: url('https://raw.githubusercontent.com/evelynh037/dsc106-final-graph/main/static/text.jpg'); 
		background-repeat: no-repeat;
		background-size: contain; 
		background-position: center center;
		width: 200px;
		height: 242px; 
		padding:10px;
		font-size: 16px;

	}

	.tooltip {
        font-size: 14px;
		fill: rgb(2, 100, 12);
		font-weight: bold;
    }

	.tooltip-text {
		fill: #000;
		font-weight: bold;
		dominant-baseline: hanging; /* Align the text to the top */
	}

	.tooltip-box{
		position: relative;
  		z-index: 1;
		/* Prevents the tooltip from interfering with mouse events */
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
		
	}
</style>