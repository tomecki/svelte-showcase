<script>
  export let data = [];
  import * as Pancake from "@sveltejs/pancake";
  import {
    NumberInput,
    MultiSelect,
    RadioButtonGroup,
    RadioButton,
    Tag,
    Form,
    FormGroup,
    FormLabel,
    Button
  } from "carbon-components-svelte";
  import { writable } from "svelte/store";
  let closest;
  let country;
  let value = 0;
  let y_axis = writable("linear");
  let startAt = writable(0);
  const DEFAULT_COUNTRIES = ["Poland", "Germany", "Korea, South", "US"];

  let selectedIds = DEFAULT_COUNTRIES;
  const preparePlotData = function(data, startAt, selectedIds, y_axis) {
    let filteredBelowStartAt = Object.keys(data)
      .filter(country =>
        selectedIds.length > 0 ? selectedIds.includes(country) : true
      )
      .map(x => {
        let filtered = data[x].filter(point => point.raw >= startAt);
        return {
          name: x,
          data:
            filtered.length > 0
              ? filtered.map(point => ({
                  x: point.x - filtered[0].x,
                  y: $y_axis === "logarithmic" ? Math.log10(point.raw) : point.raw,
                  raw: point.raw
                }))
              : []
        };
      });

    return filteredBelowStartAt;
  };
  $: if ($y_axis === "logarithmic") {
    $startAt = Math.max(1, $startAt);
  }
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
  const teststore = writable("xd");
  $: plotData = preparePlotData(data, $startAt, selectedIds, $y_axis);
  $: plotBounds = getPlotBounds(plotData);
  let items = Object.keys(data).map(data => ({ id: data, text: data }));
  $: points = plotData.reduce((points, country) => {
    return points.concat(
      country.data.map(d => ({
        x: d.x,
        y: d.y,
        raw: d.raw,
        country
      }))
    );
  }, []);
</script>

<style>
  #multiselect {
    grid-column-start: 1;
    grid-column-end: 3;
    grid-row: 1;
  }

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
  #formGrid {
    display: grid;
    grid-row-gap: 1em;
    grid-column-gap: 1em;
  }
  #numberInput {
    grid-row: 2;
    grid-column: 1;
  }
  #yaxisInput {
    grid-row: 2;
    grid-column: 3;
  }
</style>

<Form>
  <div id="formGrid">
    <div id="multiselect">
      <MultiSelect
        placeholder="Filter countries..."
        helperText="Countries"
        bind:selectedIds
        filterable={true}
        bind:items />
      <Button on:click={() => (selectedIds = DEFAULT_COUNTRIES)}>
        Default countries
      </Button>

    </div>
    <div id="defaultButton" />
    <div id="numberInput">
      <NumberInput
        label="Discard until cases above"
        labelmin="0"
        max="1000"
        bind:value={$startAt} />
    </div>
    <div id="yaxisInput">
      <FormGroup legendText="Y Axis scale">
        <RadioButtonGroup  bind:selected={$y_axis}>
          <RadioButton value="linear" id="linear" labelText="Linear" />
          <RadioButton
            value="logarithmic"
            id="logarithmic"
            labelText="Logarithmic" />
        </RadioButtonGroup>
      </FormGroup>
    </div>
  </div>
</Form>
<div>
  {#each selectedIds as country}
    <Tag type="blue">{country}</Tag>
  {/each}
</div>
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

      {#if closest}
        <Pancake.SvgLine data={closest.country.data} let:d>
          <path class="highlight" {d} />
        </Pancake.SvgLine>
      {/if}
    </Pancake.Svg>

    {#if closest}
			<Pancake.Point x={closest.x} y={closest.y}>
				<span class="annotation-point"></span>
				<div class="annotation" style="transform: translate(-{100 * ((closest.x - plotBounds.minx) / (plotBounds.maxx - plotBounds.minx))}%,0)">
					<strong>{closest.country.name}</strong>
					<span>Day {closest.x}: {closest.raw} Cases</span>
				</div>
			</Pancake.Point>
		{/if}

    <Pancake.Quadtree data={points} bind:closest />
  </Pancake.Chart>
</div>
