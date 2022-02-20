---
layout: page
title: d3 Test
permalink: /d3test/
---
<style>

.links line {
  stroke: #999;
  stroke-opacity: 0.6;
}

.nodes circle {
  stroke: #fff;
  stroke-width: 1.5px;
}
</style>

<script src="https://d3js.org/d3.v5.min.js"></script>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
<svg id="d3viz"></svg>
<script>
    {
        let width = 1000;
        let height = 800;

        let svg = d3.select("#d3viz")
            .attr('width', width)
            .attr('height', height);
	console.log(svg);
        let color = d3.scaleOrdinal(d3.schemePaired);

        // Here we create our simulation, and give it some forces to apply 
        //  to all the nodes:
        let simulation = d3.forceSimulation()
            // forceLink creates tension along each link, keeping connected nodes together
            .force("link", d3.forceLink().id(d => d.id))
            // forceManyBody creates a repulsive force between nodes, 
            //  keeping them away from each other
            .force("charge", d3.forceManyBody().strength(-5))
            // forceCenter acts like gravity, keeping the whole visualization in the 
            //  middle of the screen
            .force("center", d3.forceCenter(width / 2, height / 2))
	    .force("radial", d3.forceRadial(300, width /2, height / 2));

        // This part triggers an asynchronous call to go grab the data in another file...file places in _site folder?
        //  stuff inside this fuction might not actually happen for a while!
        d3.json("/data.json").then( graph => {
	    console.log(graph);

            // First we create the links in their own group that comes before the node 
            //  group (so the circles will always be on top of the lines)
            let linkLayer = svg.append("g")
                .attr("class", "links");
            // Now let's create the lines
            let links = linkLayer.selectAll("line")
                .data(graph.links)
                .enter().append("line")
                .attr("stroke-width", d => Math.sqrt(d.value));

            // Now we create the node group, and the nodes inside it
            let texts = svg.selectAll(null).data(graph.nodes)
            .enter()
            .append('text')
            .text(d => d.entity)
            .attr('color', 'black')
            .attr('font-size', 15)
            
            let nodeLayer = svg.append("g")
                .attr("class", "nodes");
            
            let nodes = nodeLayer
                .selectAll("circle")
                .data(graph.nodes)
                .enter().append("circle")
                .attr("r", 5) 
                //.attr("fill", d => color(d.group))
                // This part adds event listeners to each of the nodes; when you click,
                //  move, and release the mouse on a node, each of these functions gets 
                //  called (we've defined them at the end of the file)
                .call(d3.drag()
                    .on("start", dragstarted)
                    .on("drag", dragged)
                    .on("end", dragended));

            // We can add a tooltip to each node, so when you hover over a circle, you 
            //  see the node's id
            nodes.append("title")
                .text( d => d.id );

            // Now that we have the data, let's give it to the simulation...
            simulation.nodes(graph.nodes);
            // The tension force (the forceLink that we named "link" above) also needs
            //  to know about the link data that we finally have - we couldn't give it 
            //  earlier, because it hadn't been loaded yet!
            simulation.force("link")
                .links(graph.links);

            // Finally, let's tell the simulation how to update the graphics
            simulation.on("tick", function () {
                // Every "tick" of the simulation will create / update each node's 
                //  coordinates; we need to use those coordinates to move the lines
                //  and circles into place
                links
                    .attr("x1", function (d) {
                        return d.source.x;
                    })
                    .attr("y1", function (d) {
                        return d.source.y;
                    })
                    .attr("x2", function (d) {
                        return d.target.x;
                    })
                    .attr("y2", function (d) {
                        return d.target.y;
                    });

                nodes
                    .attr("cx", function (d) {
                        return d.x;
                    })
                    .attr("cy", function (d) {
                        return d.y;
                    });
                texts.attr('x', (data) => {
                return data.x
                })
                .attr('y', (data) => {
                return data.y
                });

            });
        }).catch(error => console.log(error));

        function dragstarted(d) {
            if (!d3.event.active) simulation.alphaTarget(0.3).restart();
            d.fx = d.x;
            d.fy = d.y;
        }

        function dragged(d) {
            d.fx = d3.event.x;
            d.fy = d3.event.y;
        }

        function dragended(d) {
            if (!d3.event.active) simulation.alphaTarget(0);
            d.fx = null;
            d.fy = null;
        }
    }
</script>
[jekyll-organization]: https://github.com/jekyll
