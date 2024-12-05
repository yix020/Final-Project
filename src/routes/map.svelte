<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    let groupedData = [];
    let filteredData = [];
    let selectedStressLevel = "All";

    onMount(async () => {
        const rawData = await d3.csv(
            'https://raw.githubusercontent.com/yix020/Final-Project/refs/heads/main/static/cleaned_data%20-%20cleaned_data%20copy.csv',
            d => ({
                Stress_Level: d['Stress_Level'],
                Age: +d['Age']
            })
        );

        const stressLevelMapping = { Low: 1, Medium: 2, High: 3 };
        const processedData = rawData
            .map(d => ({
                ...d,
                Stress_Level: stressLevelMapping[d.Stress_Level] || null
            }))
            .filter(d => d.Stress_Level !== null);

        groupedData = Array.from(
            d3.rollup(
                processedData,
                v => v.length,
                d => d.Age,
                d => d.Stress_Level
            ),
            ([Age, stressGroup]) => ({
                Age: Age,
                StressGroups: Array.from(stressGroup, ([Stress_Level, Count]) => ({
                    Stress_Level,
                    Count
                }))
            })
        ).flatMap(d =>
            d.StressGroups.map(group => ({
                Age: d.Age,
                Stress_Level: group.Stress_Level,
                Count: group.Count
            }))
        );

        filteredData = [...groupedData];
        drawScatterPlot();
    });

    function filterData(stressLevel) {
        selectedStressLevel = stressLevel;

        if (stressLevel === "All") {
            filteredData = [...groupedData];
        } else {
            const stressLevelMapping = { Low: 1, Medium: 2, High: 3 };
            const numericLevel = stressLevelMapping[stressLevel];
            filteredData = groupedData.filter(d => d.Stress_Level === numericLevel);
        }

        drawScatterPlot();
    }

    function drawScatterPlot() {
        const margin = { top: 40, right: 150, bottom: 100, left: 70 }; // Adjusted left margin for Y-axis label
        const width = 700 - margin.left - margin.right;
        const height = 360 - margin.top - margin.bottom;

        d3.select("#scatterplot").html("");

        const svg = d3
            .select("#scatterplot")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left},${margin.top})`);

        const xScale = d3
            .scalePoint()
            .domain([...new Set(groupedData.map(d => d.Age))].sort((a, b) => a - b))
            .range([0, width])
            .padding(0.5);

        const sizeScale = d3
            .scaleLinear()
            .domain([0, d3.max(groupedData, d => d.Count)])
            .range([3, 15]);

        const yScale = d3
            .scaleLinear()
            .domain([0, d3.max(groupedData, d => d.Count)])
            .range([height, 0]);

        const colorScale = d3
            .scaleOrdinal()
            .domain([1, 2, 3])
            .range(["green", "orange", "red"]);

        // Add title
        svg
            .append("text")
            .attr("x", width / 2)
            .attr("y", -margin.top / 2)
            .attr("text-anchor", "middle")
            .style("font-size", "16px")
            .style("font-weight", "bold")
            .text("Employee Count by Age and Stress Level");

        // Add X axis
        svg
            .append("g")
            .attr("transform", `translate(0,${height})`)
            .call(d3.axisBottom(xScale))
            .selectAll("text")
            .attr("transform", "rotate(-45)")
            .style("text-anchor", "end");

        // Add X-axis label
        svg
            .append("text")
            .attr("x", width / 2)
            .attr("y", height + margin.bottom - 30) // Position below the X-axis
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .text("Age");

        // Add Y axis
        svg.append("g").call(d3.axisLeft(yScale).ticks(5));

        // Add Y-axis label
        svg
            .append("text")
            .attr("transform", "rotate(-90)") // Rotate the label
            .attr("x", -height / 2) // Center vertically
            .attr("y", -margin.left + 20) // Position to the left of the Y-axis
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .text("Employee Count");

        // Add tooltip
        const tooltip = d3
            .select("body")
            .append("div")
            .attr("class", "tooltip")
            .style("position", "absolute")
            .style("background-color", "white")
            .style("border", "1px solid #ccc")
            .style("padding", "5px")
            .style("border-radius", "5px")
            .style("pointer-events", "none")
            .style("font-size", "12px")
            .style("opacity", 0);

        // Add points
        svg
            .selectAll("circle")
            .data(filteredData)
            .enter()
            .append("circle")
            .attr("cx", d => xScale(d.Age))
            .attr("cy", d => yScale(d.Count))
            .attr("r", d => sizeScale(d.Count))
            .attr("fill", d => colorScale(d.Stress_Level))
            .attr("opacity", 0.8)
            .on("mouseover", (event, d) => {
                tooltip
                    .style("opacity", 1)
                    .html(
                        `<strong>Age:</strong> ${d.Age}<br>
                         <strong>Stress Level:</strong> ${
                             d.Stress_Level === 1 ? "Low" : d.Stress_Level === 2 ? "Medium" : "High"
                         }<br>
                         <strong>Count:</strong> ${d.Count}`
                    );
                d3.select(event.target).attr("stroke", "black").attr("stroke-width", 2);
            })
            .on("mousemove", event => {
                tooltip
                    .style("left", event.pageX + 10 + "px")
                    .style("top", event.pageY - 20 + "px");
            })
            .on("mouseout", event => {
                tooltip.style("opacity", 0);
                d3.select(event.target).attr("stroke", null);
            });

        // Add legend
        const legend = svg
            .selectAll(".legend")
            .data(colorScale.domain())
            .enter()
            .append("g")
            .attr("class", "legend")
            .attr("transform", (d, i) => `translate(${width + 20},${i * 20})`);

        legend
            .append("rect")
            .attr("x", 0)
            .attr("width", 12)
            .attr("height", 12)
            .attr("fill", d => colorScale(d));

        legend
            .append("text")
            .attr("x", 20)
            .attr("y", 6)
            .attr("dy", "0.35em")
            .style("font-size", "12px")
            .text(d => (d === 1 ? "Low" : d === 2 ? "Medium" : "High"));
    }
</script>

<div style="text-align: center; margin-bottom: 10px;">
    <button on:click={() => filterData("All")}>Show All</button>
    <button on:click={() => filterData("Low")}>Low Stress</button>
    <button on:click={() => filterData("Medium")}>Medium Stress</button>
    <button on:click={() => filterData("High")}>High Stress</button>
</div>

<svg id="scatterplot"></svg>
