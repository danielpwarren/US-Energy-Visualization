<script>
  import Choropleth from "./components/Choropleth.svelte";
  import data from "./assets/data.json";
  import { onMount } from "svelte";

  let currentMonthIndex = 0;
  let isAnimating = false;
  let animationInterval;

  const totalMonths = data.length;

  function handleScrollChange(event) {
    currentMonthIndex = parseInt(event.target.value);
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
<div>
  <p>{new Date(data[currentMonthIndex].Year, data[currentMonthIndex].Month - 1).toLocaleString('default', { month: 'long' })}, {data[currentMonthIndex].Year}</p>
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
</div>
<Choropleth data={data[currentMonthIndex].Data} />
