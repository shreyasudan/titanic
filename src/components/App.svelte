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
    font-size: 17px;
  }
  h6 {
    text-align: center;
    font-size: 17px;
    font-weight: strong;
  }




</style>
</head>

<body>
<h1>The Curse of the Titanic</h1>
<p> 
Step back in time to April 1912, a momentous period marked by the inaugural voyage of the RMS Titanic,
a marvel of engineering and opulence touted as "unsinkable." As we embark on our journey through the annals of history, 
we delve into the heart of one of the most infamous maritime disasters ever to unfold—the sinking of the Titanic. 
Through the lens of the Titanic Olympics, we pose a daring question: 
</p>

<p> Would you have what it takes to survive the treacherous waters of the North Atlantic aboard this legendary vessel? </p>

<p>
The visualization below explores the relationship between passenger age, fare paid, and survival aboard the Titanic, 
shedding light on the dynamics of this historic tragedy.
</p> 

<p>
The visualization illustrates a scatter plot where each circle represents a passenger. 
The x-axis denotes passenger age, while the y-axis reflects the fare paid. 
Circles are color-coded based on survival status—blue for survived and yellow for those who did not. 
Hovering over each circle reveals additional details, including passenger age, fare paid, and gender.
</p>

<h2>Titanic Time Machine: Voyage Through Age and Fare</h2>

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
    createSurvivalHistogram(titanicData);
    
    
    document.getElementById('maleButton').addEventListener('click', () => filterData('male'));
    document.getElementById('femaleButton').addEventListener('click', () => filterData('female'));
    document.getElementById('bothButton').addEventListener('click', () => filterData('both'));
    

    function filterData(sex) {
      let filteredData;
      if (sex === 'male') {
        // Filter data for male passengers
        filteredData = titanicData.filter(d => d.Sex === 'male');
      } else if (sex === 'female') {
        // Filter data for female passengers
        filteredData = titanicData.filter(d => d.Sex === 'female');
      } else {
        // Use the original data (both male and female passengers)
        filteredData = titanicData;
      }

      // Update the histogram
      d3.select("h5").selectAll("svg").remove();
      createSurvivalHistogram(filteredData);
  }
    
    

    document.getElementById('predictionForm').addEventListener('submit', function(event) {
      console.log('Form submitted');
      event.preventDefault(); // Prevent form submission

      


      // Get user input values
      const sex = document.getElementById('sex').value;
      const embarked = document.getElementById('embarked').value;
      const pclass = document.getElementById('pclass').value;
      

      // Call the classifier function to predict survival
      classifier({ Sex: sex, Embarked: embarked, Pclass: pclass });

      // Update visualization based on the prediction
      // updateVisualization(prediction);
    });
    
  

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

