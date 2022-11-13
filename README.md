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
