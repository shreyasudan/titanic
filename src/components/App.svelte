<head>
<style>
  body {
     //background-color: #cccccc;
     /* Center aligning the text */
     text-align: right;
  }
  h1 {
    color: black; /* Changing text color */
    text-align: center;
  }
  h2 {
    text-align: center;
    font-size: 20px;

  }
  p {
    color: black; /* Changing text color */
    font-size: 20px;
    text-align: center;
  }

  h3 {
    text-align: left;
    font-size: 15px;
    font-weight: normal;
  }
  h4 {
    text-align: left;
  }
  h5 {
    text-align: center;
    font-size: 20px;
  }
  h6 {
    text-align: center;
    font-size: 17px;
    font-weight: normal;

  }



</style>
</head>

<body>
<h1>The Titanic Challenge: Do You Have What It Takes to Survive?</h1>
<h2>Unveiling Titanic: Age, Fare, and Survival Insights</h2>
<h3></h3>
</body>

<script>
  import * as d3 from 'd3';
  import { onMount } from 'svelte';

  let titanicData = [];

  onMount(async () => {
    const res = await fetch('Titanic-Dataset.csv'); 
    const csv = await res.text();
    titanicData = d3.csvParse(csv, d3.autoType);
    console.log(titanicData);
    createBubbleChart(); // Ensure this is called after data is loaded
    createSurvivalHistogram();
  });

  function createBubbleChart() {
    // Declare and initialize these variables at the start
    const svgWidth = 800; // Set SVG width
    const svgHeight = 650; // Set SVG height

    // Adjust margins dynamically
    const margin = { top: 20, right: 30, bottom: 60, left: 70 };
    const width = svgWidth - margin.left - margin.right;
    const height = svgHeight - margin.top - margin.bottom;

    // Adjust SVG selection to account for margins
    const svg = d3.select("svg")
      .attr("width", svgWidth)
      .attr("height", svgHeight)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    // Define scales
    const xScale = d3.scaleLinear()
      .domain([0, d3.max(titanicData, d => d.Age)]).nice()
      .range([0, width]);

    // Adjusted y-scale with step increments of 500 or 1000
    const yScale = d3.scaleLinear()
      .domain([0, d3.max(titanicData, d => d.Fare * 31.80)]).nice()
      .range([height, 0]);

    const colorScale = d3.scaleOrdinal()
      .domain([0, 1])
      .range(["#355C7D", "#F7DB4F"]);

    const filteredData = titanicData.filter(d => d.Age !== null)

    // Add circles
    const circles = svg.selectAll(".point")
      .data(filteredData)
      .enter().append("circle")
      .attr("class", "point")
      .attr("cx", d => xScale(d.Age))
      .attr("cy", d => yScale(d.Fare * 31.80))
      .attr("r", 12) // Radius of circles
      .attr("fill", d => colorScale(d.Survived))
      .attr("stroke", d => colorScale(d.Survived))
			.attr("stroke-width", 1)
			.attr("fill-opacity", 0.8);
    

    // X-axis and Y-axis
    svg.append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(xScale));

    // Adjusted y-axis with step increments of 500 or 1000
    svg.append("g")
      .call(d3.axisLeft(yScale).tickValues(d3.range(0, d3.max(titanicData, d => d.Fare * 31.80), 1000))); // Adjusted tick values

    svg.append("text")
    .attr("class", "axis-label")
    .attr("x", width / 2)
    .attr("y", height + margin.top + 20) // Adjust position below the x-axis
    .style("text-anchor", "middle")
    .text("Age");

  // Y-axis label
  svg.append("text")
    .attr("class", "axis-label")
    .attr("transform", "rotate(-90)")
    .attr("x", -height / 2)
    .attr("y", -margin.left + 15) // Adjust position to the left of the y-axis
    .style("text-anchor", "middle")
    .text("Fare ($)");

  const legend = svg.append("g")
    .attr("class", "legend")
    .attr("transform", `translate(${width + margin.right-125},${margin.top})`);

  const legendColors = ["#355C7D", "#F7DB4F"];
  const legendLabels = ["Survived", "Did not survive"];

  legend.selectAll(".legend-item")
    .data(legendColors)
    .enter().append("g")
    .attr("class", "legend-item")
    .attr("transform", (d, i) => `translate(0, ${i * 20})`);

  legend.selectAll(".legend-item")
    .append("circle")
    .attr("cx", 0)
    .attr("cy", 0)
    .attr("r", 6)
    .style("fill", d => d);

  legend.selectAll(".legend-item")
    .append("text")
    .attr("x", 10)
    .attr("y", 5)
    .text((d, i) => legendLabels[i]);

  legend.selectAll(".legend-item")
    .on("mouseover", function(event, d) {
      // Reduce opacity of non-hovered circles
      svg.selectAll(".point")
        .attr("fill-opacity", data => colorScale(data.Survived) === d ? 0.8 : 0)
        .attr("stroke-opacity", data => colorScale(data.Survived) === d ? 1 : 0);
    })
    .on("mouseout", function() {
      // Reset opacity of circles
      svg.selectAll(".point")
        .attr("fill-opacity", 0.8)
        .attr("stroke-opacity", 1);
    });
  
  svg.append("text")
        .attr("x", width - margin.right - 25)
        .attr("y", margin.top + 80) // Position above the legend
        .attr("text-anchor", "end")
        .style("font-size", "17px")
        .style("font-weight", "bold")
        .text("Hover over legend to change visualization");


  const tooltip = d3.select("h3").append("div")
    .attr("class", "tooltip")
    .style("opacity", 0);

  svg.selectAll("circle")
      .on("mouseover", function(event, d) {
          tooltip.transition()
              .duration(200)
              .style("opacity", .9);
          tooltip.html(`Passenger Age: ${d.Age}<br>Passenger Fare: ${d["Fare"]}<br>Gender: ${d.Sex}`)
              .style("left", (event.pageX) + "px")
              .style("top", (event.pageY) + "px");
      })
      .on("mouseout", function(d) {
          tooltip.transition()
              .duration(500)
              .style("opacity", 0);
      });
  }

