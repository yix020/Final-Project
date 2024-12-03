<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
  
    let data = [];
    let filteredData = [];
    let selectedStressLevel = "All";
    let selectedRole = "All";
    let selectedIndustry = "All";
    let loading = true;
  
    const margin = { top: 60, right: 150, bottom: 50, left: 70 };
    const width = 600 - margin.left - margin.right;
    const height = 300 - margin.top - margin.bottom;
  
    const stressLevels = ["High", "Medium", "Low"];
    const roles = ["All", "Data Scientist", "Designer", "HR", "Marketing", "Project Manager", "Sales", "Software Engineer"];
    const industries = ["All", "Consulting", "Education", "Finance", "Healthcare", "IT", "Manufacturing", "Retail"];
  
    const color = d3.scaleOrdinal()
      .domain(stressLevels)
      .range([
      "#FF0000", // Red for High
      "#CCCCCC", // Gray for Medium
      "#00FF00"  // Green for Low
    ]);
  
    onMount(async () => {
      loading = true;
      data = await d3.csv(
        'https://raw.githubusercontent.com/yix020/Final-Project/refs/heads/main/static/cleaned_data%20-%20cleaned_data%20copy.csv',
        d => ({
          Stress_Level: d['Stress_Level'],
          Work_Location: d['Work_Location'],
          Role: d['Job_Role'],
          Industry: d['Industry']
        })
      );
      filteredData = data;
      drawChart();
      loading = false;
    });
  
    function updateFilter() {
      loading = true;
      filteredData = data.filter(d =>
        (selectedStressLevel === "All" || d.Stress_Level === selectedStressLevel) &&
        (selectedRole === "All" || d.Role === selectedRole) &&
        (selectedIndustry === "All" || d.Industry === selectedIndustry)
      );
      drawChart();
      loading = false;
    }
  
    function drawChart() {
      d3.select("#barchart").selectAll("*").remove();
  
      const groupedData = d3.rollups(
        filteredData,
        v => {
          const stressLevelCounts = d3.rollup(v, v => v.length, d => d.Stress_Level);
          return { stressLevelCounts };
        },
        d => d.Work_Location
      );
  
      const stackedData = d3.stack()
        .keys(stressLevels)
        .value((d, key) => d[1].stressLevelCounts.get(key) || 0)(groupedData);
  
      const x0 = d3.scaleBand()
        .domain(["Remote", "Onsite", "Hybrid"])
        .range([0, width])
        .padding(0.2);
  
      const y = d3.scaleLinear()
        .domain([0, d3.max(stackedData.flat(), d => d[1])])
        .nice()
        .range([height, 0]);
  
      const svg = d3.select("#barchart").html("")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left}, ${margin.top})`);

  
    //   const plotGroup = svg.append("g")
      svg.append("g").attr("transform", `translate(${margin.left},${margin.top})`);
  
      svg.append("g")
        .call(d3.axisBottom(x0))
        .attr("transform", `translate(0,${height})`);
  
        svg.append("g")
        .call(d3.axisLeft(y));
  
      // Adding x-axis label
      svg.append("text")
        .attr("x", width / 2)
        .attr("y", height + margin.bottom - 10)
        .attr("text-anchor", "middle")
        .style("font-size", "14px")
        .text("Work Model");
  
      // Adding y-axis label
      svg.append("text")
        .attr("transform", "rotate(-90)")
        .attr("y", -margin.left + 20)
        .attr("x", -height / 2)
        .attr("dy", "1em")
        .style("text-anchor", "middle")
        .style("font-size", "14px")
        .text("Number of Employees Reported");
  
      // Adding chart title
    
      const tooltip = d3.select("#tooltip").html("")
        .style("position", "absolute")
        .style("visibility", "hidden");
  
      svg.selectAll(".layer")
        .data(stackedData)
        .join("g")
        .attr("fill", d => color(d.key))
        .selectAll("rect")
        .data(d => d)
        .join("rect")
        .attr("x", d => x0(d.data[0]))
        .attr("y", d => y(d[1]))
        .attr("height", d => y(d[0]) - y(d[1]))
        .attr("width", x0.bandwidth())
        .on("mouseover", (event, d) => {
          tooltip.style("visibility", "visible").html(`Count: ${d[1] - d[0]}`);
        })
        .on("mousemove", event => {
          tooltip.style("top", `${event.pageY}px`).style("left", `${event.pageX + 10}px`);
        })
        .on("mouseout", () => tooltip.style("visibility", "hidden"));
  
      const legend = svg.append("g")
        .attr("transform", `translate(${width + 50}, ${margin.top})`);
  
      legend.append("text")
        .attr("x", 0)
        .attr("y", -10)
        .style("font-size", "14px")
        .style("font-weight", "bold")
        .text("Stress Level");
  
      stressLevels.forEach((level, i) => {
        legend.append("rect")
          .attr("x", 0)
          .attr("y", i * 20)
          .attr("width", 15)
          .attr("height", 15)
          .attr("fill", color(level));
  
        legend.append("text")
          .attr("x", 20)
          .attr("y", i * 20 + 12)
          .text(level)
          .style("font-size", "12px")
          .attr("alignment-baseline", "middle");
      });
    }
  </script>
  
  <!-- Chart Title Container -->
  <div class="title-container">
    <!-- <svg width="100%" height="50"> -->
      <text
        x="50%"
        y="30"
        text-anchor="middle"
        font-size="18"
        font-weight="bold"
      >
        Job Roles & Industry vs. Stress Level under Different Work Models
      </text>
    <!-- </svg> -->
  </div>
  
  <!-- Dropdown Filters -->
  <div class="filter-container">
    <div>
      <label for="roles">Role:</label>
      <select id="roles" bind:value={selectedRole} on:change={updateFilter}>
        {#each roles as role}
          <option value={role}>{role}</option>
        {/each}
      </select>
    </div>
    <div>
      <label for="industries">Industry:</label>
      <select id="industries" bind:value={selectedIndustry} on:change={updateFilter}>
        {#each industries as industry}
          <option value={industry}>{industry}</option>
        {/each}
      </select>
    </div>
  </div>
  
  <!-- Chart Container -->
  <div id="barchart"></div>
  <div id="tooltip"></div>
  
  <style>
    .title-container {
      text-align: center;
      margin-bottom: 20px; /* Space between title and dropdown */
    }
  
    .filter-container {
      display: flex;
      justify-content: center;
      margin-top: 20px;  /* Space between the title and filters */
      gap: 20px; /* Space between dropdowns */
    }
  
    #tooltip {
      background: white;
      border: 1px solid black;
      padding: 5px;
      border-radius: 3px;
    }
  </style>