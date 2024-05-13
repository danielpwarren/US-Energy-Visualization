<script>
  import { geoPath } from "d3-geo";
  import { scaleLinear } from "d3-scale";
  import * as topojson from "topojson-client";
  import us from "../assets/us.json";
  import Legend from "./Legend.svelte";
  import { writable } from 'svelte/store';

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
      const HydroelectricConventional = d.EnergyMix['Hydroelectric Conventional'] || 0;
      const Wind = d.EnergyMix['Wind'] || 0;
      const SolarThermalandPhotovoltaic = d.EnergyMix['Solar Thermal and Photovoltaic'] || 0;
      const Geothermal = d.EnergyMix['Geothermal'] || 0;
      const Coal = d.EnergyMix['Coal'] || 0;
      const NaturalGas = d.EnergyMix['Natural Gas'] || 0;
      const Petroleum = d.EnergyMix['Petroleum'] || 0;
      const Other = d.EnergyMix['Other'] || 0;
      const OtherBiomass = d.EnergyMix['Other Biomass'] || 0;
      const OtherGases = d.EnergyMix['Other Gases'] || 0;
      const WoodandWoodDerivedFuels = d.EnergyMix['Wood and Wood Derived Fuels'] || 0;
      const Nuclear = d.EnergyMix['Nuclear'] || 0;

      return {
        id: d.id,
        renewablePercentage: (renewableEnergy / totalEnergy) * 100,
        nonRenewablePercentage: (nonRenewableEnergy / totalEnergy) * 100,
        HydroelectricConventionalPercentage: (HydroelectricConventional / totalEnergy) * 100,
        WindPercentage: (Wind / totalEnergy) * 100,
        SolarThermalandPhotovoltaicPercentage: (SolarThermalandPhotovoltaic / totalEnergy) * 100,
        GeothermalPercentage: (Geothermal / totalEnergy) * 100,
        CoalPercentage: (Coal / totalEnergy) * 100,
        NaturalGasPercentage: (NaturalGas / totalEnergy) * 100,
        PetroleumPercentage: (Petroleum / totalEnergy) * 100,
        OtherPercentage: (Other / totalEnergy) * 100,
        OtherBiomassPercentage: (OtherBiomass / totalEnergy) * 100,
        OtherGasesPercentage: (OtherGases / totalEnergy) * 100,
        WoodandWoodDerivedFuelsPercentage: (WoodandWoodDerivedFuels / totalEnergy) * 100,
        NuclearPercentage: (Nuclear / totalEnergy) * 100
      };
    });

    energyMap = new Map(
      energyPercentages.map((d) => [
        d.id,
        {
          HydroelectricConventional: d.HydroelectricConventionalPercentage,
          renewable: d.renewablePercentage,
          nonRenewable: d.nonRenewablePercentage,
          Wind: d.WindPercentage,
          SolarThermalandPhotovoltaic: d.SolarThermalandPhotovoltaicPercentage,
          Geothermal: d.GeothermalPercentage,
          Coal: d.CoalPercentage,
          NaturalGas: d.NaturalGasPercentage,
          Petroleum: d.PetroleumPercentage,
          Other: d.OtherPercentage,
          OtherBiomass: d.OtherBiomassPercentage,
          OtherGases: d.OtherGasesPercentage,
          WoodandWoodDerivedFuels: d.WoodandWoodDerivedFuelsPercentage,
          Nuclear: d.NuclearPercentage
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
  let shiftPressed = writable(false);

  const colorScale = scaleLinear().domain([0, 100]).range(["red", "green"]);

   window.addEventListener('keydown', (event) => {
    if (event.key === 'Shift') {
      shiftPressed.set(true);
    }
  });

  window.addEventListener('keyup', (event) => {
    if (event.key === 'Shift') {
      shiftPressed.set(false);
    }
  });

  // Respond to shift key state changes
  shiftPressed.subscribe(isPressed => {
    if (currentHoveredState) {
      if (isPressed) {
        const energy = energyMap.get(currentHoveredState.id);
        tooltipText.innerHTML = `
          <strong>${currentHoveredState.properties.name}</strong><br>
          Hydroelectric Conventional: ${energy.HydroelectricConventional.toFixed(1)}%<br>,
          Wind: ${energy.Wind.toFixed(1)}%<br>,
          Solar ThermalandPhotovoltaic: ${energy.SolarThermalandPhotovoltaic.toFixed(1)}%<br>,
          Geothermal: ${energy.Geothermal.toFixed(1)}%<br>,
          Coal: ${energy.Coal.toFixed(1)}%<br>,
          Natural Gas: ${energy.NaturalGas.toFixed(1)}%<br>,
          Petroleum: ${energy.Petroleum.toFixed(1)}%<br>,
          Other: ${energy.Other.toFixed(1)}%<br>,
          Other Biomass: ${energy.OtherBiomass.toFixed(1)}%<br>,
          Other Gases: ${energy.OtherGases.toFixed(1)}%<br>,
          Wood and Wood Derived Fuels: ${energy.WoodandWoodDerivedFuels.toFixed(1)}%<br>,
          Nuclear: ${energy.Nuclear.toFixed(1)}%<br>
        `;} else {
        updateTooltip();
      }
    }
  });

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