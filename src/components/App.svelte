<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import * as topojson from 'topojson-client'; // Import TopoJSON library
  import { feature } from 'topojson-client';
  import Internet from './Internet.svelte';
  
  let data = [];
  let svg;

  onMount(async () => {
    const res = await fetch('Internet_usage_data.csv');
    const csv = await res.text();
    data = await d3.csvParse(csv, d3.autoType);
    console.log(data.length);
    
    svg = document.querySelector('.chart-svg');
    drawChart(); 
  });

  async function drawChart() {
    if (data.length === 0 || !svg) return;
    const margin = {top: 1000, right: 0, bottom: -10, left: 250},
          width = 250,
          height = 100;

    const worldMapResponse = await fetch('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json');
    const worldMap = await worldMapResponse.json();
    const countries = feature(worldMap, worldMap.objects.countries);

    const filteredData = data.filter(d => d.Year === 2010);
    const mappedData = filteredData.map(d => ({ name: d['Region/Country/Area name'], value: d.Value }));
    const color = d3.scaleSequential()
                   .domain(d3.extent(mappedData, d => d.value))
                   .interpolator(d3.interpolateYlGnBu);

    // Clear previous SVG contents if any
    d3.select(svg).selectAll("*").remove();

    const svgContent = d3.select(svg)
                         .attr('viewBox', `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`)
                         .style("transform", "rotateX(180deg)")
                         .append('g')
                         .attr('transform', `translate(${margin.left}, ${margin.top})`);

    // Append country paths
    svgContent.selectAll("path")
              .data(countries.features)
              .enter().append("path")
                .attr("fill", d => {
                  const countryData = mappedData.find(item => item.name === d.properties.name);
                  return countryData ? color(countryData.value) : "gray";
                })
                .attr("d", d3.geoPath())
                .on("mouseover", function(event, d) {
                  const countryName = d.properties.name;
                  const countryData = mappedData.find(item => item.name === countryName);
                  const tooltipText = countryData ? `Country: ${countryName}<br/>Value: ${countryData.value}` : `Country: ${countryName}<br/>No data available`;
                  showTooltip(tooltipText, event.offsetX, event.offsetY);
                })
                .on("mouseout", function() {
                  hideTooltip();
                });

      // Add country borders
      svgContent.append("path")
                .datum(topojson.mesh(worldMap, worldMap.objects.countries))
                .attr("fill", "none")
                .attr("stroke", "white")
                .attr("stroke-width", 0.5) 
                .attr("d", d3.geoPath());

      function showTooltip(text, x, y) {
        const tooltip = d3.select('#tooltip');
        tooltip.html(text)
              .style("left", (x + 10) + "px")
              .style("top", (y - 28) + "px")
              .style("opacity", 1);
      }

      function hideTooltip() {
        const tooltip = d3.select('#tooltip');
        tooltip.transition()
              .duration(500)
              .style("opacity", 0);
      }
    }
</script>

<main>
  <h1>Internet Usage Project</h1>
  <svg class="chart-svg"></svg>
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
