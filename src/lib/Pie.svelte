<script>
    import * as d3 from 'd3';

    export let data = [];
    
    let fillColors = d3.scaleOrdinal(d3.schemeTableau10); // returns a function that uses index to produce a color
    let sliceGenerator = d3.pie().value(d => d.value);
    let arcData = sliceGenerator(data);

    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);
    let arcs = arcData.map(d => arcGenerator(d));
    
</script>

<style>
    
    svg{
        max-width: 20em;
        margin-block: 2em;
        /* Doesn't clip the shape even if it'd outside of viewbox */
        overflow: visible;
    }

    .container{
        align-items: right;
        flex: 1;
        gap: 10ch;
    }

    span.swatch{
        display: inline-block;
        aspect-ratio: 1 / 1;
        background-color: aquamarine;
        border-radius: 50%;
        background-color: transparent;
        border-radius:50em; 
        /* dotted blue; */
        border-width: 5em;
        border-width: 5px;
        /* border-style: solid; */
        /* border-style: dotted; */
        /* border-style: ridge; */
        /* border-color: black; */
        
    }

    ul.legend{
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(9em, 1fr));
        border-radius: 5em solid black;
        /* border-color: black; */
        /* border: 1em;
        border-color: black;
        border-radius: 80%; */
    }

    li::marker{
        display: flex;
        align-items: center;
        gap: 10fr;
        font-size: 40px;
        /* line-height: 1.5;
        vertical-align: 0.1em;
        transform: translate(-0.1em, 0.2em); */
        /* color:black */
        /* border-radius: 5px solid black; */
    }

    li{
        font-palette: black;
        /* color: aquamarine; */
    }
</style>

<div class="container">

    <!-- NEED HELP ON THE CSS FOR THE LEGEND FOR THE PIE CHART -->
    <svg viewBox= "-50 -50 100 100">
        {#each arcs as arc, i}
            <path d={arc} fill= {fillColors(i)}/>
        {/each}  
    </svg>
    <ul class="legend">
        {#each data as d, index}
            <li style=" color: { fillColors(index) }">
                <span class="swatch"></span>
                {d.label} <em> ({d.value})</em>
            
            </li>
            
        {/each}
    </ul>
</div>
    