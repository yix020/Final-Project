<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    let data = [];
    let filteredData = [];
    let selectedWorkLocation = "All"; // Default filter value for work location
    let selectedStressLevel = "All"; // Default filter value for stress level
    let loading = true; // Flag for showing the loading animation
    let observer; // Intersection Observer variable

    const margin = { top: 60, right: 150, bottom: 50, left: 70 };
    const width = 600 - margin.left - margin.right;
    const height = 300 - margin.top - margin.bottom;

    const workLocations = ["Onsite", "Hybrid", "Remote"];
    const stressLevels = ["High", "Medium", "Low"]; // Predefined stress levels

    onMount(async () => {
        loading = true; // Show loading animation
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

        loading = false; // Hide loading animation

        // Setup the Intersection Observer
        setupObserver();
    });

    function setupObserver() {
        const chartElement = document.getElementById("chart");

        observer = new IntersectionObserver(
            ([entry]) => {
                if (entry.isIntersecting) {
                    resetAnimation(); // Reset chart and animation state
                    startAutomaticTransition(); // Start the animation
                }
            },
            { threshold: 0.5 } // Trigger when 50% of the chart is visible
        );

        observer.observe(chartElement);
    }

    function resetAnimation() {
        // Clear the chart and reset the selected stress level
        selectedStressLevel = "All";
        filteredData = data; // Reset filtered data
        drawChart(); // Redraw chart with initial data
    }

    function updateFilter(stressLevel = "All") {
        loading = true; // Show loading animation
        selectedStressLevel = stressLevel; // Update the stress level filter

        // Filter data by work location and stress level
        filteredData = data.filter(d =>
            (selectedWorkLocation === "All" || d.Work_Location === selectedWorkLocation) &&
            (stressLevel === "All" || d.Stress_Level === stressLevel)
        );

        drawChart(); // Redraw chart with filtered data
        loading = false; // Hide loading animation
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
            stressLevels.map(stressLevel => ({
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
            .domain(stressLevels)
            .range([0, x0.bandwidth()])
            .padding(0.1);

        const y = d3.scaleLinear()
            .domain([0, d3.max(flattenedData, d => d.count)])
            .nice()
            .range([height, 0]);

        const color = d3.scaleOrdinal()
            .domain(stressLevels)
            .range([d3.schemeTableau10[2], d3.schemeTableau10[1], d3.schemeTableau10[0]]); // "High" gets "Low" color, and vice versa

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
            .text(
                selectedStressLevel === "All"
                    ? "Stress Levels Across Work Locations"
                    : `Stress Level: ${selectedStressLevel}`
            );

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
        plotGroup.append("g").call(d3.axisLeft(y));

        // Add Y-axis label
        plotGroup.append("text")
            .attr("transform", "rotate(-90)")
            .attr("y", -margin.left + 20)
            .attr("x", -height / 2)
            .attr("dy", "1em")
            .style("text-anchor", "middle")
            .style("font-size", "12px")
            .text("Number of Employees");

        // Add grouped bars
        plotGroup.append("g")
            .selectAll("g")
            .data(workLocations) // Group by work location
            .enter()
            .append("g")
            .attr("transform", d => `translate(${x0(d)},0)`) // Position groups
            .selectAll("rect")
            .data(workLocation => stressLevels.map(level => ({
                level,
                workLocation,
                count: flattenedData.find(
                    d => d.workLocation === workLocation && d.stressLevel === level
                )?.count || 0
            })))
            .enter()
            .append("rect")
            .attr("x", d => x1(d.level)) // Position within group
            .attr("y", height)
            .attr("width", x1.bandwidth())
            .attr("height", 0)
            .attr("fill", d => color(d.level))
            .transition()
            .duration(2500) // Slower animation
            .ease(d3.easeCubicOut)
            .attr("y", d => y(d.count))
            .attr("height", d => height - y(d.count));

                // Add legend
                const legend = svg.append("g")
            .attr("transform", `translate(${width + margin.left + 20}, ${margin.top})`);

        stressLevels.forEach((level, i) => {
            const legendRow = legend.append("g")
                .attr("transform", `translate(0, ${i * 20})`);

            // Legend color box
            legendRow.append("rect")
                .attr("width", 15)
                .attr("height", 15)
                .attr("fill", color(level));

            // Legend text
            legendRow.append("text")
                .attr("x", 20)
                .attr("y", 12)
                .style("text-anchor", "start")
                .style("font-size", "12px")
                .text(level);
        });

    }

    function startAutomaticTransition() {
        const steps = ["All", "Low", "Medium", "High"];
        let stepIndex = 0;

        function nextStep() {
            updateFilter(steps[stepIndex]);
            stepIndex++;

            // Stop the animation after "High stress level"
            if (stepIndex < steps.length) {
                setTimeout(nextStep, 5000); // 5-second delay between steps
            }
        }

        nextStep(); // Start the first step
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

    .loading-container {
        text-align: center;
        margin-top: 2rem;
        font-size: 1.2rem;
        font-weight: bold;
    }
</style>

<!-- <div>
    <label for="stress-level-filter">Filter by Stress Level: </label>
    <select id="stress-level-filter" bind:value={selectedStressLevel} on:change={updateFilter}>
        <option value="All">All</option>
        {#each stressLevels as level}
            <option value={level}>{level}</option>
        {/each}
    </select>
</div> -->

{#if loading}
    <div class="loading-container">Loading data...</div>
{/if}

<div id="chart" style:display={loading ? "none" : "block"}></div>