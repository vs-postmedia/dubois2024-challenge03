<script>
    // PROPS
    export let data = [];
    export let value = '';

    // LIBS
    import rough from 'roughjs';
    import { onMount } from 'svelte';
    import { max } from 'd3-array';
    import { scaleBand, scaleLinear } from 'd3-scale';
    
    // COMPONENTS
    import Tooltip from '$components/Tooltip.svelte';

    // DATA
    import backgroundImage from "$images/plate-background.jpg"
	

    // VARS
    let tooltipData;
    let width = 600;
    let height = 800;
    let roughSvg, svg, xExtent, xMax, xScale, yScale;
    const svgContainerId = 'inner-chart';
    const margin = {
		top: 0,
		right: 100,
		bottom: 20,
		left: 0 
	};
    const barHeight = 20;
    let innerHeight = height - margin.top - margin.bottom;
    $: innerWidth = width - margin.left - margin.right;


    // roughjs options
    const fillOptions = {
        bowing: 0.25,
        fill: 'rgba(253,59,90,0.9)',
        fillStyle: 'hachure',
        fillWeight: 1,
        hachureAngle: 90,
        hachureGap: 0.75,
        roughness: 2,
        seed: 1000,
        strokeWidth: 0.25
    };

    // FUNCTIONS
    function addBarLabel(d, index, svg) {
        // create a new text element
        let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
        // set text attributes
        if (index !== 0) {
            text.setAttribute('x', xScale(d.Land) / 1.5);
        } else {
            text.setAttribute('x', xScale(d.Land) / 2);
        }
        text.setAttribute('y', yScale(d.Date) + yScale.bandwidth() / 2);
        text.setAttribute('dy', '0.25rem');
        text.setAttribute('text-anchor', 'center');
        text.setAttribute('font-family', 'BentonSansCond-Bold');
        text.textContent = addCommasToNumber(parseFloat(d.Land));
        svg.appendChild(text);
    }

    function addCommasToNumber(number) {
        return number.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    }
    
    function highlightBar(rect) {
        const bars = document.querySelectorAll(`${svgContainerId} g > path`);

        bars.forEach(d => {
            d.classList.add('non-active');
        })
    }

    $: createRoughPaths = function(d, data, index, fillOptions) {
        if (!roughSvg) return;

        // draw bars
        const rect = roughSvg.rectangle(50, yScale(d.Date), xScale(d.Land), barHeight, fillOptions);
        // add properties object for mouseover events
        rect.properties = d;
        // add classes
        rect.setAttribute('class', 'bar');
        rect.setAttribute('transition', 'opacity 0.3s ease-in-out;');

        // add tooltip
        rect.addEventListener('mouseover', e => {
            rect.properties.x = e.layerX;
            rect.properties.y = e.layerY;
    
            tooltipData = rect.properties;

            rect.setAttribute('opacity', '0.3');
        });

        rect.addEventListener('mouseout', e => {
            tooltipData = null;
            rect.setAttribute('opacity', '1');
        })

        // add rect to svg
        svg.appendChild(rect);

        // append text on first & last bars
        if (index === 0 || index === data.length - 1) {
           addBarLabel(d, index, svg);
        }
    }

    function init() {
        // get svg for roughjs
        svg = document.getElementById(svgContainerId);
        roughSvg = rough.svg(svg);
    }
    
    // REACTIVE VARIABLES
    $: console.log(data);
    $: xMax = max(data, d => parseFloat(d.Land));

    $: xScale = scaleLinear()
        .domain([0, xMax])
        .range([0, innerWidth]);

    $: yScale = scaleBand()
        .domain(data.map(d => d.Date))
        .range([innerHeight, 0])
        .padding(0.25);
    
    // LIGHTS! CAMERA! ACTIONS
    onMount(init);
</script>


<div class="chart-container" bind:clientWidth={width}>
    <h2 id="viz-header">Acres of land owned by Blacks in Georgia.</h2>

    <svg id="chart" {height} {width}>
        <g id="inner-chart" transform={`translate(${margin.left}, ${margin.top})`}>
            {#each data as d, i}
                <g class="axis-y" dy="100">
                    <text
                        text-anchor="end"
                        x="35"
                        y={yScale(d.Date) + yScale.bandwidth() / 2}
                        dy={barHeight / 5}
                        >{d.Date}
                    </text>
                    {createRoughPaths(d, data, i, fillOptions)}
                </g>
            {/each}
        </g>
    </svg>
    {#if tooltipData}
        <Tooltip data={tooltipData} width={innerWidth} />
    {/if}
</div>

<style>
    #viz-header {
        font-size: 1.65rem;
        letter-spacing: 0.1rem;
        line-height: 1.15;
        margin: 0 18%;
        text-align: center;
        text-transform: uppercase;
    }
    .chart-container {
        background-image: url('$images/plate-background.jpg');
        background-position: center top;
        background-repeat: no-repeat;
        background-size: cover;
        padding: 40px;
    }
    #chart text {
        color: black;
    }

    #chart .axis-y text {
        fill: rgba(134,112,93, 0.8);
        letter-spacing: 0.015rem;
        line-height: 1.75rem;
        text-transform: uppercase;
    }
    #chart .active {
        opacity: 1
    }

    #chart .bar > path:hover {
        opacity: 0.3;
    }
</style>