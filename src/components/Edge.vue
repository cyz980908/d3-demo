<template>
<!-- https://bl.ocks.org/mbostock/1044242 -->
  <div id="my_edge">
  </div>
</template>

import { log } from 'console';

import { log } from 'console';

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  async mounted() {
    let d3 = window.d3
    let diameter = 960,
    radius = diameter / 2,
    innerRadius = radius - 200;

    var cluster = d3.cluster()
    .size([360, innerRadius]);

    var line = d3.radialLine()
    // .curve(d3.curveBundle.beta(0.6))
    .curve(d3.curveCardinal )
    .radius(function(d) { return d.y; })
    .angle(function(d) { return d.x / 180 * Math.PI; });

    var svg = d3.select("#my_edge").append("svg")
    .attr("width", diameter)
    .attr("height", diameter)
    .append("g")
    .attr("transform", "translate(" + radius + "," + radius + ")");

    var link = svg.append("g").selectAll(".link"),
    node = svg.append("g").selectAll(".node");

let classes = await d3.json("./flare.json")

  console.log('classes',classes);
  var root = packageHierarchy(classes)
      .sum(function(d) { return d.size; });

  cluster(root);

    link
    .data(packageImports(root.leaves()))
    .enter().append("path")
      .each(function(d) { d.source = d[0], d.target = d[d.length - 1]; })
      .attr("class", "link")
      .attr("d", line);
      
    node
    .data(root.leaves())
    .enter().append("text")
      .attr("class", "node")
      .attr("dy", "0.31em")
      .attr("transform", function(d) { return "rotate(" + (d.x - 90) + ")translate(" + (d.y + 8) + ",0)" + (d.x < 180 ? "" : "rotate(180)"); })
      .attr("text-anchor", function(d) { return d.x < 180 ? "start" : "end"; })
    //   .text(function(d) { return d.data.key; });

// Lazily construct the package hierarchy from class names.
function packageHierarchy(classes) {
  var map = {};

  function find(name, data) {
    var node = map[name], i;
    if (!node) {
      node = map[name] = data || {name: name, children: []};
      if (name.length) {
        node.parent = find(name.substring(0, i = name.lastIndexOf(".")));
        node.parent.children.push(node);
        node.key = name.substring(i + 1);
      }
    }
    return node;
  }

  classes.forEach(function(d) {
    find(d.name, d);
  });

  return d3.hierarchy(map[""]);
}

// Return a list of imports for the given array of nodes.
function packageImports(nodes) {
  var map = {},
      imports = [];

  // Compute a map from name to node.
  nodes.forEach(function(d) {
    map[d.data.name] = d;
  });

  // For each import, construct a link from the source to target node.
  nodes.forEach(function(d) {
    if (d.data.imports) d.data.imports.forEach(function(i) {
      imports.push(map[d.data.name].path(map[i]));
    });
  });

  return imports;
}
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less">
.node {
  font: 10px sans-serif;
}

.link {
  stroke: steelblue;
  stroke-opacity: 0.5;
  fill: none;
  pointer-events: none;
}
</style>