function createSurvivalHistogram() {
    const svgWidth = 800; // Set SVG width
    const svgHeight = 650; // Set SVG height

    // Adjust margins dynamically
    const margin = { top: 20, right: 30, bottom: 60, left: 50 };
    const width = svgWidth - margin.left - margin.right;
    const height = svgHeight - margin.top - margin.bottom;

    var svg = d3.select("h6")
            .append("svg")
                .attr("width", width + margin.left + margin.right)
                .attr("height", height + margin.top + margin.bottom)
            .append("g")
                .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

// X axis: scale and draw
var x = d3.scaleLinear()
            .domain([0, d3.max(titanicData, d => d.Age)])
            .range([0, width]);
svg.append("g")
    .attr("transform", "translate(0," + height + ")")
    .call(d3.axisBottom(x));

// Set up histogram function
var histogram = d3.histogram()
    .value(d => d.Age)
    .domain(x.domain())
    .thresholds(x.ticks(45));

// Apply histogram function to data to get the bins for 'Survived' = 1
var bins1 = histogram(titanicData.filter(d => d.Survived === 1));

// Apply histogram function to data to get the bins for 'Survived' = 0
var bins0 = histogram(titanicData.filter(d => d.Survived === 0));


const total1 = bins1.length > 0 ? bins1.map(bin => bin.length).reduce((a, b) => a + b) : 1;
const total0 = bins0.length > 0 ? bins0.map(bin => bin.length).reduce((a, b) => a + b) : 1;
bins1.forEach(bin => { bin.proportion = bin.length / total1; });
bins0.forEach(bin => { bin.proportion = bin.length / total0; });

// Y axis: scale and draw
var y = d3.scaleLinear()
            .range([height, 0])
            .domain([0, 0.1]); // Set the domain to [0, 1]
svg.append("g")
    .call(d3.axisLeft(y).ticks(10)); // Format ticks as percentages

// Append the bars for 'Survived' = 1
svg.selectAll(".bar1")
    .data(bins1)
    .enter()
    .append("rect")
    .attr("class", "bar1")
    .attr("x", d => x(d.x0))
    .attr("y", d => y(d.proportion)) // Use proportion instead of length
    .attr("width", d => x(d.x1) - x(d.x0) - 1)
    .attr("height", d => height - y(d.proportion))
    .style("fill", "#69b3a2")
    .style("opacity", 0.6);

// Append the bars for 'Survived' = 0
svg.selectAll(".bar0")
    .data(bins0)
    .enter()
    .append("rect")
    .attr("class", "bar0")
    .attr("x", d => x(d.x0))
    .attr("y", d => y(d.proportion)) // Use proportion instead of length
    .attr("width", d => x(d.x1) - x(d.x0) - 1)
    .attr("height", d => height - y(d.proportion))
    .style("fill", "#404080")
    .style("opacity", 0.6);

  
svg.append("text")
    .attr("class", "axis-label")
    .attr("x", width / 2)
    .attr("y", height + margin.top + 20) // Adjust position below the x-axis
    .style("text-anchor", "middle")
    .text("Age");

  // Y-axis label
  svg.append("text")
    .attr("class", "axis-label")
    .attr("transform", "rotate(-90)")
    .attr("x", -height / 2)
    .attr("y", -margin.left + 11.25) // Adjust position to the left of the y-axis
    .style("text-anchor", "middle")
    .text("Density");

}


  
</script>

<svg style="display:block;margin:auto;"></svg>

<body>
<h4 style="text-align: center; font-size: 24px;">Write-Up</h4>

<p> Our objective with this project is to create a unique user experience that plays with 
their odds against one of the most sensationalized tradegies: Titanic. We have created an interactive 
visualization that allows for Exploratory Data Analysis at the hands of the user. Some notable encodings
in our visualization include an interactive legend, allowing our users to look at the difference in 
distributions of the survivors and the victims of this unforseen calamity. We applied a data transformation to
the "Fare" of the passengers. We wanted to communicate the value of the fare in today's monetary value. Thus,
we applied the inflation rate of 31.80 that we computed from the <a href="https://data.bls.gov/cgi-bin/cpicalc.pl">Bureau of Labor Statistics</a>.
Finally, we utilized tooltips to allow information on the data points that is not overtly obvious in the visualization. 
For instance, the tooltips discloses the 'Sex' of the passanger to allow the user to conclude that most survivors of the Titanic
were actually female.
</p>

<p> Our endgame for this project, however, is much more ambitious. After performing some preliminary EDA,
we observed that much of our data points seemed like a constant with a few outliers. We would like to log-scale the 'Fare'
to manage these outliers in order to highlight the true relationship between 'Age' and 'Fare'. Moreover, we plan to add 
another exploratory visualization to highlight the relationship between the variables 'Sex', 'Number of Dependents',
and 'Survival'. Next we'd like to create a somewhat of a Classification model that allows the user to enter information and see
their odds for survival. We aim to do this by applying a "Nearest Neighbor Classifier" that zooms to the bubble with
their stats. Presenltly, we are also considerening a "Bayes Classifier" that attempts to provide them with a quantifiable 
float values as their 'Survival Odds' 
</p>

<h5> Feature Engineering </h5>
<h6></h6>
</body>
