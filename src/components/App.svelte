<script>
  import * as d3 from 'd3';
    
  import { onMount } from 'svelte';

    let titanicData = [];
    
    onMount(async () => {

        const res = await fetch('Titanic-Data.csv'); 

        const csv = await res.text();

        emissionData = d3.csvParse(csv, d3.autoType)

        $: console.log(titanicData);
        createBubbleChart();

    });


  function createBubbleChart() {
      const svg = d3.select("svg");
      const width = +svg.attr("width");
      const height = +svg.attr("height");
  
      // Define scales
      const xScale = d3.scaleLinear()
                       .domain(d3.extent(titanicData, d => d.Age))
                       .range([0, width]);
                       
      const yScale = d3.scaleLinear()
                       .domain(d3.extent(titanicData, d => d.Fare))
                       .range([height, 0]);
                       
      const colorScale = d3.scaleOrdinal()
                           .domain([0, 1])
                           .range(["red", "green"]);
  
      // Add circles
      svg.selectAll(".point")
         .data(titanicData)
         .enter().append("circle")
         .attr("class", "point")
         .attr("cx", d => xScale(d.Age))
         .attr("cy", d => yScale(d.Fare))
         .attr("r", 5) // Radius of circles
         .attr("fill", d => colorScale(d.Survived));
  
      // Optional: Add Axes
      const xAxis = d3.axisBottom(xScale);
      const yAxis = d3.axisLeft(yScale);
  
      svg.append("g")
         .attr("transform", `translate(0,${height})`)
         .call(xAxis);
  
      svg.append("g")
         .call(yAxis);
    }
</script>

<svg width="600" height="400"></svg>
