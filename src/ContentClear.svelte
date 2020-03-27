<script>
  import * as d3 from "d3";
  import _ from "lodash";
  import DaChart from "./DaChart.svelte";
	let x1 = +Infinity;
	let x2 = -Infinity
	let y1 = +Infinity;
    let y2 = -Infinity;
    let closest;
	let filter = '';
  let data = {};
  let startAt = 0;
  const fixedKeys = ["Country/Region", "Province/State", "Lat", "Long"];
  const fetchImage = (async () => {
    return await d3
      .csv("/time_series_covid_19_confirmed.csv")
      .then(function(data) {
        return preprocessCsv(data);
      });
  })();
  const preprocessCsv = function(data) {
    const ticks = Object.keys(data[0]).filter(x => !fixedKeys.includes(x));
    const ticksMapping = Object.fromEntries(_.map(ticks, (v, i) => [v, i]));

    let grouped = _.groupBy(data, x => x["Country/Region"]);
    let summarizedOverCities = {};
    Object.keys(grouped).forEach(key => {
      const summed = 
        ticks.map(tick => ({
          "x": ticksMapping[tick],
          "y": _.sum(_.map(grouped[key], prov => +prov[tick]))
        }));
      summarizedOverCities[key] = summed;
    });
    console.log(summarizedOverCities);
    return summarizedOverCities;
  };
 
</script>

<main>
  <h1>Example</h1>
  {#await fetchImage}
    <p>...waiting</p>
  {:then data}
    <DaChart data={data} startAt={startAt}/>
{/await}
</main>
