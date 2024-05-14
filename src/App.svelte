<script>
  import Choropleth from "./components/Choropleth.svelte";
  import data from "./assets/data.json";
  import { onMount } from "svelte";

  let currentMonthIndex = 0;
  let isAnimating = false;
  let animationInterval;
  let animationSpeed = 250;

  const totalMonths = data.length;

  function handleScrollChange(event) {
    currentMonthIndex = parseInt(event.target.value);
  }

  function handleSpeedChange(event) {
    animationSpeed = parseInt(event.target.value);
    if (isAnimating) {
      toggleAnimation(); // Restart animation with new speed
      toggleAnimation();
    }
  }

  function toggleAnimation() {
    if (isAnimating) {
      clearInterval(animationInterval);
      isAnimating = false;
    } else {
      isAnimating = true;
      animationInterval = setInterval(() => {
        currentMonthIndex = (currentMonthIndex + 1) % totalMonths;
        if (currentMonthIndex === totalMonths - 1) {
          toggleAnimation(); // Stop at the end of the cycle
        }
      }, animationSpeed);
    }
  }

  onMount(() => {
    handleScrollChange({ target: { value: currentMonthIndex } });
  });
</script>

<div class="controls">
  <p>
    {new Date(
      data[currentMonthIndex].Year,
      data[currentMonthIndex].Month - 1
    ).toLocaleString("default", { month: "long" })}, {data[currentMonthIndex]
      .Year}
  </p>
  <input
    type="range"
    min="0"
    max={totalMonths - 1}
    value={currentMonthIndex}
    on:input={handleScrollChange}
  />
  Animation Speed:
  <input
    type="range"
    min="50"
    max="500"
    value={animationSpeed}
    on:input={handleSpeedChange}
  />
  <button on:click={toggleAnimation}>
    {#if isAnimating}
      Stop Animation
    {:else}
      Start Animation
    {/if}
  </button>
</div>
<Choropleth data={data[currentMonthIndex].Data} />
