<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    let data = [];

    onMount(async () => {
        // Fetch and process the data
        data = await d3.csv(
            'https://raw.githubusercontent.com/yix020/Final-Project/refs/heads/main/static/cleaned_data%20-%20cleaned_data%20copy.csv',
            d => ({
                Stress_Level: d['Stress_Level'], // Stress level (e.g., "Low", "Medium", "High")
                Hours_Worked_Per_Week: +d['Hours_Worked_Per_Week'] // Convert to numeric
            })
        );

        console.log("Processed Data:", data);
        drawGroupedBarPlot();
    });

    function drawGroupedBarPlot() {
        const margin = { top: 50, right: 30, bottom: 100, left: 70 };
        const width = 700 - margin.left - margin.right;
        const height = 400 - margin.top - margin.bottom;

        // Clear existing SVG
        d3.select("#grouped-barplot").html("");

        // Create SVG container
        const svg = d3
            .select("#grouped-barplot")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left},${margin.top})`);

        // Tooltip element
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

        // Group data by stress level
        const groupedData = d3.group(data, d => d.Stress_Level);

        // Define categories for Hours Worked
        const hoursCategories = ["<30", "30-40", ">40"];
        const categorizedData = [];

        groupedData.forEach((values, stressLevel) => {
            const groupedByHours = d3.rollups(
                values,
                v => v.length, // Count rows instead of summing employees
                d => {
                    if (d.Hours_Worked_Per_Week < 30) return "<30";
                    else if (d.Hours_Worked_Per_Week <= 40) return "30-40";
                    else return ">40";
                }
            );

            groupedByHours.forEach(([hoursCategory, count]) => {
                categorizedData.push({
                    Stress_Level: stressLevel,
                    Hours_Category: hoursCategory,
                    Count: count
                });
            });
        });

        console.log("Categorized Data:", categorizedData);

        // Define scales
        const x0Scale = d3
            .scaleBand()
            .domain(["Low", "Medium", "High"]) // Stress levels
            .range([0, width])
            .padding(0.2);

        const x1Scale = d3
            .scaleBand()
            .domain(hoursCategories) // Hours categories
            .range([0, x0Scale.bandwidth()])
            .padding(0.1);

        const yScale = d3
            .scaleLinear()
            .domain([0, d3.max(categorizedData, d => d.Count)])
            .nice()
            .range([height, 0]);

        const colorScale = d3
            .scaleOrdinal()
            .domain(hoursCategories)
            .range(["#1f77b4", "#ff7f0e", "#2ca02c"]);

        // Draw X-axis
        svg
            .append("g")
            .attr("transform", `translate(0,${height})`)
            .call(d3.axisBottom(x0Scale));

        // Add X-axis label
        svg
            .append("text")
            .attr("x", width / 2)
            .attr("y", height + margin.bottom - 10)
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .text("Stress Level");

        // Draw Y-axis
        svg.append("g").call(d3.axisLeft(yScale));

        // Add Y-axis label
        svg
            .append("text")
            .attr("transform", "rotate(-90)")
            .attr("x", -height / 2)
            .attr("y", -50)
            .attr("text-anchor", "middle")
            .style("font-size", "14px")
            .text("Number of Employees");

        // Draw bars with tooltip
        svg
            .selectAll("g.bars")
            .data(Array.from(groupedData.keys()))
            .enter()
            .append("g")
            .attr("transform", d => `translate(${x0Scale(d)},0)`)
            .selectAll("rect")
            .data(d =>
                categorizedData.filter(item => item.Stress_Level === d)
            )
            .enter()
            .append("rect")
            .attr("x", d => x1Scale(d.Hours_Category))
            .attr("y", d => yScale(d.Count))
            .attr("width", x1Scale.bandwidth())
            .attr("height", d => height - yScale(d.Count))
            .attr("fill", d => colorScale(d.Hours_Category))
            .on("mouseover", (event, d) => {
                d3.select(event.currentTarget)
                    .attr("opacity", 0.8)
                    .attr("stroke", "black")
                    .attr("stroke-width", 1.5);

                tooltip
                    .style("opacity", 1)
                    .html(
                        `<strong>Stress Level:</strong> ${d.Stress_Level}<br>
                         <strong>Hours Category:</strong> ${d.Hours_Category}<br>
                         <strong>Count:</strong> ${d.Count}`
                    )
                    .style("left", event.pageX + 10 + "px")
                    .style("top", event.pageY - 20 + "px");
            })
            .on("mousemove", event => {
                tooltip
                    .style("left", event.pageX + 10 + "px")
                    .style("top", event.pageY - 20 + "px");
            })
            .on("mouseout", event => {
                d3.select(event.currentTarget)
                    .attr("opacity", 1)
                    .attr("stroke", "none");

                tooltip.style("opacity", 0);
            });

        // Add legend
        const legend = svg
            .append("g")
            .attr("transform", `translate(${width - 150}, -10)`);

        hoursCategories.forEach((category, i) => {
            legend
                .append("rect")
                .attr("x", 0)
                .attr("y", i * 20)
                .attr("width", 12)
                .attr("height", 12)
                .attr("fill", colorScale(category));

            legend
                .append("text")
                .attr("x", 20)
                .attr("y", i * 20 + 10)
                .style("font-size", "12px")
                .text(category);
        });

        // Add title
        svg
            .append("text")
            .attr("x", width / 2)
            .attr("y", -margin.top / 2)
            .attr("text-anchor", "middle")
            .style("font-size", "16px")
            .style("font-weight", "bold")
            .text("Number of Employees by Stress Level and Hours Worked");
    }
</script>

<style>
    .tooltip {
        pointer-events: none;
    }
</style>

<svg id="grouped-barplot"></svg>
