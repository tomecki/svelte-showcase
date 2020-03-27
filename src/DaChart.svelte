<script>
  export let data = [];
  import * as Pancake from "@sveltejs/pancake";
  import { NumberInput, MultiSelect, Toggle } from "carbon-components-svelte";
  let closest;
  let country;
  let startAt = 100;
  let value = 0;
  let selectedIds = [];
  const preparePlotData = function(data, startAt, selectedIds, logarithmic) {
    console.log(logarithmic);
    let filteredBelowStartAt = Object.keys(data)
      .filter(country =>
        selectedIds.length > 0 ? selectedIds.includes(country) : true
      )
      .map(x => {
        let filtered = data[x].filter(point => point.y >= startAt);
        return {
          name: x,
          data:
            filtered.length > 0
              ? filtered.map(point => ({
                  x: point.x - filtered[0].x,
                  y: logarithmic ? Math.log10(point.y) : point.y
                }))
              : []
        };
      });

    return filteredBelowStartAt;
  };
  const getPlotBounds = function(plotData) {
    let minx = +Infinity;
    let maxx = -Infinity;
    let miny = +Infinity;
    let maxy = -Infinity;
    plotData.forEach(country => {
      country.data.forEach(point => {
        minx = minx < point.x ? minx : point.x;
        maxx = maxx > point.x ? maxx : point.x;
        miny = miny < point.y ? miny : point.y;
        maxy = maxy > point.y ? maxy : point.y;
      });
    });
    return {
      minx: minx,
      maxx: maxx,
      miny: miny,
      maxy: maxy
    };
  };

  let toggled = false;
  $: plotData = preparePlotData(data, value, selectedIds, toggled);
  $: plotBounds = getPlotBounds(plotData);
  let items = Object.keys(data).map(data => ({ id: data, text: data })); //Object.keys(data).map(data => ({ id: data.name, text: data.name }));
  //   $: points = filtered.reduce((plotData, country) => {
  //     return points.concat(
  //       country.data.map(d => ({
  //         x: d.x,
  //         y: d.y,
  //         country
  //       }))
  //     );
  //   }, []);
</script>

<style>
  .chart {
    height: 400px;
    padding: 3em 0 2em 2em;
    margin: 0 0 36px 0;
  }

  input {
    font-size: inherit;
    font-family: inherit;
    padding: 0.5em;
  }

  .grid-line {
    position: relative;
    display: block;
  }

  .grid-line.horizontal {
    width: calc(100% + 2em);
    left: -2em;
    border-bottom: 1px dashed #ccc;
  }

  .grid-line span {
    position: absolute;
    left: 0;
    bottom: 2px;
    font-family: sans-serif;
    font-size: 14px;
    color: #999;
  }

  .x-label {
    position: absolute;
    width: 4em;
    left: -2em;
    bottom: -22px;
    font-family: sans-serif;
    font-size: 14px;
    color: #999;
    text-align: center;
  }

  path.data {
    stroke: rgba(0, 0, 0, 0.2);
    stroke-linejoin: round;
    stroke-linecap: round;
    stroke-width: 1px;
    fill: none;
  }

  .highlight {
    stroke: #ff3e00;
    fill: none;
    stroke-width: 2;
  }

  .annotation {
    position: absolute;
    white-space: nowrap;
    bottom: 1em;
    line-height: 1.2;
    background-color: rgba(255, 255, 255, 0.9);
    padding: 0.2em 0.4em;
    border-radius: 2px;
  }

  .annotation-point {
    position: absolute;
    width: 10px;
    height: 10px;
    background-color: #ff3e00;
    border-radius: 50%;
    transform: translate(-50%, -50%);
  }

  .annotation strong {
    display: block;
    font-size: 20px;
  }

  .annotation span {
    display: block;
    font-size: 14px;
  }
</style>
<MultiSelect
  id="multiselect"
  placeholder="Filter countries..."
  bind:selectedIds
  filterable = {true}
  bind:items />
<NumberInput min="0" max="1000" bind:value />
<Toggle bind:toggled />
<!-- {JSON.stringify(plotData)} -->
<div class="chart">
  <Pancake.Chart
    x1={plotBounds.minx}
    x2={plotBounds.maxx}
    y1={plotBounds.miny}
    y2={plotBounds.maxy}>
    <Pancake.Grid horizontal count={5} let:value>
      <div class="grid-line horizontal">
        <span>{value}</span>
      </div>
    </Pancake.Grid>

    <Pancake.Grid vertical count={5} let:value>
      <span class="x-label">{value}</span>
    </Pancake.Grid>

    <Pancake.Svg>
      {#each plotData as country}
        <Pancake.SvgLine data={country.data} let:d>
          <path class="data" {d} />
        </Pancake.SvgLine>
      {/each}

      <!--		{#if closest}
				<Pancake.SvgLine data={closest.country.data} let:d>
					<path class="highlight" {d}></path>
				</Pancake.SvgLine>
			{/if} -->
    </Pancake.Svg>

    <!-- {#if closest}
			<Pancake.Point x={closest.x} y={closest.y}>
				<span class="annotation-point"></span>
				<div class="annotation" style="transform: translate(-{100 * ((closest.x - x1) / (x2 - x1))}%,0)">
					<strong>{closest.country.name}</strong>
					<span>{closest.x}: {closest.y} years</span>
				</div>
			</Pancake.Point>
		{/if} -->

    <!-- <Pancake.Quadtree data={points} bind:closest /> -->
  </Pancake.Chart>
</div>
