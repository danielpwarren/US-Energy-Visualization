<script>
  export let usTotalRenewablePercentage;
</script>

<svg width="1000" height="100">
  <defs>
    <linearGradient id="energyGradient" x1="0%" y1="0%" x2="100%" y2="0%">
      <stop offset="0%" style="stop-color:green; stop-opacity:1" />
      <stop
        offset={usTotalRenewablePercentage + "%"}
        style="stop-color:green; stop-opacity:1"
      />
      <stop
        offset={usTotalRenewablePercentage + "%"}
        style="stop-color:red; stop-opacity:1"
      />
      <stop offset="100%" style="stop-color:red; stop-opacity:1" />
    </linearGradient>
  </defs>

  <text
    class="legend-title"
    fill="currentColor"
    font-weight="bold"
    x="610"
    y="20"
  >
    Total U.S. Energy Mix
  </text>

  <rect fill="url(#energyGradient)" x="610" y="30" height="10" width="270" />

  <line
    x1={610 + 2.7 * usTotalRenewablePercentage}
    y1="30"
    x2={610 + 2.7 * usTotalRenewablePercentage}
    y2="50"
    stroke="white"
    stroke-width="1"
  />

  <text
    fill="currentColor"
    x={600 + 2.7 * usTotalRenewablePercentage}
    y="60"
    text-anchor="end"
  >
    {usTotalRenewablePercentage.toFixed(0)}%
  </text>

  <text fill="currentColor" x={620 + 2.7 * usTotalRenewablePercentage} y="60">
    {(100 - usTotalRenewablePercentage).toFixed(0)}%
  </text>
</svg>
