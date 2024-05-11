<script>
  //import * as d3 from 'd3';
  import { scaleQuantize } from "d3-scale";
  import { geoPath } from "d3-geo";
  import { schemeBlues } from "d3-scale-chromatic";
  import * as topojson from "topojson-client";
  import us from "../assets/us.json";
  import Legend from "./Legend.svelte";

  // Receive plot data as prop.
  export let data;

  // Specify the chart’s dimensions.
  const width = 975;
  const height = 610;

  const margin = {
    top: 10,
    right: 10,
    bottom: 20,
    left: 55,
  };

  // Prepare the scale color encoding.
  const colorScale = scaleQuantize([1, 10], schemeBlues[9]);

  // Construct a geographic path generator.
  const path = geoPath();

  // create a lookup for key-value pairs of
  // county -> rate (what we encode with colour).
  const valuemap = new Map(data.map((d) => [d.id, d.rate]));

  // The counties feature collection is all U.S. counties, each with a
  // five-digit FIPS identifier. The statemap lets us lookup the name of
  // the state that contains a given county; a state’s two-digit identifier
  // corresponds to the first two digits of its counties’ identifiers.
  const counties = topojson.feature(us, us.objects.counties).features;
  const states = topojson.feature(us, us.objects.states);
  const statemap = new Map(states.features.map((d) => [d.id, d]));

  // The statemesh is just the internal borders between states, i.e.,
  // everything but the coastlines and country borders. This avoids an
  // additional stroke on the perimeter of the map, which would otherwise
  // mask intricate features such as islands and inlets. (Try removing
  // the last argument to topojson.mesh below to see the effect.)
  const statemesh = topojson.mesh(us, us.objects.states, (a, b) => a !== b);
</script>

<svg
  {width}
  {height}
  viewBox="0 0 {width} {height}"
  style:max-width="100%"
  style:height="auto"
>
  <!-- Loop through all the counties and add the 
			colour encoding according to the data value -->
  <g class="data">
    {#each counties as county}
      <path fill={colorScale(valuemap.get(county.id))} d={path(county)} />
    {/each}
  </g>

  <!-- Add the statemesh aka the white borders between states. -->
  <path
    fill="none"
    stroke="white"
    stroke-linejoin="round"
    d={path(statemesh)}
  />

  <!-- Add the color legend. -->
  <Legend {colorScale} />
</svg>
