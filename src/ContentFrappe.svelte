<script>
  import { Select, SelectItem, MultiSelect } from "carbon-components-svelte";
  import Chart from "svelte-frappe-charts";
  export let allCountriesData = {};
  const labelData = Object.keys(
    allCountriesData[Object.keys(allCountriesData)[0]]
  ).filter(item => /^\d/.test(item));

  $: countriesData = Object.fromEntries(
    selectedIds.map(x => [x, allCountriesData[x]])
  );

  $: plotData = transformToPlotData(countriesData);

  const transformToPlotData = function(countriesData) {
    // debugger;
    return Object.keys(countriesData).map(country => ({
        values: labelData.map(day => countriesData[country][day]),
        name: "Active cases in " + country,
        chartType: "line"
      }));

  };

  const getChartConfig = (data, labels) => {
    return {
      data: {
        labels: labels,
        datasets: data
      },
      type: "line",
      height: 480,
      yRegions: [{ label: "Region", start: 0, end: 100 }],
      yMarkers: [{ label: "Marker", value: 70 }],
      axisOptions: { xIsSeries: 1 },
      isNavigable: 1
    };
  };
  let value = "";

  let items = Object.keys(allCountriesData).map(key => ({
    id: key,
    text: key
  }));
  let selectedIds = ["Poland"];
</script>

<main>
  <MultiSelect
    id="countrySelector"
    bind:selectedIds
    bind:items
    bind:value
    placeholder="Filter..."
    filterable="true" />
{#if selectedIds.length > 0}
  <Chart {...getChartConfig(plotData, labelData)} /> 
  {:else}
  <h3>Please set countries filter</h3>
  {/if}
  <i>
    Data for /{selectedIds}/:
    <br />
    <br />
    <br />
    <!-- {JSON.stringify(getChartConfig(plotData, labelData))} -->
  </i>

</main>