function createSurvivalHistogram(titanicData) {
  const svgWidth = 800; // Set SVG width
  const svgHeight = 650; // Set SVG height

  // Adjust margins dynamically
  const margin = { top: 20, right: 30, bottom: 60, left: 50 };
  const width = svgWidth - margin.left - margin.right;
  const height = svgHeight - margin.top - margin.bottom;

  var svg = d3.select("h5")
            .append("svg")
                .attr("width", width + margin.left + margin.right)
                .attr("height", height + margin.top + margin.bottom)
            .append("g")
                .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

  // X axis: scale and draw
  var x = d3.scaleLinear()
              .domain([0, 80])
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

  var legend = svg.selectAll(".legend")
      .data(["Survived", "Did not survive"])
      .enter().append("g")
      .attr("class", "legend")
      .attr("transform", function(d, i) { return "translate(0," + i * 20 + ")"; });

  legend.append("rect")
      .attr("x", width - 18)
      .attr("width", 18)
      .attr("height", 18)
      .style("fill", function(d, i) { return i === 0 ? "#69b3a2" : "#404080"; });

  legend.append("text")
      .attr("x", width - 24)
      .attr("y", 9)
      .attr("dy", ".35em")
      .style("text-anchor", "end")
      .text(function(d) { return d; });
    
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

function classifier({ Sex, Embarked, Pclass }) {
  // Filter data for survived and not survived passengers
  const survivedData = titanicData.filter(d => d.Survived === 1);
  const notSurvivedData = titanicData.filter(d => d.Survived === 0);

  // Calculate prior probabilities
  const totalPassengers = titanicData.length;
  const priorSurvived = survivedData.length / totalPassengers;
  const priorNotSurvived = notSurvivedData.length / totalPassengers;

  // Calculate likelihoods for 'Sex', 'Embarked', and 'Pclass' features given each class
  const likelihoods = {
    survived: {
      sex: {
        
      },
      embarked: {},
      pclass: {}
    },
    notSurvived: {
      sex: {},
      embarked: {},
      pclass: {}
    }
  };

  ['Sex', 'Embarked', 'Pclass'].forEach(feature => {

    likelihoods.survived[feature] = {};
    likelihoods.notSurvived[feature] = {};

    // Calculate likelihoods for each category of the feature
    const categories = Array.from(new Set(titanicData.map(d => d[feature])));
    categories.forEach(category => {
      likelihoods.survived[feature][category] = survivedData.filter(d => d[feature] === category).length / survivedData.length;
      likelihoods.notSurvived[feature][category] = notSurvivedData.filter(d => d[feature] === category).length / notSurvivedData.length;
    });
  });

  // Implement function to predict class based on feature values
  function predictClass({ Sex, Embarked, Pclass }) {
    const likelihoodSurvived = priorSurvived * likelihoods.survived.sex[Sex] * likelihoods.survived.embarked[Embarked] * likelihoods.survived.pclass[Pclass];
    const likelihoodNotSurvived = priorNotSurvived * likelihoods.notSurvived.sex[Sex] * likelihoods.notSurvived.embarked[Embarked] * likelihoods.notSurvived.pclass[Pclass];
    //console.log(likelihoodSurvived > likelihoodNotSurvived ? 1 : 0);

    //prediction = likelihoodSurvived > likelihoodNotSurvived ? 1 : 0
    if (likelihoodSurvived > likelihoodNotSurvived ) {
      //return '<img src="lifeboat.jpg" alt="Survived" />';
      return 'Survived'
    }
    else {
      //return <img src="iceberg.jpg" alt="Did not Survive" />;
      return 'Did not Survive'
    }
  }

  const likelihood = predictClass({ Sex: Sex, Embarked: Embarked, Pclass: Pclass });

  // Update the DOM to display the likelihood value
  const resultContainer = document.getElementById('resultContainer');
  resultContainer.innerText = `Survival Likelihood: ${predictClass({ Sex, Embarked, Pclass })}`;

  // Test the classifier function with some example feature values
  //const prediction = predictClass({ Sex: 'female', Embarked:'Southampton' , Pclass: 3 });
  //console.log('Predicted class:', prediction);
}



</script>

<svg style="display:block;margin:auto;"></svg>

<body>
<p>
A closer examination reveals a stark reality: amidst the luxury and grandeur of the Titanic, survival was not guaranteed for all. 
Despite paying varying fares and spanning different age groups, many passengers tragically perished in the icy waters of the North Atlantic.
Although some might believe that age would play a huge factor in survival, as the procedure allowed women and children to evacuate first, 
the visualization clearly displays how most survivors were actually between the ages of 20 to 60.
</p>

<p>
This visualization serves as a poignant reminder of the unpredictability of life and the harrowing choices faced by Titanic passengers. 
It prompts reflection on the factors that influenced survival—perhaps age, fare class, or sheer luck—and ignites curiosity 
about the individual stories hidden within the data. As we delve deeper into the Titanic's narrative, 
we embark on a quest to uncover the untold tales of courage, sacrifice, and resilience that define this enduring maritime disaster.
</p>

<p> Hmmm... this scatter plot looks almost bell-shaped. I wonder... </p>

<div style="text-align: center;">
  <button id="maleButton">Male</button>
  <button id="femaleButton">Female</button>
  <button id="bothButton">Both</button>
</div>

<h4 style="text-align: center; font-size: 20px;">Gender Bender: How Did Sex And Age Win You A Ticket To The Lifeboat? </h4>
<h5></h5>

<p> 
In our quest to understand the dynamics of survival aboard the Titanic, 
we delve into the age distribution of passengers through a pair of histograms. 
These visualizations illuminate the relationship between age and survival, 
providing key insights into the tragic events of that fateful night.
</p>

<p>
The histograms depict the age distribution of passengers who survived (in teal) and those who did not (in navy blue). 
Each bar represents a range of ages, while the height of the bars signifies the density of passengers within that age range. 
Notably, the histograms reveal distinct patterns in the age distribution of survivors and non-survivors, 
hinting at potential underlying factors influencing survival rates.
</p>

<p> 
A closer examination uncovers nuances within the age distributions of survivors and non-survivors. 
While both histograms exhibit peaks and troughs indicative of Gaussian-like distributions, 
subtle differences emerge in the shape and spread of the curves. 
These disparities underscore the complex interplay between age, survival, and other variables yet to be explored.
</p>

<p>
As we unravel the mysteries hidden within these histograms, 
we embark on a journey to train predictive models that harness the power of Gaussian distributions.
By leveraging insights gleaned from the histograms, we aim to develop robust algorithms 
capable of forecasting survival outcomes based on passenger characteristics. Through this innovative approach, 
we seek to honor the memory of those lost aboard the Titanic by shedding light on the factors that determined their fate.
</p>


<h6 style="text-align: center; font-size: 20px;">The Titanic Challenge: Do You Have What It Takes To Survive?</h6>
  <style>
    form {
      text-align: center; /* Center the form */
      margin-top: 20px; /* Add some top margin for spacing */
    }

    form label,
    form select {
      display: inline-block; /* Display elements in the form on the same line */
      margin-right: 10px; /* Add some right margin for spacing between elements */
    }

    form button {
      display: block; /* Make the button a block element to move it to a new line */
      margin: 10px auto; /* Center the button horizontally and add some spacing */
      font-size: 1em; /* Increase the font size */
      //padding: 8px 16px; 
    }

    form br {
      display: none; /* Hide line breaks */
    }
  </style>

  <form id="predictionForm">
    <label for="sex">Sex:</label>
    <select id="sex" name="sex">
      <option value="male">Male</option>
      <option value="female">Female</option>
    </select><br>

    <label for="embarked">Embarked:</label>
    <select id="embarked" name="embarked">
      <option value="S">Southampton</option>
      <option value="C">Cherbourg</option>
      <option value="Q">Queenstown</option>
    </select><br>

    <label for="pclass">Passenger Class:</label>
    <select id="pclass" name="pclass">
      <option value="1">First Class</option>
      <option value="2">Second Class</option>
      <option value="3">Third Class</option>
    </select><br>

    <button type="submit">Predict Survival</button>
  </form>

  <style>
    /* Center the result container */
    #resultContainer {
      text-align: center;
      margin-top: 20px;
    }

    /* Make the text inside the result container bigger */
    #resultContainer {
      font-size: 1.2em;
    }
  </style>

  <div id="resultContainer"></div>
</body>
