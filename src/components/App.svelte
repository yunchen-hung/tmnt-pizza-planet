<!-- Code from d3-graph-gallery.com -->
<!DOCTYPE html>
<meta charset="utf-8">

<!-- Load d3.js -->
<script src="https://d3js.org/d3.v4.js"></script>

<!-- Initialize a select button -->
<select id="selectButton"></select>

<!-- Create a div where the graph will take place -->
<div id="my_dataviz"></div>


<script>

  // set the dimensions and margins of the graph
  var margin = {top: 40, right: 70, bottom: 80, left: 70},
    width = 460 - margin.left - margin.right,
    height = 400 - margin.top - margin.bottom;

  // append the svg object to the body of the page
  var svg = d3.select("#my_dataviz")
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform",
      "translate(" + margin.left + "," + margin.top + ")");

  // Read the data
  d3.csv("static/yelpilicious_data.csv", function (data) {
    console.log(data)
    // List of groups (each state is an option)
    var allGroup = d3.map(data, function (d) { return (d.state) }).keys()

    // add the options to the button
    d3.select("#selectButton")
      .selectAll('myOptions')
      .data(allGroup)
      .enter()
      .append('option')
      .text(function (d) { return d; }) // text showed in the menu
      .attr("value", function (d) { return d; }) // corresponding value returned by the button

    // A color scale: one color for each group
    var myColor = d3.scaleOrdinal()
      .domain(allGroup)
      .range("pink", "purple", "royalblue");

    
    firstFilter = data.filter(function (d) { return d.state == allGroup[0] })
    console.log(firstFilter)
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
      .text("Stars Given To Restaurant");

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
        .text("Proportion of Counts")

    // add title 
    svg.append("text")
        .attr("x", (width / 2)) 
        .attr("y", 0 - (margin.top / 2))            
        .attr("text-anchor", "middle")  
        .style("font-size", "15px") 
        .attr("font-family","sans-serif")
        .text("Which State Should The TMNT Fly To For Pizza?");

    // Bars
    svg.selectAll("mybar")
      .data(firstFilter)
      .enter()
      .append("rect")
      .attr("x", function (d) { return x(d.stars); })
      .attr("y", function (d) { return y(+d.count); })
      .attr("width", x.bandwidth())
      .attr("height", function (d) { return height - y(+d.count); })
      .attr("fill", "#69b3a2")


    // A function that update the chart
    function update(selectedGroup) {

      // Create new data with the selection?
      var dataFilter = data.filter(function (d) { return d.state == selectedGroup })
      // remove  bars
      svg.selectAll("mybar").remove()
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
        .attr("fill", "#69b3a2")
        .delay(function (d, i) { console.log(i); return (i * 100) })

    }

    // When the button is changed, run the updateChart function
    d3.select("#selectButton").on("change", function (d) {
      // recover the option that has been chosen
      var selectedOption = d3.select(this).property("value")
      // run the updateChart function with this selected option
      update(selectedOption);
    })

  })


</script>