<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { feature } from 'topojson-client';
  import { geoWinkel3 } from 'd3-geo-projection';

  let data = [];
  let svg;
  let selectedYear = 2000;
  let allowedYears = [2000, 2005, 2010, 2015, 2019, 2020, 2021];
  let filteredData = [];
  let tooltipContent = ''; 
  let tooltipStyle = 'visibility: hidden; position: absolute;'; 
  let yearIndex = allowedYears.indexOf(selectedYear); 

  onMount(async () => {
    const res = await fetch('Internet_usage_data.csv');
    const csv = await res.text();
    data = d3.csvParse(csv, d3.autoType);

    svg = d3.select('.chart-svg');
    filteredData = data.filter(d => d.Year === selectedYear);
    drawChart();
  });

  $: yearIndex = allowedYears.indexOf(selectedYear);
  $: if (data.length > 0 && selectedYear) {
    filteredData = data.filter(d => d.Year === selectedYear);

    // Define the zoom behavior
    const zoom = d3.zoom()
      .scaleExtent([1, 8]) // Set minimum and maximum scale
      .on('zoom', ({transform}) => {
        svg.selectAll('path') // Zoom and pan the map paths
           .attr('transform', transform);
        // Optionally, adjust other elements like tooltips or points if necessary
      });

    svg.call(zoom); // Apply the zoom behavior to the SVG element

    drawChart();
  }

  async function drawChart() {
    if (data.length === 0 || !svg.node()) return;
    const margin = { top: 20, right: 20, bottom: 20, left: 20 },
          width = 960 - margin.left - margin.right,
          height = 500 - margin.top - margin.bottom;

    const worldMapResponse = await fetch('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json');
    const worldMap = await worldMapResponse.json();
    const countries = feature(worldMap, worldMap.objects.countries).features;

    const colorScale = d3.scaleSequential(d3.interpolateYlGnBu)
                         .domain([0, d3.max(filteredData, d => d.Value)]);

    svg.selectAll("*").remove();

    const projection = d3.geoEqualEarth() // or d3.geoEqualEarth()
      .scale(200) // Start with a larger scale
      .translate([width / 2, height / 1.5]); // Adjust translate values as needed
    const path = d3.geoPath().projection(projection);

    svg.attr('viewBox', `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`)
       .selectAll('path')
       .data(countries)
       .enter().append('path')
       .attr('d', path)
       .attr('fill', d => {
         const countryData = filteredData.find(fd => fd['Region/Country/Area name'] === d.properties.name);
         return countryData ? colorScale(countryData.Value) : '#ccc';
       })
       .on('mouseover', (event, d) => {
         const countryData = filteredData.find(fd => fd['Region/Country/Area name'] === d.properties.name);
         tooltipContent = countryData 
                          ? `Region: ${countryData['Region/Country/Area name']}, Value: ${countryData.Value}` 
                          : 'No data';
         tooltipStyle = `visibility: visible; top: ${event.pageY}px; left: ${event.pageX}px;`;
       })
       .on('mousemove', (event) => {
         tooltipStyle = `visibility: visible; top: ${event.pageY}px; left: ${event.pageX}px;`;
       })
       .on('mouseout', () => {
         tooltipContent = '';
         tooltipStyle = 'visibility: hidden;';
       });
  }

  function updateYear(event) {
    yearIndex = +event.target.value;
    selectedYear = allowedYears[yearIndex];
    drawChart();
  }
</script>

<main>
  <h1>How Does Internet Usage Evolve around the World During the Past Two Decades?</h1>
  <div class="slider-container">
    <input
      type="range"
      min="0"
      max="{allowedYears.length - 1}"
      step="1"
      bind:value="{yearIndex}"
      on:input="{updateYear}"
    />
    <span class="year-label">{selectedYear}</span>
  </div>
  <svg class="chart-svg" width="960" height="500"></svg>
  <div class="tooltip" style={tooltipStyle}>{tooltipContent}</div>
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
  .slider-container {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    margin: 20px 0;
  }

  main {
    text-align: center;
    font-family: 'Nunito', sans-serif;
    font-weight: 300;
    line-height: 2;
    font-size: 15px;
    color: var(--color-text);
  }

  h1 {
    font-size: 2em;
    font-weight: 300;
    line-height: 2;
  }

  .chart-svg {
    width: 100%; /* Ensure SVG fills the container */
    height: auto; /* Maintain aspect ratio */
    display: block; /* Remove default inline behavior */
    background-color: var(--color-bg); /* Optional: set a background color */
  }

  .tooltip {
    position: absolute;
    padding: 5px;
    background: rgba(0, 0, 0, 0.75);
    color: white;
    border-radius: 5px;
    pointer-events: none; /* Ignore mouse events on the tooltip itself */
    transition: opacity 0.3s;
  }
</style>
