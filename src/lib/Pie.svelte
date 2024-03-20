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

    function toggleWedge(index, event){
        if(selectedIndex === index)
        {
            selectedIndex = -1;
            console.log("they are equal!!");
        }
        
        else if(!event.key || event.key === "Enter")
        {
            selectedIndex = index;
        }
    }
    
</script>

<style>
    
    svg{
        max-width: 20em;
        margin-block: 2em;
        /* Doesn't clip the shape even if it'd outside of viewbox */
        overflow: visible;

    }

    svg:has(path:hover, path:focus-visible)
    {
        path:not(:hover, :focus-visible){
            opacity: 50%;
        }
    }

    path{
        transition: 300ms;
        outline: none;
        cursor: pointer;
        --angle: calc(var(--end-angle) - var(--start-angle));
        --mid-angle: calc(var(--start-angle) + var(--angle) / 2);
        transform: rotate(var(--mid-angle)) translateY(0) rotate(calc(-1 * var(--mid-angle)));

        &.selected{
            transform: rotate(var(--mid-angle)) translateY(-6px) scale(1.1)
            rotate(calc(-1 * var(--mid-angle)));
        }
    }

    .container{
        align-items: center;
        flex: 1;
        display: flex;
        gap: 60ch;
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

    <svg viewBox= "-190 -55 100 100">
        {#each arcs as arc, i}
            <path d={arc} 
            style="
            --start-angle: { arcData[i]?.startAngle }rad;
            --end-angle: { arcData[i]?.endAngle }rad;"
            
            fill= {fillColors(i)} 
            tabindex="0"
            aria-label="Pie Chart"
            role= "button"
            class:selected={selectedIndex === i} 
            on:click={e => toggleWedge(i, e)} 
            on:keyup={e => toggleWedge(i, e)}/>
            <!-- selectedIndex = selectedIndex === i ? -1: i} /> -->
        {/each}  
    </svg>
    <ul class="legend">
        {#each data as d, index}
            <li style="--color: { fillColors(index) }">
                <span class="swatch" class:selected={selectedIndex === index}></span>
                {d.label} <em> ({d.value})</em>
            
            </li>
            
        {/each}
    </ul>
</div>
    