<template>
<!-- https://d3-graph-gallery.com/graph/circular_barplot_double.html -->
  <div id="my_dataviz">
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  async mounted() {
  const makeRepeated = (arr, repeats) =>[].concat(...Array.from({ length: repeats }, () => arr));
  
    let d3 = window.d3
    console.log(d3)
    // set the dimensions and margins of the graph
    var margin = {top: 100, right: 0, bottom: 0, left: 0},
        width = 460 - margin.left - margin.right,
        height = 460 - margin.top - margin.bottom,
        innerRadius = 100,
        outerRadius = 130;   // the outerRadius goes from the middle of the SVG area to the border

    // append the svg object
    var svg = d3.select("#my_dataviz")
      .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
      .append("g")
        .attr("transform", "translate(" + (width / 2 + margin.left) + "," + (height / 2 + margin.top) + ")");
        
      let postiveData = [ 1,5,3,5,5,7,5,2,3,7,7,4]
      let nagativeData = [ 1,5,3,5,5,7,5,2,3,7,7,4].map(item=>-item)
      let data = [...postiveData,...nagativeData]
      new Array(10).fill(1).forEach(()=>{
        let random = ~~(Math.random()*2)+2
        let arr = Math.random()>0.5?postiveData:nagativeData
        data.push(...makeRepeated(arr,random))
      })
      data = data.map((item,index)=>({
        Value: item,
        Name: index +''
      }))
      console.log(data)
      // X scale: common for 2 data series
      var x = d3.scaleBand()
          .range([0, 2 * Math.PI])    // X axis goes from 0 to 2pi = all around the circle. If I stop at 1Pi, it will be around a half circle
          .align(0)                  // This does nothing
          .domain(data.map(function(d) { return d.Name; })); // The domain of the X axis is the list of states.

      // Y scale outer variable
      var y = d3.scaleRadial()
          .range([innerRadius, outerRadius])   // Domain will be define later.
          .domain([0, 10]); // Domain of Y is from 0 to the max seen in the data

      // Second barplot Scales
      var ybis = d3.scaleRadial()
          .range([innerRadius, 70])   // Domain will be defined later.
          .domain([0, 10]);

      console.log(ybis)

      // Add the bars
      svg.append("g")
        .selectAll("path")
        .data(data)
        .enter()
        .append("path")
          .attr("fill", "#69b3a2") //green
          .attr("class", "yo")
          .attr("d", d3.arc()     // imagine your doing a part of a donut plot
              .innerRadius(innerRadius)
              .outerRadius( function(d) { return d['Value']<0?y(Math.abs(d['Value'])):ybis(0) })
              .startAngle(function(d) { return x(d.Name); })
              .endAngle(function(d) { return x(d.Name) + x.bandwidth(); })
              .padAngle(0.01)
              .padRadius(innerRadius))

      // Add the second series
      svg.append("g")
        .selectAll("path")
        .data(data)
        .enter()
        .append("path")
          .attr("fill", "red")
          .attr("d", d3.arc()     // imagine your doing a part of a donut plot
              .innerRadius( function() { return ybis(0) })
              .outerRadius( function(d) { return d['Value']>0?ybis(d['Value']):ybis(0) })
              .startAngle(function(d) { return x(d.Name); })
              .endAngle(function(d) { return x(d.Name) + x.bandwidth(); })
              .padAngle(0.01)
              .padRadius(innerRadius))
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
</style>
