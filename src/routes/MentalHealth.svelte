
<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    let data = [];
    let filteredData = [];
    let selectedWorkLocation = "All"; // Default filter value for work location
    let selectedStressLevel = "All"; // Default filter value for stress level
    const margin = { top: 60, right: 150, bottom: 50, left: 70 };
    const width = 800 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;

    // Work location options in desired order
    const workLocations = ["Onsite", "Hybrid", "Remote"];
    const stressLevels = ["All", "Low", "Medium", "High"]; // Predefined stress levels

    onMount(async () => {
        // Fetch and process the data
        data = await d3.csv(
            'https://raw.githubusercontent.com/yix020/Final-Project/refs/heads/main/static/cleaned_data%20-%20cleaned_data%20copy.csv',
            d => ({
                Stress_Level: d['Stress_Level'],
                Work_Location: d['Work_Location'],
            })
        );

        filteredData = data; // Initialize filtered data
        drawChart();
    });

    function updateFilter() {
        // Filter data by work location and stress level
        filteredData = data.filter(d =>
            (selectedWorkLocation === "All" || d.Work_Location === selectedWorkLocation) &&
            (selectedStressLevel === "All" || d.Stress_Level === selectedStressLevel)
        );
        drawChart(); // Redraw chart with filtered data
    }

    function drawChart() {
        // Clear the existing chart
        d3.select("#chart").selectAll("*").remove();

        // Group data by Work_Location and Stress_Level
        const groupedData = d3.rollups(
            filteredData,
            v => v.length,
            d => d.Work_Location,
            d => d.Stress_Level
        );

        // Flatten grouped data into an array for easy plotting
        const flattenedData = workLocations.flatMap(workLocation =>
            stressLevels.filter(level => level !== "All").map(stressLevel => ({
                workLocation,
                stressLevel,
                count: groupedData.find(d => d[0] === workLocation)?.[1]?.find(d => d[0] === stressLevel)?.[1] || 0
            }))
        );

        // Scales
        const x0 = d3.scaleBand()
            .domain(workLocations)
            .range([0, width])
            .padding(0.2);

        const x1 = d3.scaleBand()
            .domain(stressLevels.filter(level => level !== "All")) // Exclude "All" from the chart
            .range([0, x0.bandwidth()])
            .padding(0.1);

        const y = d3.scaleLinear()
            .domain([0, d3.max(flattenedData, d => d.count)])
            .nice()
            .range([height, 0]);

        const color = d3.scaleOrdinal()
            .domain(stressLevels.filter(level => level !== "All")) // Exclude "All" from colors
            .range(d3.schemeTableau10);

        // Create the SVG container
        const svg = d3.select("#chart")
            .append("svg")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom);

        // Add the title
        svg.append("text")
            .attr("x", (width + margin.left + margin.right) / 2)
            .attr("y", margin.top / 2)
            .attr("text-anchor", "middle")
            .style("font-size", "18px")
            .style("font-weight", "bold")
            
        const plotGroup = svg
            .append("g")
            .attr("transform", `translate(${margin.left},${margin.top})`);

        // Add X axis
        plotGroup.append("g")
            .call(d3.axisBottom(x0))
            .attr("transform", `translate(0,${height})`)
            .selectAll("text")
            .attr("transform", "translate(0,10)")
            .style("text-anchor", "middle");

        // Add Y axis
        plotGroup.append("g")
            .call(d3.axisLeft(y));

        // Add Y-axis label
        plotGroup.append("text")
            .attr("transform", "rotate(-90)")
            .attr("y", -margin.left + 20)
            .attr("x", -height / 2)
            .attr("dy", "1em")
            .style("text-anchor", "middle")
            .style("font-size", "12px")
            .text("Number of Employees");

        // Create tooltip
        const tooltip = d3.select("#chart")
            .append("div")
            .style("position", "absolute")
            .style("background", "#fff")
            .style("border", "1px solid #ccc")
            .style("padding", "8px")
            .style("border-radius", "4px")
            .style("box-shadow", "0px 2px 6px rgba(0,0,0,0.1)")
            .style("visibility", "hidden")
            .style("pointer-events", "none")
            .style("font-size", "12px");

        // Add grouped bars with tooltip interaction
        plotGroup.append("g")
            .selectAll("g")
            .data(workLocations) // Group by work location
            .enter()
            .append("g")
            .attr("transform", d => `translate(${x0(d)},0)`) // Position groups
            .selectAll("rect")
            .data(workLocation => stressLevels.filter(level => level !== "All").map(level => ({
                level,
                workLocation,
                count: flattenedData.find(
                    d => d.workLocation === workLocation && d.stressLevel === level
                )?.count || 0
            })))
            .enter()
            .append("rect")
            .attr("x", d => x1(d.level)) // Position within group
            .attr("y", d => y(d.count))
            .attr("width", x1.bandwidth())
            .attr("height", d => height - y(d.count))
            .attr("fill", d => color(d.level))
            .on("mouseover", (event, d) => {
                tooltip
                    .style("visibility", "visible")
                    .style("top", `${event.pageY - 20}px`)
                    .style("left", `${event.pageX + 20}px`)
                    .html(`
                        <strong>Stress Level:</strong> ${d.level}<br>
                        <strong>Work Location:</strong> ${d.workLocation}<br>
                        <strong>Employees:</strong> ${d.count}
                    `);
            })
            .on("mousemove", (event) => {
                tooltip
                    .style("top", `${event.pageY - 20}px`)
                    .style("left", `${event.pageX + 20}px`);
            })
            .on("mouseout", () => {
                tooltip.style("visibility", "hidden");
            });

        // Add legend
        const legend = svg.append("g")
            .attr("transform", `translate(${width + margin.left + 20}, ${margin.top})`);

        legend.selectAll(".legend-item")
            .data(stressLevels.filter(level => level !== "All")) // Exclude "All" from legend
            .enter()
            .append("g")
            .attr("class", "legend-item")
            .attr("transform", (d, i) => `translate(0, ${i * 20})`)
            .each(function(d) {
                d3.select(this)
                    .append("rect")
                    .attr("width", 18)
                    .attr("height", 18)
                    .style("fill", color(d));

                d3.select(this)
                    .append("text")
                    .attr("x", 24)
                    .attr("y", 9)
                    .attr("dy", ".35em")
                    .style("text-anchor", "start")
                    .style("font-size", "12px")
                    .text(d);
            });
    }
</script>

<style>
    #chart {
        font-family: Arial, sans-serif;
        margin-top: 1rem;
    }

    select {
        margin: 1rem 0;
        padding: 0.5rem;
        font-size: 1rem;
    }
</style>

<div>
    <label for="work-location-filter">Filter by Work Location: </label>
    <select id="work-location-filter" bind:value={selectedWorkLocation} on:change={updateFilter}>
        <option value="All">All</option>
        {#each workLocations as location}
            <option value={location}>{location}</option>
        {/each}
    </select>

    <label for="stress-level-filter">Filter by Stress Level: </label>
    <select id="stress-level-filter" bind:value={selectedStressLevel} on:change={updateFilter}>
        <option value="All">All</option>
        {#each stressLevels.filter(level => level !== "All") as level}
            <option value={level}>{level}</option>
        {/each}
    </select>
</div>


<div id="chart"></div>
