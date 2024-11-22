

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
    <div id="chart-container">
        <h2>Stress Levels Across Work Locations</h2> <!-- Chart title moved above the chart -->
        <StressLevelsChart /> <!-- Embed your chart component -->
    </div>
</main>

<script>
    import StressLevelsChart from './MentalHealth.svelte'; // Import your chart component
</script>

