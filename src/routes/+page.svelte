
<script>
    import StressLevelsChart from './MentalHealthBarChart.svelte'; // Import your bar chart component
    import Heatmap from './Heatmap.svelte'; // Import the heatmap component

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

</main>

