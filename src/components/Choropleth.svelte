<script>
  import { geoPath } from "d3-geo";
  import { scaleLinear } from "d3-scale";
  import * as topojson from "topojson-client";
  import us from "../assets/us.json";
  import TotalPowerUsage from "./TotalPowerUsage.svelte";
  import { writable } from "svelte/store";

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

  const energySourceInfo = {
    HydroelectricConventional: {
      name: "Hydroelectric",
      renewable: "renewable",
    },
    Wind: { name: "Wind Power", renewable: "renewable" },
    SolarThermalandPhotovoltaic: {
      name: "Solar Energy",
      renewable: "renewable",
    },
    Geothermal: { name: "Geothermal", renewable: "renewable" },
    Coal: { name: "Coal", renewable: "non-renewable" },
    NaturalGas: { name: "Natural Gas", renewable: "non-renewable" },
    Petroleum: { name: "Petroleum", renewable: "non-renewable" },
    Other: { name: "Other Sources", renewable: "non-renewable" },
    OtherBiomass: { name: "Biomass", renewable: "non-renewable" },
    OtherGases: { name: "Other Gases", renewable: "non-renewable" },
    WoodandWoodDerivedFuels: { name: "Wood Fuels", renewable: "non-renewable" },
    Nuclear: { name: "Nuclear Energy", renewable: "non-renewable" },
  };

  let energyMap, usTotalRenewablePercentage;

  let currentHoveredState = null;
  let shiftPressed = writable(false);

  const colorScale = scaleLinear().domain([0, 100]).range(["red", "green"]);

  $: $shiftPressed, updateTooltip();
  $: currentHoveredState, updateTooltip();

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
      const HydroelectricConventional =
        d.EnergyMix["Hydroelectric Conventional"] || 0;
      const Wind = d.EnergyMix["Wind"] || 0;
      const SolarThermalandPhotovoltaic =
        d.EnergyMix["Solar Thermal and Photovoltaic"] || 0;
      const Geothermal = d.EnergyMix["Geothermal"] || 0;
      const Coal = d.EnergyMix["Coal"] || 0;
      const NaturalGas = d.EnergyMix["Natural Gas"] || 0;
      const Petroleum = d.EnergyMix["Petroleum"] || 0;
      const Other = d.EnergyMix["Other"] || 0;
      const OtherBiomass = d.EnergyMix["Other Biomass"] || 0;
      const OtherGases = d.EnergyMix["Other Gases"] || 0;
      const WoodandWoodDerivedFuels =
        d.EnergyMix["Wood and Wood Derived Fuels"] || 0;
      const Nuclear = d.EnergyMix["Nuclear"] || 0;

      return {
        id: d.id,
        renewablePercentage: (renewableEnergy / totalEnergy) * 100,
        nonRenewablePercentage: (nonRenewableEnergy / totalEnergy) * 100,
        HydroelectricConventionalPercentage:
          (HydroelectricConventional / totalEnergy) * 100,
        WindPercentage: (Wind / totalEnergy) * 100,
        SolarThermalandPhotovoltaicPercentage:
          (SolarThermalandPhotovoltaic / totalEnergy) * 100,
        GeothermalPercentage: (Geothermal / totalEnergy) * 100,
        CoalPercentage: (Coal / totalEnergy) * 100,
        NaturalGasPercentage: (NaturalGas / totalEnergy) * 100,
        PetroleumPercentage: (Petroleum / totalEnergy) * 100,
        OtherPercentage: (Other / totalEnergy) * 100,
        OtherBiomassPercentage: (OtherBiomass / totalEnergy) * 100,
        OtherGasesPercentage: (OtherGases / totalEnergy) * 100,
        WoodandWoodDerivedFuelsPercentage:
          (WoodandWoodDerivedFuels / totalEnergy) * 100,
        NuclearPercentage: (Nuclear / totalEnergy) * 100,
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
          Nuclear: d.NuclearPercentage,
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

  function updateTooltip() {
    if (currentHoveredState && energyMap.has(currentHoveredState.id)) {
      const energy = energyMap.get(currentHoveredState.id);
      let htmlContent = `<strong>${currentHoveredState.properties.name}</strong><br>
      <div class='stat-container'>`;
      if ($shiftPressed) {
        Object.keys(energy).forEach((key) => {
          if (energy[key] > 0 && !["renewable", "nonRenewable"].includes(key)) {
            const info = energySourceInfo[key];
            htmlContent += `<div class='energy-stat'>
                              <span class='label ${info.renewable}'>${info.name}</span>
                              <span class='percent'>${energy[key].toFixed(1)}%</span>
                            </div>
                            </div>`;
          }
        });
      } else {
        htmlContent += `
          <div class='energy-stat'>
            <span class='label renewable'>Renewable: </span>
            <span class='percent'>${energy.renewable.toFixed(1)}%</span>
          </div>
          <div class='energy-stat'>
            <span class='label non-renewable'>Non-Renewable:</span>
            <span class='percent'>${energy.nonRenewable.toFixed(1)}%</span>
          </div>
          </div>
          <div class="tooltip-footer tooltip-footer-animate">
            Hold Shift for more details.
          </div>`;
      }
      tooltipText.innerHTML = htmlContent;
    }
  }

  const showTooltip = (event, state) => {
    currentHoveredState = state;
    tooltipElement.style.visibility = "visible";
    tooltipElement.style.left = `${event.pageX + 10}px`;
    tooltipElement.style.top = `${event.pageY + 10}px`;
  };

  const hideTooltip = () => {
    tooltipElement.style.visibility = "hidden";
  };

  window.addEventListener("keydown", (event) => {
    if (event.key === "Shift") {
      shiftPressed.set(true);
    }
  });

  window.addEventListener("keyup", (event) => {
    if (event.key === "Shift") {
      shiftPressed.set(false);
    }
  });

  shiftPressed.subscribe((isPressed) => {
    if (currentHoveredState) {
      updateTooltip();
    }
  });
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

  <TotalPowerUsage {usTotalRenewablePercentage} />
</svg>
<div bind:this={tooltipElement} class="tooltip">
  <span bind:this={tooltipText}></span>
</div>
