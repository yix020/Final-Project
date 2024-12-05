
<script>
    import StressLevelsChart from './MentalHealthBarChart.svelte'; // Import your bar chart component
    import Heatmap from './heatmap.svelte'; // Import the heatmap component
    import Barchart from './barchart.svelte';
    import Bubblemap from './map.svelte';
    import Plot from './plot.svelte';

</script>
<style>
    main {
        display: flex;
        flex-direction: column;
        align-items: center; /*Center content horizontally */
        justify-content: center; /* Center content vertically */
        text-align: center;
        min-height: 100vh; /* Full height of the viewport */
        padding: 2rem;
        font-family: Arial, sans-serif;
    }

    h1 {
        font-size: 2rem;
        margin-bottom: 1rem;
        align-items: center;
    }

    #chart-container {
        width: 100%;
        max-width: 900px; /* Optional: Limit the width of the chart */
        align-items: center;
    }

    h2 {
        font-size: 1rem;
        margin-bottom: 1rem;
    }

    p {
        font-size: 1rem;
        line-height: 1.6;
        max-width: 800px;
        text-align: left;
        margin-bottom: 2rem;
    }
</style>

<main>
    <h1>Is the Trend of Remote Work Positive or Negative?</h1>

    <p>
        <strong>Introduction</strong><br><br>
        What is your job? Do you work in an office, or do you work from home in pajama pants? 
        For many, remote work sounds ideal, but just a decade ago, this concept seemed far-fetched. 
        Then, the global pandemic catalyzed a dramatic transformation. Almost overnight, schools, 
        workplaces, meetings, and interviews moved online.
        <br><br>
        This shift marked the beginning of a new era. According to the Quarterly Economic Briefing 
        in 2020, only 6% of U.S. employees primarily worked remotely before the pandemic, and 75% 
        had never done so. In 2020, however, over 35% of employees were forced to work from home 
        during the shutdown period.
        <br><br>
        As the pandemic subsided, life began to return to normal—but only partially. A 2020 PwC 
        survey revealed that over 80% of employees wanted to continue working remotely at least 
        one day a week. Many discovered the benefits of working from home, including flexibility 
        and better work-life balance. This shift introduced three primary work models: on-site, 
        fully remote, and hybrid, where employees split time between home and the office.
        <br><br>
        Yet, an interesting paradox arises: data suggests that remote workers report slightly 
        higher stress levels compared to on-site or hybrid workers. This raises a key question: 
        Is remote work a beneficial trend, or does it present hidden challenges for employees?
        <br><br>
        We conducted an in-depth analysis of 5,000 records collected from employees worldwide, 
        examining how different work models correlate with both physical and mental health outcomes.
    </p>

    <div id="chart-container"> <!-- Chart title moved above the chart -->
        <StressLevelsChart /> <!-- Embed your chart component -->
    </div>

    <p>
        <strong>Analysis</strong><br><br>
        The animation plot reveals that remote work is associated with a higher proportion of employees reporting High stress level compared to other work models. This trend suggests that remote work may be linked to increased stress for many employees. In contrast, onsite work shows a larger proportion of employees with Low stress level, indicating it is the least stressful work model overall. Hybrid work falls in the middle, with a more balanced distribution of stress levels, where Medium stress is the most prevalent.
        <br><br>
        These findings highlight a significant variation in stress levels across different work models. The higher prevalence of high stress among remote workers emphasizes the need for further exploration into the factors contributing to stress in remote work environments, while onsite and hybrid models may offer potential pathways for reducing stress in the workforce.
    </p>

    <div id="interactive-container">
        <div id="heatmap">
            <Heatmap /> <!-- Heatmap Component -->
        </div>

    </div>

    <p>
        Beyond variations in stress levels across work models, our heatmap and pie chart analyses offer deeper insights into the interplay between stress, mental health conditions, and access to resources. The heatmap highlights distinct patterns, with High Stress Levels frequently linked to specific mental health conditions. This finding aligns with the observation that remote work environments see higher proportions of employees experiencing high stress. The concentration of high-stress conditions in these contexts suggests that mental health challenges may exacerbate stress for remote workers.
        <br><br>
        The pie chart further uncovers disparities in resource accessibility for these groups. Notably, individuals in high-stress categories often report limited access to mental health resources, as shown by the larger "No" segments in the pie chart for these conditions. This lack of support may compound the challenges faced by remote workers. In contrast, conditions linked to Low Stress Levels, more common in onsite work models, demonstrate significantly better resource availability. The combination of supportive conditions and accessible resources may help explain why onsite work is associated with lower stress levels.
        <br><br>
        These findings underscore the importance of addressing resource disparities to alleviate stress for high-stress groups, particularly in remote work settings. By improving access to mental health resources and tailoring interventions for specific conditions, employers can mitigate the stress disparities observed and foster healthier work environments across all models.
    </p>


    <div id= "map-container">
        <div id="map">
            <Bubblemap />
        </div>
    </div>

    <p>
        This scatter plot visualizes the employee count across different age groups and stress levels. The size of the points represents the number of employees, while the color indicates the stress level: green for Low, orange for Medium, and red for High. The X-axis represents the age groups, sorted in ascending order, and the Y-axis indicates the number of employees in each group.
        <br><br>
        The plot reveals that employees with Low stress levels (green) are evenly distributed across most age groups, while Medium stress levels (orange) dominate in certain age ranges, particularly in the mid-30s to 40s. High stress levels (red) are less frequent but appear consistently across various ages, with some clusters in younger employees. Larger points in the mid-age groups suggest higher employee concentrations, likely correlating with more experienced workers or common age ranges in the workforce.
        <br><br>
        This plot provides insights into how age and stress levels interact, highlighting potential areas for targeted interventions, such as stress management programs for high-stress groups or further investigation into the causes of stress among specific age ranges.

    </p>

    <div id="barchart-container">
        <div id="Barchart">
            <Barchart/> 
        </div>

    </div>
    
    <p>
        We can use this tool to explore how stress levels vary across different roles and industries. Notably, distinct patterns emerge. For example, software engineers report lower stress levels when working remotely compared to onsite, while designers experience higher stress when working remotely. Similarly, employees in Education report lower stress levels when onsite, whereas those in the Finance industry report significantly higher stress levels when working remotely. Additionally, we can compare the stress levels of data scientists across different industries, such as healthcare, IT, and consulting.

    </p>

    <div id="barchart2-container">
        <div id="bar">
            <Plot/> 
        </div>

    </div>

    <p>
        The grouped bar plot visualizes the distribution of employees based on stress levels (Low, Medium, High) and hours worked per week (&lt;30, 30-40, &gt;40). It shows that the 30-40 hours category is the most common across all stress levels, while the &lt;30 and &gt;40 hours categories exhibit more variability. Employees in the High-Stress group are more likely to work over 40 hours per week, whereas the Low-Stress group demonstrates a more balanced distribution across all hours categories. The Medium Stress group has the most uniform distribution.

    </p>

    <p>
        <strong>Conclusion & Summary </strong><br><br>
        In this analysis, we examined how different work models relate to employees’ stress levels and work schedules, considering factors such as mental health, access to mental health resources, age, roles, industries, and weekly working hours. At first glance, it seems that remote work may lead to higher stress levels for employees. However, after exploring various subcategories, we found that while there is a relationship between stress levels and work models, it is not strongly significant. Nevertheless, we can still draw some important insights.
        <br><br>
        It is clear that high-stress levels are strongly correlated with mental health issues like burnout and anxiety, and a significant portion of employees report lacking access to mental health resources. This suggests that we should pay more attention to employee mental health, regardless of the work model. Specifically, we could offer programs such as stress relief consultations, exercise activities, or even mindfulness classes, as studies have shown that physical activity improves both mental and emotional well-being. Additionally, based on the data regarding roles and industries, we could adjust the allocation of resources to industries with higher stress levels to provide more targeted support.
        
    </p>
 
    <p>
        <strong>Reference </strong> <br><br>
        Coate, Patrick. “Remote Work Before, During, and After the Pandemic”. Jan 25, 2021.
        https://www.ncci.com/SecureDocuments/QEB/QEB_Q4_2020_RemoteWork.html



    </p>
</main>

