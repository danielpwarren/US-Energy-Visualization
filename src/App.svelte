<script>
  import Choropleth from "./components/Choropleth.svelte";
  import data from "./assets/data.json";
  import { onMount } from "svelte";

  let currentMonthIndex = 0;
  let isAnimating = false;
  let animationInterval;

  // Assuming your data spans multiple years and months
  const totalMonths = data.length; // Each record is a month

  function handleScrollChange(event) {
    currentMonthIndex = parseInt(event.target.value);
    // You can also update other components or trigger animations here
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
          toggleAnimation();
        }
      }, 100);
    }
  }

  onMount(() => {
    handleScrollChange({ target: { value: currentMonthIndex } });
  });
</script>

<Choropleth data={data[currentMonthIndex].Data} />

<button on:click={toggleAnimation}>
  {#if isAnimating}
    Stop Animation
  {:else}
    Start Animation
  {/if}
</button>
<input
  type="range"
  min="0"
  max={totalMonths - 1}
  value={currentMonthIndex}
  on:input={handleScrollChange}
/>
