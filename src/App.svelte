<script>
    // COMPONENTS
    import { onMount } from 'svelte';
    import { csvParse } from 'd3-dsv';
    import Chart from "$components/Chart.svelte";

    // DATA
    const dataUrl = 'https://raw.githubusercontent.com/ajstarks/dubois-data-portraits/master/challenge/2024/challenge03/data.csv';

    // VARIABLES
    let data, value;

    // REACTIVE VARIABLES
    

    async function fetchData(url) {
        const resp = await fetch(url);
        const data = await resp.text();
        return csvParse(data);
    }


    async function init() {
        // fetch data
        data = await fetchData(dataUrl);
    }

    // build the chart
    onMount(init);
</script>

<header>
    <h1>DuBois Data Portraits</h1>
    <p class="subhead">Challenge #3. Visit the <a target="_blank" href="https://www.datavisualizationsociety.org/news/2024/2/2/advance-your-data-viz-skills-with-the-weekly-2024-du-bois-visualization-challenge">Data visualization Society</a> for more information.</p>
</header>

<main>
    {#if data}
        <Chart 
            data={data.sort((a,b) => parseFloat(b.Date) - parseFloat(a.Date))}
            value={value}
        />
    {/if}
</main>
  
<style>
    @import '$css/normalize.css';
    @import '$css/fonts.css';
    @import '$css/colors.css';
    @import '$css/app.css';

    header {
		margin-bottom: 2rem;
	}
	header > h1 {
		text-align: center;
	}
	header .subhead {
		margin: 0 auto;
		max-width: 525px;
		text-align: center;
	}

    /* COMBOBOX SELECTOR */
  	:global(.svelte-select) {
		margin: 1rem auto !important;
		max-width: 250px;
  	}
  	:global(input:focus) {
		outline: none;
  	}

	:global(
		.svelte-select .selected-item,
		.svelte-select .item,
		.svelte-select input
	) {
		font-family: 'BentonSansCond-Regular', sans;
	}
</style>
