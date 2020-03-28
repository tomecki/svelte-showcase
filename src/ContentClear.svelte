<script>
  import * as d3 from "d3";
  import _ from "lodash";

  import { Loading } from "carbon-components-svelte";
  import DaChart from "./DaChart.svelte";
  let data = {};
  const fixedKeys = ["Country/Region", "Province/State", "Lat", "Long"];
  const fetchImage = (async () => {
    return await d3
      .csv("https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv")
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
      const summed = ticks.map(tick => ({
        x: ticksMapping[tick],
        raw: _.sum(_.map(grouped[key], prov => +prov[tick]))
      }));
      summarizedOverCities[key] = summed;
    });
    console.log(summarizedOverCities);
    return summarizedOverCities;
  };
</script>

<main>
  <h1>Covid Dashboard</h1>
  {#await fetchImage}
    <Loading />
  {:then data}
    <div class="grid-container">
      <DaChart {data} />
    </div>
  {/await}
</main>

<style>
.grid-container {
  display: grid;
}
</style>
