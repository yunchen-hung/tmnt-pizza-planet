<meta charset="utf-8">

<!-- Initialize a select button -->
<select id="selectButton"></select>

<!-- Create a div where the graph will take place -->
<div id="my_dataviz"></div>


<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let data = [];

  onMount(async () => {

    const res = await fetch('yelpilicious_data.csv'); 

    const csv = await res.text();

    data = d3.csvParse(csv, d3.autoType)

    console.log(data);

    var margin = {top: 50, right: 70, bottom: 80, left: 70},
    width = 600 - margin.left - margin.right,
    height = 500 - margin.top - margin.bottom;

    var svg = d3.select("#my_dataviz")
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform",
      "translate(" + margin.left + "," + margin.top + ")");

    var allGroup = [... new Set(data.map(d => d.state))];
    
    // add the options to the button
    d3.select("#selectButton")
      .selectAll('myOptions')
          .data(allGroup)
        .enter()
          .append('option')
          .text(d => d)
          .attr('value', d => d);
    console.log(allGroup)

    var myColor = d3.scaleOrdinal()
      .domain(allGroup)
      .range(["#332288", "#117733", "#44AA99", "#88CCEE", 
      "#DDCC77", "#CC6677", "#AA4499", "#882255", 
      "#E69F00", "#56B4E9", "#009E73", "#F0E442", "#D55E00", "#CC79A7"]);
    
    var firstFilter = data.filter(function (d) { return d.state == allGroup[0] })
   
    //x axis mean line
    var xMean = d3.scaleLinear()
  .range([30, width])
  .domain([1, 5.3]); // Adjust the domain as needed
    // X axis
    var x = d3.scaleBand()
      .range([0, width])
      .domain(data.map(function (d) { return d.stars; }))
      .padding(0.2);
    svg.append("g")
      .attr("transform", "translate(0," + height + ")")
      .call(d3.axisBottom(x))
      .selectAll("text")
      .attr("transform", "translate(-10,0)rotate(-45)")
      .style("text-anchor", "end");

    // x axis text
    svg.append("text")
      .attr("text-anchor", "end")
      .attr("x", width)
      .attr("y", height + margin.top + 20)
      .style("font-size", "15px") 
      .attr("font-family","sans-serif")
      .text("Stars Given To Pizza Restaurant");
    
    // Add Y axis
    var y = d3.scaleLinear()
      .domain([0, 1])
      .range([height, 0]);
    svg.append("g")
      .call(d3.axisLeft(y));
    
    // Y axis label:
    svg.append("text")
        .attr("text-anchor", "end")
        .attr("transform", "rotate(-90)")
        .attr("y", -margin.left+20)
        .attr("x", -margin.top)
        .style("font-size", "15px") 
        .attr("font-family","sans-serif")
        .text("Proportion of Counts For Each Review")

     // add title 
     svg.append("text")
        .attr("x", (width / 2)) 
        .attr("y", 0 - (margin.top / 2))            
        .attr("text-anchor", "middle")  
        .style("font-size", "17px") 
        .attr("font-family","sans-serif")
        .text("Which State Should The Teenage Mutant Ninja Turtles Fly To For Pizza?");

    
    // Bars
    svg.selectAll("mybar")
      .data(firstFilter)
      .enter()
      .append("rect")
      .attr("x", function (d) { return x(d.stars); })
      .attr("y", function (d) { return y(+d.count); })
      .attr("width", x.bandwidth())
      .attr("height", function (d) { return height - y(+d.count); })
      .style("fill", function(d){ return myColor(allGroup[0]) })
      // add mean line 
    console.log(d3.sum(firstFilter, d => d.stars * d.count))
    svg.selectAll("mean")
      .data(firstFilter)
      .enter()
      .append("line")
      .style("stroke", "red")
      .style("stroke-width", 2)
      .attr("x1", function (d) { return xMean(d3.sum(firstFilter, d => d.stars * d.count)); })
      .attr("y1", 0)
      .attr("x2", function (d) { return xMean(d3.sum(firstFilter, d => d.stars * d.count)); })
      .attr("y2", height)
      .append("title")  // Append a <title> element to each line
      .text("The mean of the ratings are " + d3.sum(firstFilter, d => d.stars * d.count));   


      //BOX with count
      var boxWidth = 140;
      var boxHeight = 50;
      var boxX = width-75; 
      var boxY = 20; 

      // box
      var textBox = svg.append("g")
          .attr("transform", "translate(" + boxX + "," + boxY + ")");

      // rectangle
      textBox.append("rect")
          .attr("width", boxWidth)
          .attr("height", boxHeight)
          .style("fill", "lightgray")
          .style("stroke", "black");

      // text
      textBox.append("text")
          .attr("x", boxWidth / 2)
          .attr("y", boxHeight / 2)
          .attr("text-anchor", "middle")
          .attr("alignment-baseline", "middle")
          .text("# of review(s): " + firstFilter[0].num_reviews);
        console.log(firstFilter[0].num_reviews)
      

     // A function that update the chart
    function update(selectedGroup) {
      
      // Create new data with the selection?
      var dataFilter = data.filter(function (d) { return d.state == selectedGroup })
      //const mean = d3.mean(dataFilter.values(count))
      console.log(dataFilter)
      
      // remove  bars
      svg.selectAll("rect")
        .transition()
          .duration(100)
        .remove()
      // remove lines
        svg.selectAll("line")
        .transition()
          .duration(100)
        .remove()
      svg.append("g")
        .attr("transform", "translate(0," + height + ")")
        .call(d3.axisBottom(x))
        .selectAll("text")
        .attr("transform", "translate(-10,0)rotate(-45)")
        .style("text-anchor", "end");
      // Give these new data to update bar
      svg.selectAll("mybar")
        .data(dataFilter)
        .enter()
        .append("rect")
        .transition()
        .duration(800)
        .attr("x", function (d) { return x(d.stars); })
        .attr("y", function (d) { return y(+d.count); })
        .attr("width", x.bandwidth())
        .attr("height", function (d) { return height - y(+d.count); })
        .attr("fill",  function(d){ return myColor(selectedGroup) })
        .delay(function (d, i) {return (i * 100) })
      //mean line
      console.log(d3.sum(dataFilter, d => d.stars * d.count))
      svg.selectAll("mean")
        .data(dataFilter)
        .enter()
        .append("line")
        .style("stroke", "red")
        .style("stroke-width", 2)
        .attr("x1", function (d) { return xMean(d3.sum(dataFilter, d => d.stars * d.count)); })
        .attr("y1", 0)
        .attr("x2", function (d) { return xMean(d3.sum(dataFilter, d => d.stars * d.count)); })
        .attr("y2", height)
        .append("title")  // Append a <title> element to each line
        .text("The mean of the ratings are " + d3.sum(dataFilter, d => d.stars * d.count));   

        //annotation
        // box
      var textBox = svg.append("g")
          .attr("transform", "translate(" + boxX + "," + boxY + ")");

      // rectangle
      textBox.append("rect")
          .attr("width", boxWidth)
          .attr("height", boxHeight)
          .style("fill", "lightgray")
          .style("stroke", "black");

      // text
      textBox.append("text")
          .attr("x", boxWidth / 2)
          .attr("y", boxHeight / 2)
          .attr("text-anchor", "middle")
          .attr("alignment-baseline", "middle")
          .text("# of review(s): " + dataFilter[0].num_reviews);
        console.log(dataFilter[0].num_reviews)
    }
    // When the button is changed, run the updateChart function
    d3.select("#selectButton").on("change", function (d) {
      // recover the option that has been chosen
      
      var selectedOption = d3.select(this).property("value")
      // run the updateChart function with this selected option
      update(selectedOption);
      
    })

    });
 </script>

 