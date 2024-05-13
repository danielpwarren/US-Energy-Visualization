<script>
  import { geoPath } from "d3-geo";
  import { scaleLinear } from "d3-scale";
  import * as topojson from "topojson-client";
  import us from "../assets/us.json";
  import Legend from "./Legend.svelte";

  export let data;

  const width = 975;
  const height = 610;

  const path = geoPath();

  let tooltipElement, tooltipText;

  const states = topojson.feature(us, us.objects.states);
  const states_features = states.features;

  const statemesh = topojson.mesh(us, us.objects.states);

  const renewableSources = [
    "Hydroelectric Conventional",
    "Wind",
    "Solar Thermal and Photovoltaic",
    "Geothermal",
  ];
  const nonRenewableSources = [
    "Coal",
    "Natural Gas",
    "Petroleum",
    "Other",
    "Other Biomass",
    "Other Gases",
    "Wood and Wood Derived Fuels",
    "Nuclear",
  ];

  let energyMap, usTotalRenewablePercentage;
  
  let currentHoveredState = null;
  $: {
    const energyPercentages = data.map((d) => {
      const totalEnergy = Object.values(d.EnergyMix).reduce(
        (acc, cur) => acc + cur,
        0
      );
      const renewableEnergy = renewableSources.reduce(
        (acc, source) => acc + (d.EnergyMix[source] || 0),
        0
      );
      const nonRenewableEnergy = nonRenewableSources.reduce(
        (acc, source) => acc + (d.EnergyMix[source] || 0),
        0
      );
      return {
        id: d.id,
        renewablePercentage: (renewableEnergy / totalEnergy) * 100,
        nonRenewablePercentage: (nonRenewableEnergy / totalEnergy) * 100,
      };
    });

    energyMap = new Map(
      energyPercentages.map((d) => [
        d.id,
        {
          renewable: d.renewablePercentage,
          nonRenewable: d.nonRenewablePercentage,
        },
      ])
    );

    usTotalRenewablePercentage = energyMap.get("00")
      ? energyMap.get("00").renewable
      : 0;

    if (currentHoveredState && energyMap) {
      updateTooltip();
    }
  }

  const colorScale = scaleLinear().domain([0, 100]).range(["red", "green"]);

  const showTooltip = (event, state) => {
    currentHoveredState = state;
    updateTooltip();
    tooltipElement.style.visibility = "visible";
    tooltipElement.style.left = `${event.pageX + 10}px`;
    tooltipElement.style.top = `${event.pageY + 10}px`;
  };

  function updateTooltip() {
    const energy = energyMap.get(currentHoveredState.id);
    tooltipText.innerHTML = `
      <strong>${currentHoveredState.properties.name}</strong><br>
      Renewable: ${energy.renewable.toFixed(1)}%<br>
      Non-Renewable: ${energy.nonRenewable.toFixed(1)}%
    `;
  }

  const hideTooltip = () => {
    tooltipElement.style.visibility = "hidden";
  };
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
    {#each states_features as state}
      <path
        fill={colorScale(energyMap.get(state.id).renewable || 0)}
        d={path(state)}
        role="button"
        tabindex="0"
        on:mouseover={(event) => showTooltip(event, state)}
        on:mouseout={hideTooltip}
        on:focus={(event) => showTooltip(event, state)}
        on:blur={hideTooltip}
      />
    {/each}
  </g>

  <!-- Add the statemesh aka the white borders between states. -->
  <path
    fill="none"
    stroke="white"
    stroke-linejoin="round"
    d={path(statemesh)}
  />

  <Legend {usTotalRenewablePercentage} />
</svg>
<div bind:this={tooltipElement} class="tooltip">
  <span bind:this={tooltipText}></span>
</div>
