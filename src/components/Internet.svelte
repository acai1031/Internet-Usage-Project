<script>
  import * as d3 from 'd3';

  export let data;

  const width = 928;
  const height = 600;
  const marginTop = 20;
  const marginRight = 30;
  const marginBottom = 30;
  const marginLeft = 40;

  $: console.log(data[0]);
</script>

<div class="internet-plot">
  <svg
    {width}
    {height}
    viewBox="0 0 {width} {height}"
    style="max-width: 100%; height: auto;"
  >
  <g>
  </g>
  </svg>
</div>

<!--
Sources:
https://observablehq.com/@d3/global-temperature-trends?intent=fork
https://www.nytimes.com/interactive/2017/01/18/science/earth/2016-hottest-year-on-record.html
-->
