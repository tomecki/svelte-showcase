<script>
  import {
    UIShell,
    Form,
    FormGroup,
    Checkbox,
    Tile,
    Button
  } from "carbon-components-svelte";
    import Papa from "papaparse";

  // import ContentFusion from './ContentFusion.svelte';
  // import ContentFrappe from './ContentFrappe.svelte';
  import ContentClear from './ContentClear.svelte';

  const fetchImage = (async () => {
    const response = await fetch("/time_series_covid_19_confirmed.csv");
	return  await response.text();
  })();

  
  const csvToJS = function(data) {
    let allCountriesData = {};
    Papa.parse(data, {
      header: true,
      complete: results => {
        allCountriesData = results.data.reduce((accumulator, current) => {
          accumulator[current["Country/Region"]] = current;
          return accumulator;
        }, {});
      }
    })
    return allCountriesData;
  }

</script>

<svelte:head>
  <link
    rel="stylesheet"
    href="https://unpkg.com/carbon-components@10.9.0/css/carbon-components.min.css" />
</svelte:head>

<main>
  <UIShell
    href="https://appsilon.com"
    company="Svelte"
    platformName="Visualizer" />
  <br />
  <br />
  <br />
  <Tile>
    {#await fetchImage}
      <p>...waiting</p>
    {:then data}
		
		<!-- <ContentFusion data={data} /> -->
		<!-- <ContentFrappe allCountriesData={csvToJS(data)} /> -->
    <ContentClear />
    {/await}
  </Tile>

</main>
