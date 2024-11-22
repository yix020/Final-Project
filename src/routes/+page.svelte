

<style>
    main {
        display: flex;
        flex-direction: column;
        align-items: center; /* Center content horizontally */
        justify-content: center; /* Center content vertically */
        text-align: center;
        min-height: 100vh; /* Full height of the viewport */
        padding: 2rem;
        font-family: Arial, sans-serif;
    }

    h1 {
        font-size: 2rem;
        margin-bottom: 1rem;
    }

    #chart-container {
        width: 100%;
        max-width: 900px; /* Optional: Limit the width of the chart */
    }

    h2 {
        font-size: 1.5rem;
        margin-bottom: 1rem;
    }
</style>

<main>
    <h1>Is the Trend of Remote Work Positive or Negative?</h1>
    <h2>Stress Levels Across Work Locations</h2>
    <div id="chart-container"> <!-- Chart title moved above the chart -->
        <StressLevelsChart /> <!-- Embed your chart component -->
    </div>

    <h2>Page Outline</h2>
    <div style="margin-top: 2rem; font-family: Arial, sans-serif; line-height: 1.6; text-align: left;">
        <p><strong>Introduction:</strong>
            Why this topic? Any special phenomenon observed? Why does this topic matter? 
            With remote work becoming a norm, its impact on mental health has been a topic of concern. 
            Are people happier or more stressed working from home? Let's explore the data.
        </p>

        <h3>Body</h3>
        <p><strong>Sub-topics to explore the relationship based on analysis of data:</strong></p>
        <ul>
            <li><strong>Job Category:</strong> Role, industry, and level of seniority vs. work location.</li>
            <li><strong>Demographic (Employee):</strong> Age, gender, and region vs. work location.</li>
            <li><strong>Working Schedule:</strong> Onsite vs. remote vs. hybrid:
                <ul>
                    <li>Working hours</li>
                    <li>Number of virtual meetings</li>
                    <li>Productivity</li>
                </ul>
            </li>
            <li><strong>Health:</strong> 
                <ul>
                    <li>Physical (activity)</li>
                    <li>Mental (social isolation, work-life balance, access to mental support, satisfaction with remote work, sleep quality)</li>
                </ul>
            </li>
        </ul>

        <h3>Conclusion</h3>
        <p>
            What did we find from the above analysis? 
            What's the conclusion? Is there a causal relationship or correlation between remote work and mental issues?
        </p>
        <p>
            <strong>Recommendations:</strong> 
            What suggestions to do something differently to improve employees' quality of both life and work? 
            How should employers manage remote teams? 
            How can workers maintain good mental health?
        </p>
        <h2>Writeup</h2>
        <p>
            For the initial prototype of our final project, we focused on two main tasks. The first was outlining the structure of our article, 
            and the second was creating the interactive visualization. Our research topic centers around whether remote work impacts employees' mental health. 
            Since the dataset includes three types of work styles—on-site, hybrid, and remote—we structured the article to explore the topic from different perspectives by 
            comparing the patterns among these work styles. We conclude the article by discussing whether the remote work trend is beneficial or detrimental to employees and how 
            they can cope with its challenges. To visually represent the data, we used a bar chart to display the distribution of stress levels across the different work styles. We incorporated an interactive 
            filtering bar that allows the audience to explore various stress levels and work styles.
            The most challenging aspect of developing the initial prototype was determining the appropriate visualizations to include in the structure. Initially, we planned to visualize 
            the relationship between mental conditions and the three work styles to provide an initial glimpse of how remote work might affect mental health. However, the data didn't show 
            significant differences in mental conditions across the work styles. As a result, we explored other variables, such as stress levels. We found that stress level provided a clearer 
            metric for differentiating patterns among the work styles. We decided to feature this graph at the beginning of the article to pose a question to the audience, introduce the topic, and encourage them to engage with the analysis in the following sections.
            We incorporated an interactive filtering bar that allows the audience to explore various stress levels and work styles.
        </p>
        <p>
            The most challenging aspect of developing the initial prototype was determining the appropriate visualizations to include in the structure. 
            Initially, we planned to visualize the relationship between mental conditions and the three work styles to provide an initial glimpse of how remote work might affect mental health. However, the data didn't show significant differences in mental conditions across the work styles. 
            As a result, we explored other variables, such as stress levels. We found that stress level provided a clearer metric for differentiating patterns among the work styles.
            We decided to feature this graph at the beginning of the article to pose a question to the audience, introduce the topic, and encourage them to engage with the analysis in the following sections.

        </p>
    </div>
    

</main>

<script>
    import StressLevelsChart from './MentalHealth.svelte'; // Import your chart component
</script>

