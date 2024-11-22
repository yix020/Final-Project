
<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    let data = [];
    let filteredData = [];
    let selectedWorkLocation = "All"; // Default filter value
    const margin = { top: 60, right: 150, bottom: 50, left: 70 };
    const width = 800 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;

    // Explicit work location options
    const workLocations = ["All", "Remote", "Hybrid", "Onsite"];
    const stressLevels = ["High", "Medium", "Low"]; // Predefined stress levels

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

    // Update chart based on filter
    function updateFilter() {
        if (selectedWorkLocation === "All") {
            filteredData = data; // Show all data
        } else {
            filteredData = data.filter(d => d.Work_Location === selectedWorkLocation);
        }
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
        const flattenedData = [];
        groupedData.forEach(([workLocation, stressLevels]) => {
            stressLevels.forEach(([stressLevel, count]) => {
                flattenedData.push({ workLocation, stressLevel, count });
            });
        });

        // Extract unique categories
        const uniqueWorkLocations = [...new Set(flattenedData.map(d => d.workLocation))];

        // Scales
        const x0 = d3.scaleBand()
            .domain(uniqueWorkLocations)
            .range([0, width])
            .padding(0.2);

        const x1 = d3.scaleBand()
            .domain(stressLevels)
            .range([0, x0.bandwidth()])
            .padding(0.1);

        const y = d3.scaleLinear()
            .domain([0, d3.max(flattenedData, d => d.count) || 1]) // Avoid 0 max issues
            .nice()
            .range([height, 0]);

        const color = d3.scaleOrdinal()
            .domain(stressLevels)
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
            .text("Stress Levels Across Work Locations");

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

        // Add bars with tooltip interaction
        plotGroup.append("g")
            .selectAll("g")
            .data(flattenedData)
            .enter()
            .append("g")
            .attr("transform", d => `translate(${x0(d.workLocation)},0)`)
            .selectAll("rect")
            .data(d => stressLevels.map(level => ({
                level,
                count: flattenedData.find(
                    x => x.workLocation === d.workLocation && x.stressLevel === level
                )?.count || 0,
                workLocation: d.workLocation
            })))
            .enter()
            .append("rect")
            .attr("x", d => x1(d.level))
            .attr("y", d => y(d.count))
            .attr("width", x1.bandwidth())
            .attr("height", d => height - y(d.count))
            .attr("fill", d => color(d.level))
            .on("mouseover", (event, d) => {
                tooltip
                    .style("visibility", "visible")
                    .html(`
                        <strong>Stress Level:</strong> ${d.level}<br>
                        <strong>Work Location:</strong> ${d.workLocation}<br>
                        <strong>Employees:</strong> ${d.count}
                    `);
            })
            .on("mousemove", (event) => {
                tooltip
                    .style("top", `${event.pageY - 10}px`)
                    .style("left", `${event.pageX + 10}px`);
            })
            .on("mouseout", () => {
                tooltip.style("visibility", "hidden");
            });

        // Add legend
        const legend = svg.append("g")
            .attr("transform", `translate(${width + margin.left + 20}, ${margin.top})`);

        legend.selectAll(".legend-item")
            .data(stressLevels)
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
    <label for="filter">Filter by Work Location: </label>
    <select id="filter" bind:value={selectedWorkLocation} on:change={updateFilter}>
        {#each workLocations as location}
            <option value={location}>{location}</option>
        {/each}
    </select>
</div>

<div id="chart"></div>
