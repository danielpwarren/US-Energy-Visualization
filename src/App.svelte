<script>
  import Choropleth from "./components/Choropleth.svelte";
  import data from "./assets/data.json";
  import { onMount } from "svelte";

  let currentMonthIndex = 0;
  let isAnimating = false;
  let isParagraphVisible = false;
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

  function toggleParagraphVisibility() {
    isParagraphVisible = !isParagraphVisible;
  }

  onMount(() => {
    handleScrollChange({ target: { value: currentMonthIndex } });
  });
</script>

<div class="title">Visualizing the U.S. Transition to Green Energy</div>
<div class="subtitle">
  {new Date(
    data[currentMonthIndex].Year,
    data[currentMonthIndex].Month - 1
  ).toLocaleString("default", { month: "long" })}, {data[currentMonthIndex]
    .Year}
</div>
<div class="controls-container">
  <div class="time-selection">
    Time Selection Slider:<br />
    <input
      type="range"
      min="0"
      max={totalMonths - 1}
      value={currentMonthIndex}
      on:input={handleScrollChange}
    />
  </div>
  <div class="animation-controls">
    Animation Speed: ({animationSpeed}ms)<br />
    <input
      type="range"
      min="50"
      max="500"
      value={animationSpeed}
      on:input={handleSpeedChange}
    />
  </div>
  <div class="button-section">
    <button on:click={toggleAnimation}>
      {#if isAnimating}
        Stop Animation
      {:else}
        Start Animation
      {/if}
    </button>
  </div>
</div>

<Choropleth data={data[currentMonthIndex].Data} />

<div class="button-section">
  <button
    class="details-toggle"
    on:click={toggleParagraphVisibility}
    aria-expanded={isParagraphVisible}
  >
    Writeup: {isParagraphVisible ? "▼" : "▶"}
  </button>
</div>

{#if isParagraphVisible}
  <div class="details">
    <p>
      In Project 3, our group created an animated choropleth comparing the
      renewable and non-renewable energy sources utilized in each US state with
      Svelte and D3. We chose this visual encoding because we found the
      technique used in Lab 6 to be interesting and wanted to implement it on
      our own dataset, with our own twists. By controlling the month selection
      slider, we can watch the renewable and non-renewable energy for each state
      shift based on the month in question, similar to the arrival and departure
      visualization from Lab 6. Using a red-green gradient, we encoded the
      percentage of renewable (green) to non-renewable (red) energy in each
      state.
    </p>
    <p>
      Besides the slider technique learned from Lab 6, our group also utilized
      animation and tooltip techniques to further the viewer's understanding of
      the data. Clicking the 'Start Animation' button begins the visualization
      and displays the trend/transition of renewable and non-renewable energy
      use in the country as a whole and in each state by updating the encoded
      color. When hovering over a state, the tooltip first displays the
      renewable and non-renewable energy percentage, then by pressing the
      'Shift' button on the keyboard, a more detailed report of each energy
      source is displayed.
    </p>
    <p>
      We made the choice to use Svelte over HTML due to Svelte's dynamic update
      ability and integrated package management. This allowed us to write our
      visualization more efficiently by utilizing Svelte to perform a lot of the
      heavy lifting in the background.
    </p>
    <p>
      We chose to work on the energy topic because all group members had worked
      on it for Project 2 and wanted to further research how and where renewable
      energy is used in our nation. We didn't specifically assign tasks for each
      member. Instead, we built upon each other's work.
    </p>
    <p>Here is the breakdown of tasks performed by each member:</p>
    <ul>
      <li>
        Eric: 7-8 hours of work. Finding the dataset. Updating the tooltip data
        and shift button in showing more source information. Properly
        positioning the tooltip to make sure it doesn't cut off from the window.
        Figuring out how to use the shift button in Svelte took quite a bit of
        time as there wasn't much documentation online. Also, trying to set up
        the baseline choropleth map and import it with our dataset took lots of
        time. Big thanks to Daniel for making it happen.
      </li>
      <li>
        Daniel: 18-22 hours of work. I setup and hosted the repo and GitHub
        pages site and added GitHub actions integration to automatically deploy
        on each push. I took inspiration from an observable/D3 <a
          href="https://observablehq.com/@d3/choropleth/2?intent=fork"
          >example choropleth</a
        > showcasing unemployment data per county across the US. I utilized the dataset
        found by Eric and extracted the information needed for the visualization
        using pandas, the notebook I created is linked in the project GitHub. I further
        modified the aforementioned choropleth to fit the data, and prototyped the
        tooltip that Eric helped to complete. I later added the controls linked to
        the animation and month selection, with help from Rukun for the styling and
        formatting. The task that took the most time was most likely getting the
        tooltip to update dynamically during the animation and maintain the expanded
        view (shift functionality) when moving between hovered states.
      </li>
      <li>
        Rukun Zhang: 3-4 hours working time. Inspected the project in general
        and made sure the style was adequate. Suggested a few text, color, and
        animations feature for a better experience with the visualization.
        Revised parts in CSS files and Svelte files which helped better
        understand and comprehend the interpretation of my group members.
      </li>
    </ul>
  </div>
{/if}
