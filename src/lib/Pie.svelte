<script>
    import * as d3 from 'd3';

    let data = [1, 2, 3, 4, 5, 5];
    let fillColors = d3.scaleOrdinal(d3.schemeTableau10); // returns a function that uses index to produce a color
    let sliceGenerator = d3.pie();
    let arcData = sliceGenerator(data);

    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);
    let arcs = arcData.map(d => arcGenerator(d))
    
</script>

<style>
    svg{
        max-width: 20em;
        margin-block: 2em;
        /* Doesn't clip the shape even if it'd outside of viewbox */
        overflow: visible;
    }
</style>


<svg viewBox= "-50 -50 100 100">
    {#each arcs as arc, i}
        <path d={arc} fill= {fillColors(i)}/>
    {/each}
</svg>
