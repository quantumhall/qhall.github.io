<!DOCTYPE html>
<meta charset="utf-8">
<style>

  .names {
  fill: none;
  stroke: #fff;
  stroke-linejoin: round;
  }

    /* Tooltip CSS */
    .d3-tip {
    line-height: 1.5;
    font-weight: 400;
    font-family:"avenir next", Arial, sans-serif;
    padding: 6px;
    background: rgba(0, 0, 0, 0.6);
    color: #FFA500;
    border-radius: 1px;
    pointer-events: none;
    }

    /* Creates a small triangle extender for the tooltip */
    .d3-tip:after {      
      box-sizing: border-box;
      display: inline;
      font-size: 8px;
      width: 100%;
      line-height: 1.5;
      color: rgba(0, 0, 0, 0.6);
      position: absolute;
      pointer-events: none;
      
    }

    /* Northward tooltips */
    .d3-tip.n:after {
      content: "\25BC";
      margin: -1px 0 0 0;
      top: 100%;
      left: 0;
      text-align: center;
    }

    /* Eastward tooltips */
    .d3-tip.e:after {
      content: "\25C0";
      margin: -4px 0 0 0;
      top: 50%;
      left: -8px;
    }

    /* Southward tooltips */
    .d3-tip.s:after {
      content: "\25B2";
      margin: 0 0 1px 0;
      top: -8px;
      left: 0;
      text-align: center;
    }

    /* Westward tooltips */
    .d3-tip.w:after {
      content: "\25B6";
      margin: -4px 0 0 -1px;
      top: 50%;
      left: 100%;
    }

/*    text{
      pointer-events:none;
    }*/

    .details{
      color:white;
    }

</style>
<body>
<script src="http://d3js.org/d3.v4.min.js"></script>
<script src="http://d3js.org/queue.v1.min.js"></script>
<script src="http://d3js.org/topojson.v1.min.js"></script>
<script src="d3-tip.js"></script>
<script>
var format = d3.format(",");

// Set tooltips
var tip = d3.tip()
            .attr('class', 'd3-tip')
            .offset([-10, 0])
            .html(function(d) {
              return "<strong>Country: </strong><span class='details'>" + d.properties.name + "<br></span>" + "<strong>Population: </strong><span class='details'>" + format(d.population) +"</span>";
            })

var margin = {top: 0, right: 0, bottom: 0, left: 0},
            width = 960 - margin.left - margin.right,
            height = 500 - margin.top - margin.bottom;

var color = d3.scaleThreshold()
    .domain([10000,100000,500000,1000000,5000000,10000000,50000000,100000000,500000000,1500000000])
    .range(["rgb(247,251,255)", "rgb(222,235,247)", "rgb(198,219,239)", "rgb(158,202,225)", "rgb(107,174,214)", "rgb(66,146,198)","rgb(33,113,181)","rgb(8,81,156)","rgb(8,48,107)","rgb(3,19,43)"]);

var path = d3.geoPath();

var svg = d3.select("body")
            .append("svg")
            .attr("width", width)
            .attr("height", height)
            .append('g')
            .attr('class', 'map');

var projection = d3.geoMercator()
                   .scale(130)
                  .translate( [width / 2, height / 1.5]);

var path = d3.geoPath().projection(projection);

svg.call(tip);

queue()
    .defer(d3.json, "world_countries.json")
    .defer(d3.tsv, "world_population.tsv")
    .await(ready);

function ready(error, data, population) {
  var populationById = {};

  population.forEach(function(d) { populationById[d.id] = +d.population; });
  data.features.forEach(function(d) { d.population = populationById[d.id] });

  svg.append("g")
      .attr("class", "countries")
    .selectAll("path")
      .data(data.features)
    .enter().append("path")
      .attr("d", path)
      .style("fill", function(d) { return color(populationById[d.id]); })
      .style('stroke', 'white')
      .style('stroke-width', 1.5)
      .style("opacity",0.8)
      // tooltips
        .style("stroke","white")
        .style('stroke-width', 0.3)
        .on('mouseover',function(d){
          tip.show(d);

          d3.select(this)
            .style("opacity", 1)
            .style("stroke","white")
            .style("stroke-width",3);
        })
        .on('mouseout', function(d){
          tip.hide(d);

          d3.select(this)
            .style("opacity", 0.8)
            .style("stroke","white")
            .style("stroke-width",0.3);
        });

  svg.append("path")
      .datum(topojson.mesh(data.features, function(a, b) { return a.id !== b.id; }))
       // .datum(topojson.mesh(data.features, function(a, b) { return a !== b; }))
      .attr("class", "names")
      .attr("d", path);
}

</script>
</body>
</html>
