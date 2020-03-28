<script>
  import {
    UIShell,
    Tile
  } from "carbon-components-svelte";

  import ContentClear from './ContentClear.svelte';

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
    <ContentClear />
  </Tile>

</main>
