# Data Visualization task 1

This exercise consist on the visualization of covid cases in differents areas of Spain. In each area a circle is going to appear and change its size depending on the cases of this region, so the larger a circle is, higher number of cases.
For this purpose we are going to use JavaScript library called D3js.

# Initialization 
First of all we are going to install all te packages needed to the application to work, including other extra packages as _topojson_ for working with maps and _composite projections_ for displaying Canary Islands below Spain

```bash
npm install
npm install topojson-client --save
npm install @types/topojson-client --save-dev
``` 

# Import modules
Once all packages are installed it is time to import data needed to print the map and all community locations

```bash
import * as d3 from "d3";
import { interpolateMagma } from "d3";
import { on } from "node:events";
import * as topojson from "topojson-client";
const spainjson = require("./spain.json");
const d3Composite = require("d3-composite-projections");
import { latLongCommunities } from "./communities";

import { statsIni, statsLast, ResultEntry } from "./covid";
```

# Functions
The first function we are goint to define is _maxAffected_. The function gets an array of dictionaries whose keys are name
(community name) and value (cases), and returns the number of cases of the most affected community. 

```bash
const maxAffected = (s: ResultEntry[]) => {
  const max = s.reduce((max, item) => (item.value > max ? item.value : max), 0);
  return max;
};
```

The next function calculates the radius of the circle to be plotted on the map following a linear growth between 0 and
the most affected community (previous function)
```bash
const calculateRadiusBasedOnAffectedCases = (
  comunidad: string,
  s: ResultEntry[]
) => {
  const entry = s.find((item) => item.name === comunidad);

  const affectedRadiusScale = d3
    .scaleLinear()
    .domain([0, maxAffected(s)])
    .range([0, 50]);

  return entry ? affectedRadiusScale(entry.value) : 0;
};
```

It is time to set Spain map configuration, size and translation
```bash
const aProjection = d3Composite
  .geoConicConformalSpain()
  .scale(3300)
  .translate([500, 400]);
const geoPath = d3.geoPath().projection(aProjection);

const geojson = topojson.feature(spainjson, spainjson.objects.ESP_adm1);
```

Now we are going to render the map and set background color

```bash
const svg = d3
  .select("body")
  .append("svg")
  .attr("width", 1024)
  .attr("height", 800)
  .attr("style", "background-color: #FBFAF0");

svg
  .selectAll("path")
  .data(geojson["features"])
  .enter()
  .append("path")
  .attr("class", "country")
  .attr("d", geoPath as any);
```

We can then print the information of the initial state in the map
```bash
svg
  .selectAll("circle")
  .data(latLongCommunities)
  .enter()
  .append("circle")
  .attr("class", "selected-country")
  .attr("r", (d) => calculateRadiusBasedOnAffectedCases(d.name, statsIni))
  .attr("cx", (d) => aProjection([d.long, d.lat])[0])
  .attr("cy", (d) => aProjection([d.long, d.lat])[1]);
```

And define a function with which we are going to update the circles in the map depending on the number cases
```bash
const updateRadius = (data: ResultEntry[]) => {
  d3.selectAll("circle")
    .data(latLongCommunities)
    .transition()
    .duration(500)
    .attr("class", "selected-country")
    .attr("r", (d) => calculateRadiusBasedOnAffectedCases(d.name, data))
    .attr("cx", (d) => aProjection([d.long, d.lat])[0])
    .attr("cy", (d) => aProjection([d.long, d.lat])[1])
};
```
To change the circles in the map we are going to use two different buttons. When we click a button an event is received
and the previous function is going to be applied. 

```bash
document
  .getElementById("Inicio")
  .addEventListener("click", function handlResultsIni() {
    updateRadius(statsIni);
  });

document
  .getElementById("Final")
  .addEventListener("click", function handlResultsFinal() {
    updateRadius(statsLast);
  });
```

# HTML code
We need to set the html code where we are going to put all the elements defined previously, from map to
buttons
```bash
<html>
  <head>
    <link rel="stylesheet" type="text/css" href="./map.css" />
  </head>
  <div style="width: 1024px;">
    <button class="button" id="Inicio">Initial cases</button>
    <button style="align-self: flex-end;" class="button" id="Final">Last 14 days cases</button>
  </div>
  <body>
    <script src="./index.ts"></script>
  </body>
</html>
```

# CSS styles
Having the html code we can set style y visual properties to all the elements in it
```bash
.country {
  stroke-width: 1;
  stroke: #2f4858;
  fill: #008c86;
}

.selected-country {
  stroke-width: 1;
  stroke: #bc5b40;
  fill: #f88f70;
  fill-opacity: 0.7;
}

.button {
  background-color: #f4511e;
  border: none;
  color: white;
  padding: 16px 32px;
  text-align: center;
  font-size: 16px;
  margin: 4px 2px;
  opacity: 0.6;
  transition: 0.3s;
  display: inline-block;
  text-decoration: none;
  cursor: pointer;
}

.button:hover {
  opacity: 1;
  box-shadow: 0 12px 16px 0 rgba(0,0,0,0.24);
}

.button:focus {opacity: 1;}
```
