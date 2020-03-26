<script>
  import Papa from "papaparse";
  import { Select, SelectItem } from "carbon-components-svelte";
    import FusionCharts from "fusioncharts";
    import Timeseries from "fusioncharts/fusioncharts.timeseries";
    import SvelteFC, { fcRoot } from "svelte-fusioncharts";

  export let data = "";

  let allCountriesData = {};
  $: Papa.parse(data, {
    header: true,
    complete: results => {
      allCountriesData = results.data.reduce((accumulator, current) => {
        accumulator[current["Country/Region"]] = current;
        return accumulator;
      }, {});
      console.log(allCountriesData["Poland"]);
    }
  });

  let selected = "Poland";
  const allowedCountries = ["China", "Poland", "United Kingdom", "Korea"];
  $: countryData = allCountriesData[selected];
  $: plotData =
    countryData !== undefined
      ? Object.keys(countryData)
          .filter(item => /^\d/.test(item))
          .map(item => [item, countryData[item]])
      : [];
  const schema = [
    {
      name: "date",
      type: "date",
      format: "%m/%d/%y"
    },
    {
      name: "active cases",
      type: "number"
    }
  ];
  const getChartConfig = data => {
    const fusionDataStore = new FusionCharts.DataStore(),
      fusionTable = fusionDataStore.createDataTable(data, schema);
    return {
    type: 'timeseries',
      width: '100%',
      height: 450,
      renderAt: 'chart-container',
      dataSource: {
        data: fusionTable,
        caption: {
          text: 'Active cases'
        },
        subcaption: {
          text: 'in ' + selected
        },
        yAxis: [
          {
            plot: {
              value: 'active cases',
              type: 'line'
            },
            title: 'Active cases'
          }
        ]
      }
    };
  };

    fcRoot(FusionCharts, Timeseries);
</script>

<main>
  <Select bind:selected>
    {#each Object.keys(allCountriesData) as selectCountry}
      <SelectItem value={selectCountry} text={selectCountry} />
    {/each}
  </Select>
  <!-- <i>Data for /{selected}/: {plotData}</i> -->
  <SvelteFC {...getChartConfig(plotData)} />

</main>
