# US Renewable Energy Visualization

## DSC 106 - UCSD

### Authors:

- Daniel Warren
- Eric Gu
- Rukun Zhang

---

[Dataset used](https://www.kaggle.com/datasets/kevinmorgado/us-energy-generation-2001-2022?resource=download)

---

### Write up

In Project 3, our group created an animated choropleth comparing the renewable and non-renewable energy sources utilized in each US state with Svelte and D3. We chose this visual encoding because we found the technique used in Lab 6 to be interesting and wanted to implement it on our own dataset, with our own twists. Controlling the month selection slider we can watch the renewable and non-renewable energy for each state shift based on the month in question, similar to the arrival and departure visualization from Lab 6. Using a red-green gradient we encoded the percentage of renewable (green) to non-renewable (red) energy in each state. Besides the slider technique learned from Lab 6, our group also utilized animation and tooltip techniques to further the viewers understanding of the data. Clicking the ‘Start Animation’ button begins the visualization and displays the trend/transition of renewable and non-renewable energy use in the country as a whole and in each state by updating the encoded color. When hovering over a state the tooltip first displays the renewable and non-renewable energy percentage, then by pressing the ‘Shift’ button on the keyboard a more detailed report of each energy source is displayed.

We made the choice to use Svelte over HTML due to Sveltes dynamic update ability and integrated package management. This allowed us to write our visualization more efficiently by utilizing Svelte to perform a lot of the heavy lifting in the background.

We chose to work on the energy topic because all group members had worked on it for Project 2 and wanted to further research how and where renewable energy is used in our nation. We didn’t specifically assign tasks for each member. Instead, we built up on each other's work.

Here is the breakdown of tasks performed by each member:

- Eric: 7-8 hours of work. Finding the dataset. Updating the tooltip data and shift button in showing more source information. Properly positioning the tooltip to make sure it doesn’t cut off from the window. Figuring out how to use the shift button in Svelte took quite a bit of time as there wasn’t much documentation online. Also, trying to set up the baseline choropleth map and import it with our dataset took lots of time. Big thanks to Daniel for making it happen.

- Daniel: 18-22 hours of work. I setup and hosted the repo and github pages site and added github actions integration to automatically deploy on each push. I took inspiration from an observable/d3 [example choropleth](https://observablehq.com/@d3/choropleth/2?intent=fork) showcasing unemployment data per county across the US. I utilized the dataset found by Eric and extracted the information needed for the visualization using pandas, the notebook I created is linked in the project github. I further modified the aforementioned choropleth to fit the data, and prototyped the tooltip that Eric helped to complete. I later added the controls linked to the animation and month selection, with help from Rukun for the styling and formatting. The task that took the most time was most likely getting the tooltip to update dynamically during the animation and maintain the expanded view (shift functionality) when moving between hovered states.

- Rukun Zhang: 3-4 hours working time. Inspect the project in general and make sure the style is adequate. Suggested a few text, color, and animations feature for a better experience with the visualization. Revise parts in css file and svelte files which helps a better understanding and comprehension of the interpretation of my group members.
