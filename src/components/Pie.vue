<template>
  <div id="pie">
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  async mounted() {
      let colors = ['#a87b64','#e65832','#da1d23', '#e1c315', '#0aa3b5','#3aa255' ]
      let arr = colors.reduce((res,item)=>{
      let obj = [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15].map(()=>{ 
          let temp = item == '#3aa255'?  Math.random()*0.6+0.4:Math.random()*0.3
          return{
          name:'depth1',
          itemStyle: {
            color: '#ffff00',
            opacity: temp
          },
          children: [
          {
              name: 'depth2',
              itemStyle: {
              color: item,
              opacity: Math.random()
              },
              children: [
                  {
                      name: 'depth3',
                      value: 1,
                      itemStyle: {
                      color: item,
                      opacity: Math.random()
                      },
                  }
              ]
          }
          ]
      }})
      res.push(...obj)
      return res
      },[])
      let data = {
        name: 'flare',
        children: arr
      }
      console.log(data);
      this.init(data)
  },
  methods:{
    init(data){
      let root = ''
      const d3 = window.d3
      const partition = data => {
        const root = d3.hierarchy(data).sum(d => d.value).sort((a, b) => b.value - a.value);
        return d3.partition().size([2 * Math.PI, root.height + 1])(root);
      }
      const arcVisible = d => {
        return d.y1 <= 3 && d.y0 >= 1 && d.x1 > d.x0;
      }
      const labelVisible = d =>{
        return d.y1 <= 3 && d.y0 >= 1 && (d.y1 - d.y0) * (d.x1 - d.x0) > 0.03;
      }

      const labelTransform = d => {
        const x = (d.x0 + d.x1) / 2 * 180 / Math.PI;
        const y = (d.y0 + d.y1) / 2 * radius;
        return `rotate(${x - 90}) translate(${y},0) rotate(${x < 180 ? 0 : 180})`;
      }

      const clicked = (event, p) => {
          parent.datum(p.parent || root);

          root.each(d => d.target = {
            x0: Math.max(0, Math.min(1, (d.x0 - p.x0) / (p.x1 - p.x0))) * 2 * Math.PI,
            x1: Math.max(0, Math.min(1, (d.x1 - p.x0) / (p.x1 - p.x0))) * 2 * Math.PI,
            y0: Math.max(0, d.y0 - p.depth),
            y1: Math.max(0, d.y1 - p.depth)
          });

          const t = g.transition().duration(750);

          // Transition the data on all arcs, even the ones that aren’t visible,
          // so that if this transition is interrupted, entering arcs will start
          // the next transition from the desired position.
          path.transition(t)
              .tween("data", d => {
                const i = d3.interpolate(d.current, d.target);
                return t => d.current = i(t);
              })
            .filter(function(d) {
              return +this.getAttribute("fill-opacity") || arcVisible(d.target);
            })
              .attr("fill-opacity", d => arcVisible(d.target) ? (d.children ? 0.6 : 0.4) : 0)
              .attr("pointer-events", d => arcVisible(d.target) ? "auto" : "none") 

              .attrTween("d", d => () => arc(d.current));

          label.filter(function(d) {
              return +this.getAttribute("fill-opacity") || labelVisible(d.target);
            }).transition(t)
              .attr("fill-opacity", d => +labelVisible(d.target))
              .attrTween("transform", d => () => labelTransform(d.current));
  }
      // const color = d3.scaleOrdinal(d3.quantize(d3.interpolateRainbow, data.children.length + 1))
      const format = d3.format(",d")
      const width = 600
      const radius = width / 2
      const rwidth = 30
      const arc = d3.arc()
        .startAngle(d => d.x0)
        .endAngle(d => d.x1)
        .padAngle(d => Math.min((d.x1 - d.x0) / 2, 0.005))
        .padRadius(radius * 1.5)
        // .innerRadius(d => { d.y0 * radius})
        .innerRadius(d =>  radius + (d.y0-1)*rwidth)
        // .outerRadius(d => Math.max(d.y0 * radius, d.y1 * radius - 1))
        .outerRadius(d => radius + (d.y1-1)*rwidth)


      root = partition(data);
      root.each(d => d.current = d);
      var svg = d3.select("#pie")
      .append("svg")
      .attr("width", width)
      .attr("height", width)
      .attr("viewBox", [-width/2, -width/2, 2*width, 2*width])
      .style("font", "10px sans-serif");

      const g = svg.append("g")
          .attr("transform", `translate(${width / 2},${width / 2})`);

      const path = g.append("g")
      .selectAll("path")
      .data(root.descendants().slice(1))
      .join("path")
        .attr("fill", d => { 
          // while (d.depth > 1) d = d.parent; 
          return d.data.itemStyle.color; })
        // .attr("fill-opacity", d => arcVisible(d.current) ? (d.children ? 0.6 : 0.4) : 0)
        .attr("fill-opacity", d => d.data.itemStyle.opacity)
        .attr("pointer-events", d => arcVisible(d.current) ? "auto" : "none")
        .attr("d", d => arc(d.current));

        path.filter(d => d.children)
        .style("cursor", "pointer")
        .on("click", clicked);

        path.append("title").text(d => `${d.ancestors().map(d => d.data.name).reverse().join("/")}\n${format(d.value)}`);

        const label = g.append("g")
          .attr("pointer-events", "none")
          .attr("text-anchor", "middle")
          .style("user-select", "none")
        .selectAll("text")
        .data(root.descendants().slice(1))
        .join("text")
          .attr("dy", "0.35em")
          .attr("fill-opacity", d => +labelVisible(d.current))
          .attr("transform", d => labelTransform(d.current))
          // .text(d => d.data.name);

        const parent = g.append("circle")
        .datum(root)
        .attr("r", radius)
        .attr("fill", "none")
        .attr("pointer-events", "all")
        .on("click", clicked);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
</style>
