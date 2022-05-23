<template>
    <div id="test">
    </div>
</template>

<script>
export default {
    name: 'HelloWorld',
    props: {
        msg: String
    },
    data() {
        return {
            svg:null
        }
    },
    mounted() {
        const width = 600
        const d3 = window.d3
        this.svg = d3.select("#test")
                .append("svg")
                .attr("width", width)
                .attr("height", width)
                .attr("viewBox", [-width / 2, -width / 2, 2 * width, 2 * width])
                .style("font", "10px sans-serif");
        this.addPie()
        this.addEdge()
    },
    methods: {
        async addPie() {
            let colors = ['#a87b64', '#e65832', '#da1d23', '#e1c315', '#0aa3b5', '#3aa255']
            let arr = colors.reduce((res, item) => {
                let obj = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15].map(() => {
                    let temp = item == '#3aa255' ? Math.random() * 0.6 + 0.4 : Math.random() * 0.3
                    return {
                        name: 'depth1',
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
                    }
                })
                res.push(...obj)
                return res
            }, [])
            let data = {
                name: 'flare',
                children: arr
            }
            this.init(data)
        },
        init(data) {
            let root = ''
            const d3 = window.d3
            const partition = data => {
                const root = d3.hierarchy(data).sum(d => d.value).sort((a, b) => b.value - a.value);
                return d3.partition().size([2 * Math.PI, root.height + 1])(root);
            }
            const arcVisible = d => {
                return d.y1 <= 3 && d.y0 >= 1 && d.x1 > d.x0;
            }
            const labelVisible = d => {
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
                    .filter(function (d) {
                        return +this.getAttribute("fill-opacity") || arcVisible(d.target);
                    })
                    .attr("fill-opacity", d => arcVisible(d.target) ? (d.children ? 0.6 : 0.4) : 0)
                    .attr("pointer-events", d => arcVisible(d.target) ? "auto" : "none")

                    .attrTween("d", d => () => arc(d.current));

                label.filter(function (d) {
                    return +this.getAttribute("fill-opacity") || labelVisible(d.target);
                }).transition(t)
                    .attr("fill-opacity", d => +labelVisible(d.target))
                    .attrTween("transform", d => () => labelTransform(d.current));
            }
            // const color = d3.scaleOrdinal(d3.quantize(d3.interpolateRainbow, data.children.length + 1))
            const format = d3.format(",d")
            const width = 600
            const radius = width / 2
            const rwidth = 25
            const mappingW = [0,1,1.9,3.2,5]
            const arc = d3.arc()
                .startAngle(d => d.x0)
                .endAngle(d => d.x1)
                .padAngle(d => Math.min((d.x1 - d.x0) / 2, 0.005))
                .padRadius(radius * 1.5)
                // .innerRadius(d => { d.y0 * radius})
                .innerRadius(d => {
                    return radius + mappingW[d.y0] * rwidth
                })
                // .outerRadius(d => Math.max(d.y0 * radius, d.y1 * radius - 1))
                .outerRadius(d => {
                    return radius + mappingW[d.y1]  * rwidth
                })


            root = partition(data);
            root.each(d => d.current = d);
            const svg = this.svg
            const g = svg.append("g")
                .attr("transform", `translate(${width / 2},${width / 2})`);

            const path = g.append("g")
                .selectAll("path")
                .data(root.descendants().slice(1))
                .join("path")
                .attr("fill", d => {
                    // while (d.depth > 1) d = d.parent; 
                    return d.data.itemStyle.color;
                })
                // .attr("fill-opacity", d => arcVisible(d.current) ? (d.children ? 0.6 : 0.4) : 0)
                .attr("fill-opacity", d => d.data.itemStyle.opacity)
                .attr("pointer-events", d => arcVisible(d.current) ? "auto" : "none")
                .attr("stroke","#999")
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
        },
        async addEdge() {

            
            const point = (that, x, y) => {
                that._context.bezierCurveTo(
                    (2 * that._x0 + that._x1) / 3,
                    (2 * that._y0 + that._y1) / 3,
                    (that._x0 + 2 * that._x1) / 3,
                    (that._y0 + 2 * that._y1) / 3,
                    (that._x0 + 4 * that._x1 + x) / 6,
                    (that._y0 + 4 * that._y1 + y) / 6
                );
                console.log(that,x,y)
                // that._context.quadraticCurveTo(
                //     (that._x0 + 2 * that._x1) / 3,
                //     (that._y0 + 2 * that._y1) / 3,
                //     (that._x0 + 4 * that._x1 + x) / 6,
                //     (that._y0 + 4 * that._y1 + y) / 6
                // )
            }

            function QuadraticCurve(context) {
                this._context = context;
            }

            QuadraticCurve.prototype = {
                areaStart: function() {
                    this._line = 0;
                },
                areaEnd: function() {
                    this._line = NaN;
                },
                lineStart: function() {
                    this._x0 = this._x1 =
                    this._y0 = this._y1 = NaN;
                    this._point = 0;
                },
                lineEnd: function() {
                    console.log(this._point)
                    switch (this._point) {
                    case 3: point(this, this._x1, this._y1); // falls through
                    case 2: this._context.lineTo(this._x1, this._y1); break;
                    }
                    if (this._line || (this._line !== 0 && this._point === 1)) this._context.closePath();
                    this._line = 1 - this._line;
                },
                point: function(x, y) {
                    x = +x, y = +y;
                    switch (this._point) {
                    case 0: this._point = 1; this._line ? this._context.lineTo(x, y) : this._context.moveTo(x, y); break;
                    case 1: this._point = 2; break;
                    case 2: this._point = 3; this._context.lineTo((5 * this._x0 + this._x1) / 6, (5 * this._y0 + this._y1) / 6); // falls through
                    default: point(this, x, y); break;
                    }
                    this._x0 = this._x1, this._x1 = x;
                    this._y0 = this._y1, this._y1 = y;
                }
            };

            const quadraticCurve = function(context) {
                return new QuadraticCurve(context, 0.5);
            };

            // var curve = Step 

            let d3 = window.d3
            let diameter = 880,
                radius = diameter / 2,
                innerRadius = radius - 120;

            var cluster = d3.cluster()
                .size([360, innerRadius]);

            var line = d3.radialLine()
                .curve(quadraticCurve)
                .radius(function (d) { return d.y; })
                .angle(function (d) { return d.x / 180 * Math.PI; });

            const svg = this.svg
            // var svg = d3.select("#test").append("svg")
            //     .attr("width", diameter)
            //     .attr("height", diameter)
                .append("g")
                .attr("transform", "translate(" + 600/2 + "," + 600/2 + ")");

            var link = svg.append("g").selectAll(".link"),
                node = svg.append("g").selectAll(".node");

            let classes = await d3.json("./flare.json")

            var root = packageHierarchy(classes)
                .sum(function (d) { return d.size; });

            cluster(root);

            link
                .data(packageImports(root.leaves()))
                .enter().append("path")
                .each(function (d) { d.source = d[0], d.target = d[d.length - 1]; })
                .attr("class", "link")
                .attr("d", line);

            node
                .data(root.leaves())
                .enter().append("text")
                .attr("class", "node")
                .attr("dy", "0.31em")
                .attr("transform", function (d) { return "rotate(" + (d.x - 90) + ")translate(" + (d.y + 8) + ",0)" + (d.x < 180 ? "" : "rotate(180)"); })
                .attr("text-anchor", function (d) { return d.x < 180 ? "start" : "end"; })
            //   .text(function(d) { return d.data.key; });

            // Lazily construct the package hierarchy from class names.
            function packageHierarchy(classes) {
                var map = {};

                function find(name, data) {
                    var node = map[name], i;
                    if (!node) {
                        node = map[name] = data || { name: name, children: [] };
                        if (name.length) {
                            node.parent = find(name.substring(0, i = name.lastIndexOf(".")));
                            node.parent.children.push(node)
                            node.key = name.substring(i + 1);
                        }
                    }
                    return node;
                }

                classes.forEach(function (d) {
                    find(d.name, d);
                });

                return d3.hierarchy(map[""]);
            }

            // Return a list of imports for the given array of nodes.
            function packageImports(nodes) {
                var map = {},
                    imports = [];

                // Compute a map from name to node.
                nodes.forEach(function (d) {
                    map[d.data.name] = d;
                });

                // For each import, construct a link from the source to target node.
                nodes.forEach(function (d) {
                    if (d.data.imports) d.data.imports.forEach(function (i) {
                        Math.random()>0.95? imports.push(map[d.data.name].path(map[i])):''
                    });
                });

                return imports;
            }
        }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
</style>
