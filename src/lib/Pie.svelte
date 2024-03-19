<script>
    import * as d3 from 'd3';

    export let data = [];
    export let selectedIndex = -1;
    
    let fillColors = d3.scaleOrdinal(d3.schemeTableau10); // returns a function that uses index to produce a color
    let sliceGenerator = d3.pie().value(d => d.value);
    let arcData;
    $: arcData = sliceGenerator(data);

    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);
    let arcs;
    $: arcs = arcData.map(d => arcGenerator(d));
    
</script>

<style>
    
    svg{
        max-width: 20em;
        margin-block: 2em;
        /* Doesn't clip the shape even if it'd outside of viewbox */
        overflow: visible;
    }

    svg:has(path:hover)
    {
        path:not(:hover){
            opacity: 50%;
        }
    }

    path{
        transition: 300ms;
        cursor: pointer;
    }

    .container{
        align-items: center;
        flex: 1;
        display: flex;
        gap: 10ch;
    }

    span.swatch{
        display: inline-block;
        aspect-ratio: 1 / 1;
        background-color: var(--color);
        width: 15px;
        height: 15px;
        border-radius: 50%;
    }

    ul.legend{
        /* display: grid; */
        grid-template-columns: repeat(auto-fill, minmax(9em, 1fr));
        border-color: rgb(193, 193, 193);
        border-width: 3px;
        border-style: solid;
        padding-left: 30px;
        padding-right: 30px;
        padding-top: 10px;
        padding-bottom: 10px;
    }

    li{
        font-palette: black;
        list-style: none;
    }

    .selected {
        --color: oklch(60% 45% 0) !important;

        &:is(path) {
            fill: var(--color);
        }
    }

</style>

<div class="container">

    <!-- NEED HELP ON THE CSS FOR THE LEGEND FOR THE PIE CHART -->
    <svg viewBox= "-50 -50 100 100">
        {#each arcs as arc, i}
            <path d={arc} fill= {fillColors(i)} 
            class:selected={selectedIndex === i} on:click={e => 
            selectedIndex = selectedIndex === i ? -1: i} />
        {/each}  
    </svg>
    <ul class="legend">
        {#each data as d, index}
            <li style="--color: { fillColors(index) }">
                <span class="swatch"></span>
                {d.label} <em> ({d.value})</em>
            
            </li>
            
        {/each}
    </ul>
</div>
    