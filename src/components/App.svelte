<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Internet from './Internet.svelte';

  let data = [];

  onMount(async () => {
    const res = await fetch('../static/Internet_usage_data.csv'); 
    const csv = await res.text();
    data = d3.csvParse(csv, d3.autoType);
    console.log(data.length);
  });
</script>

<main>
  <h1>Internet Usage Project</h1>
  <Internet {data} />
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Nunito:wght@300;400;700&display=swap');

  :root {
    --color-bg: #ffffff;
    --color-outline: #c2c2c2;
    --color-shadow: hsl(0, 0%, 0%, 0.1);
    --color-text: #3f4252;
    --color-bg-1: hsla(0, 0%, 0%, 0.2);
    --color-shadow-1: hsl(0, 0%, 96%);
  }

  *,
  *::before,
  *::after {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  main {
    text-align: center;
    font-family: 'Nunito', sans-serif;
    font-weight: 300;
    line-height: 2;
    font-size: 24px;
    color: var(--color-text);
  }

  h1 {
    font-size: 2em;
    font-weight: 300;
    line-height: 2;
  }
</style>
