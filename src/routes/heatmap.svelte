<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    let data = [];
    let heatmapData = [];
    let pieData = [];
    let selectedBlock = null;
    let pieTitle = "Access to Mental Health Resources (Overall)"; // Default title

    onMount(async () => {
        try {
            data = await d3.csv(
                'https://raw.githubusercontent.com/yix020/Final-Project/refs/heads/main/static/cleaned_data%20-%20cleaned_data%20copy.csv',
                d => ({
                    Stress_Level: d['Stress_Level'],
                    Mental_Health_Condition: d['Mental_Health_Condition'],
                    Access_to_Mental_Health_Resources: d['Access_to_Mental_Health_Resources'],
                })
            );

            // Group data for heatmap
            heatmapData = d3.rollups(
                data,
                v => v.length,
                d => d.Stress_Level,
                d => d.Mental_Health_Condition
            );

            // Calculate default pie chart data (overall distribution)
            pieData = d3.rollups(
                data,
                v => v.length,
                d => d.Access_to_Mental_Health_Resources
            );

            drawHeatmap();
            drawLegend();
            drawPieChart();
        } catch (error) {
            console.error("Error loading or processing data:", error);
        }
    });

    function drawHeatmap() {
        const svgWidth = 400;
        const svgHeight = 300;
        const margin = { top: 50, right: 30, bottom: 50, left: 70 };

        const width = svgWidth - margin.left - margin.right;
        const height = svgHeight - margin.top - margin.bottom;

        const svg = d3.select("#heatmap-container").html("").append("svg")
            .attr("width", svgWidth)
            .attr("height", svgHeight)
            .append("g")
            .attr("transform", `translate(${margin.left}, ${margin.top})`);

        const stressLevels = ["High", "Medium", "Low"];
        const conditions = Array.from(new Set(data.map(d => d.Mental_Health_Condition))).sort();

        const xScale = d3.scaleBand().domain(conditions).range([0, width]).padding(0.1);
        const yScale = d3.scaleBand().domain(stressLevels).range([0, height]).padding(0.1);

        const colorScale = d3.scaleSequential(d3.interpolateRdBu).domain([500, 300]);

        // Draw blocks
        svg.selectAll("rect")
            .data(heatmapData.flatMap(([stress, conds]) =>
                conds.map(([cond, count]) => ({ stress, cond, count }))
            ))
            .join("rect")
            .attr("x", d => xScale(d.cond))
            .attr("y", d => yScale(d.stress))
            .attr("width", xScale.bandwidth())
            .attr("height", yScale.bandwidth())
            .attr("fill", d => colorScale(d.count))
            .style("cursor", "pointer")
            .style("stroke", "none")
            .style("stroke-width", 2)
            .on("click", function (event, d) {
                if (selectedBlock) {
                    d3.select(selectedBlock)
                        .style("stroke", "none")
                        .style("opacity", 1);
                }

                d3.select(this)
                    .style("stroke", "black")
                    .style("opacity", 0.8);

                selectedBlock = this;

                // Update the pie chart title
                pieTitle = `Access to Mental Health Resources: ${d.cond} (${d.stress})`;

                // Filter data for the selected mental health condition
                const filteredData = data.filter(row =>
                    row.Stress_Level === d.stress &&
                    row.Mental_Health_Condition === d.cond
                );

                // Group data for pie chart
                pieData = d3.rollups(
                    filteredData,
                    v => v.length,
                    d => d.Access_to_Mental_Health_Resources
                );

                drawPieChart();
            });

        // Add data labels to blocks
        svg.selectAll("text")
            .data(heatmapData.flatMap(([stress, conds]) =>
                conds.map(([cond, count]) => ({ stress, cond, count }))
            ))
            .join("text")
            .attr("x", d => xScale(d.cond) + xScale.bandwidth() / 2)
            .attr("y", d => yScale(d.stress) + yScale.bandwidth() / 2)
            .attr("text-anchor", "middle")
            .attr("dominant-baseline", "middle")
            .text(d => d.count)
            .attr("fill", "#000")
            .style("font-size", "10px");

        // Add X axis
        svg.append("g")
            .attr("transform", `translate(0,${height})`)
            .call(d3.axisBottom(xScale))
            .selectAll("text")
            .style("text-anchor", "end")
            .attr("dx", "-0.8em")
            .attr("dy", "0.15em")
            .attr("transform", "rotate(-40)");

        // Add Y axis
        svg.append("g").call(d3.axisLeft(yScale));

        // Add title
        svg.append("text")
            .attr("x", width / 2)
            .attr("y", -20)
            .attr("text-anchor", "middle")
            .style("font-size", "12px")
            .style("font-weight", "bold")
            .text("Heatmap of Stress Levels by Mental Health Condition");
    }

    function drawLegend() {
        const legendWidth = 300; // Width of the legend
        const legendHeight = 20; // Height of the legend

        const svg = d3.select("#legend-container").html("").append("svg")
            .attr("width", legendWidth)
            .attr("height", legendHeight + 30) // Extra space for text labels
            .append("g")
            .attr("transform", `translate(50, 10)`); // Center below the heatmap

        const colorScale = d3.scaleSequential(d3.interpolateRdBu).domain([500, 300]);

        const legendScale = d3.scaleLinear().domain([500, 300]).range([0, legendWidth]);

        const legendAxis = d3.axisBottom(legendScale).ticks(4).tickSize(6);

        svg.selectAll("rect")
            .data(d3.range(0, 1, 1 / legendWidth))
            .join("rect")
            .attr("x", (d, i) => i)
            .attr("y", 0)
            .attr("width", 1)
            .attr("height", legendHeight)
            .attr("fill", d => colorScale(legendScale.invert(d * legendWidth)));

        svg.append("g")
            .attr("transform", `translate(0, ${legendHeight})`)
            .call(legendAxis)
            .selectAll("text")
            .style("font-size", "10px");
    }

    function drawPieChart() {
        const svgWidth = 300;
        const svgHeight = 360;
        const radius = Math.min(svgWidth, svgHeight - 80) / 2;

        const color = d3.scaleOrdinal()
            .domain(["Yes", "No"])
            .range(["#66c2a5", "#fc8d62"]);

        const svg = d3.select("#piechart-container").html("").append("svg")
            .attr("width", svgWidth)
            .attr("height", svgHeight);

        // Add user instructions at the top
        svg.append("text")
            .attr("x", svgWidth / 2)
            .attr("y", 10)
            .attr("text-anchor", "middle")
            .style("font-size", "10px")
            .style("font-style", "italic")
            .text("Click a block on the heatmap to update this chart");

        // Dynamically wrap the pie chart title if it is too long
        const titleGroup = svg.append("g")
            .attr("transform", `translate(${svgWidth / 2}, 30)`);

        const maxLineLength = 30;
        const words = pieTitle.split(" ");
        let line = "";
        let lines = [];

        // Wrap text logic
        words.forEach(word => {
            if ((line + word).length > maxLineLength) {
                lines.push(line.trim());
                line = word + " ";
            } else {
                line += word + " ";
            }
        });
        lines.push(line.trim());

        // Render each line of the title
        lines.forEach((text, i) => {
            titleGroup.append("text")
                .attr("y", i * 14)
                .attr("text-anchor", "middle")
                .style("font-size", "12px")
                .style("font-weight", "bold")
                .text(text);
        });

        // Group for the pie chart arcs and labels
        const chartGroup = svg.append("g")
            .attr("transform", `translate(${svgWidth / 2}, ${(svgHeight / 2) + 20})`);

        const pie = d3.pie().value(d => d[1]);
        const arc = d3.arc().innerRadius(0).outerRadius(radius);
        const hoverArc = d3.arc().innerRadius(0).outerRadius(radius + 10); // Effect on hover

        const slices = chartGroup.selectAll("path")
            .data(pie(pieData))
            .join("path")
            .attr("d", arc)
            .attr("fill", d => color(d.data[0]))
            .style("stroke", "#fff")
            .style("stroke-width", 1)
            .style("opacity", 0.8) // Initial opacity
            .on("mouseover", function (event, d) {
                d3.select(this)
                    .transition()
                    .duration(200) // Smooth transition
                    .attr("d", hoverArc) // Increase size
                    .style("opacity", 1); // Highlight
            })
            .on("mouseout", function (event, d) {
                d3.select(this)
                    .transition()
                    .duration(200)
                    .attr("d", arc) // Reset size
                    .style("opacity", 0.8); // Reset opacity
            });

        // Animate the slices on load
        slices.transition()
            .duration(1000)
            .attrTween("d", function (d) {
                const interpolate = d3.interpolate({ startAngle: 0, endAngle: 0 }, d);
                return function (t) {
                    return arc(interpolate(t));
                };
            });

        chartGroup.selectAll("text")
            .data(pie(pieData))
            .join("text")
            .attr("transform", d => `translate(${arc.centroid(d)})`)
            .attr("text-anchor", "middle")
            .attr("font-size", "10px")
            .text(d => `${d.data[0]} (${d.data[1]})`);
    }


</script>

<style>
    #heatmap-and-piechart {
        display: flex;
        justify-content: flex-start; /* Align heatmap and pie chart horizontally */
        align-items: flex-start; /* Align items to the top */
        gap: 20px; /* Space between heatmap and pie chart */
    }

    #heatmap-container {
        flex: 1; /* Allow heatmap to grow */
    }

    #legend-container {
        margin-top: 10px;
        width: 300px; /* Fixed width for legend */
    }

    #piechart-container {
        width: 300px; /* Fixed width for pie chart */
    }
</style>

<div id="heatmap-and-piechart">
    <div>
        <div id="heatmap-container"></div>
        <div id="legend-container"></div>
    </div>
    <div id="piechart-container"></div>
</div>
